---
title: "Threading in Python and R Parallelism"
teaching: 20
exercises: 15
---

::::::::::::::::::::::::::::::::::::: questions
- When should I use threading instead of multiprocessing in Python?
- How do I parallelize R code?
- What are common shared-memory patterns for research computing?
- What performance tips should I follow?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Recognize when threading is appropriate for Python
- Use R's parallel package for multi-core computation
- Apply common shared-memory patterns (file processing, bootstrap)
- Follow performance best practices for shared-memory parallelism
::::::::::::::::::::::::::::::::::::::::::::::::

## When Threading Is the Right Choice in Python

The previous episode argued that processes (`multiprocessing`) beat threads in Python because the GIL prevents true thread parallelism for pure Python code. That is true for CPU-bound work. For I/O-bound work, threading is often the better choice because:

- Threads share memory, so passing data between them is free (no pickling overhead).
- The GIL is released while waiting for I/O, so threads do run in parallel during network calls, disk reads, or database queries.
- Spinning up threads is much cheaper than spinning up processes.

```python
from concurrent.futures import ThreadPoolExecutor
import requests

urls = ['http://example.com/data1.csv', 'http://example.com/data2.csv']

with ThreadPoolExecutor(max_workers=8) as executor:
    responses = list(executor.map(requests.get, urls))
```

Eight concurrent HTTP requests with 8 threads finish in roughly the time of one request, not eight. Try that with `multiprocessing.Pool` and you incur process startup costs, plus pickling each `Response` object, which can be slower than running serially.

The rule: I/O-bound (network, disk wait, database) -> threading. CPU-bound (numerical computation, parsing, image processing) -> multiprocessing.

## R Parallel Computing

R has built-in parallel support via the `parallel` package, which works very differently from Python because R does not have a GIL. R's `parLapply` is closer in spirit to Python's `multiprocessing.Pool.map`.

### Parallel Apply

```r
library(parallel)

square_plus <- function(x) { return(x^2 + 2*x) }

# Serial
result_serial <- lapply(1:1000000, square_plus)

# Parallel
n_cores <- detectCores()
cl <- makeCluster(n_cores)
result_parallel <- parLapply(cl, 1:1000000, square_plus)
stopCluster(cl)
```

The `makeCluster` / `stopCluster` pair is mandatory: without `stopCluster`, the worker processes leak and stay around until the R session ends. On Sagehen HPC, leaked workers can fill up the node's process table and cause subsequent jobs to fail mysteriously.

### R Monte Carlo Example

```r
library(parallel)

estimate_pi_sample <- function(n) {
    inside <- sum(runif(n)^2 + runif(n)^2 <= 1)
    return(4 * inside / n)
}

n_cores <- detectCores()
cl <- makeCluster(n_cores)
clusterExport(cl, "estimate_pi_sample", envir = environment())

estimates <- parLapply(cl, rep(1000000, 10), estimate_pi_sample)
stopCluster(cl)

cat("pi approx", mean(unlist(estimates)), "\n")
```

`clusterExport` is needed because each worker is a fresh R process and does not inherit the parent's function definitions automatically. For one-off scripts where the function is defined inline, this is easy to miss; the workers will fail with "function not found" errors.

### R SLURM Script

```bash
#!/bin/bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G

Rscript << 'R_SCRIPT'
library(parallel)
n_cores <- as.integer(Sys.getenv("SLURM_CPUS_PER_TASK"))
cl <- makeCluster(n_cores)
# ... your parallel R code ...
stopCluster(cl)
R_SCRIPT
```

The same lesson as for Python: read `SLURM_CPUS_PER_TASK` rather than `detectCores()`. `detectCores()` returns the physical core count of the node (128 on Sagehen amd), not the cores SLURM allocated to your job. Using that count will oversubscribe and starve other jobs sharing the node.

## Common Shared-Memory Patterns

### Pattern: Parallel File Processing

```python
#!/usr/bin/env python3
from multiprocessing import Pool
import glob, os

def process_file(filepath):
    with open(filepath) as f:
        lines = f.readlines()
    return {'file': os.path.basename(filepath), 'lines': len(lines),
            'chars': sum(len(line) for line in lines)}

if __name__ == "__main__":
    files = glob.glob("data/*.txt")
    n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 4))
    with Pool(processes=n_cores) as pool:
        results = pool.map(process_file, files)
    total_lines = sum(r['lines'] for r in results)
    print(f"Processed {len(files)} files, {total_lines} total lines")
```

This is the bread-and-butter HPC workload: many independent files, identical processing. Multiprocessing scales linearly until you saturate I/O bandwidth (around 8 to 16 workers on Sagehen `/scratch`, fewer on `/bigdata`).

### Pattern: Bootstrap Resampling

```python
#!/usr/bin/env python3
import numpy as np
from multiprocessing import Pool

def bootstrap_resample(args):
    data, seed = args
    np.random.seed(seed)
    indices = np.random.choice(len(data), size=len(data), replace=True)
    return np.mean(data[indices])

if __name__ == "__main__":
    data = np.loadtxt("data.csv")
    tasks = [(data, seed) for seed in range(1000)]

    with Pool(processes=8) as pool:
        bootstrap_means = pool.map(bootstrap_resample, tasks)

    print(f"95% CI: [{np.percentile(bootstrap_means, 2.5):.4f}, "
          f"{np.percentile(bootstrap_means, 97.5):.4f}]")
```

Note the `(data, seed)` tuple. `Pool.map` only passes one argument; for multi-argument functions either pack into a tuple, or use `pool.starmap` which unpacks tuples automatically.

A subtle issue: each worker receives a copy of `data` in its `args`, which means `data` is pickled and sent to each worker. For a 1 GB array sent to 100 workers, that is 100 GB of pickling overhead. For large shared data, use `Pool` with an `initializer` that loads the data once per worker, or use a memory-mapped file.

## Performance Tips

1. **Measure overhead**: Parallelization costs (process creation, data copying) must be less than computation time. For tasks under a millisecond, parallelism makes things slower.

2. **Use the right number of processes**: Match `SLURM_CPUS_PER_TASK`, not the node's physical core count. Oversubscribing causes context-switch overhead and is rude to neighbors on shared nodes.

3. **Avoid copying large data**: For datasets over a few hundred MB, prefer memory-mapped files (`np.memmap`) that workers can read without pickling.

4. **Profile first**: Use `cProfile` to find bottlenecks before parallelizing. If 90% of time is in one function, parallelize that function. If time is spread evenly across 50 functions, consider whether the script is even worth parallelizing.

5. **Consider job arrays**: If tasks are embarrassingly parallel and total runtime is hours, SLURM job arrays are often simpler than in-script parallelism and scale across nodes.

::::::::::::::::::::::::::::::::::::: callout

## Threading vs. multiprocessing vs. arrays decision tree

Each task is short (under a second) and reads from disk: threading.

Each task is short (under a minute) and is computational: multiprocessing on one node.

Each task is medium (minutes to a few hours) and is computational, fits on one node: multiprocessing.

Each task is long (hours), or you need more than one node, or each task needs significant memory: SLURM job arrays.

When in doubt, start with multiprocessing on one node. It is the easiest to debug and offers about 100x speedup on Sagehen. Move to arrays only when one node is not enough.

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

**Bootstrap Confidence Interval**

Write a parallel bootstrap script that computes a 95% confidence interval for the mean of a dataset using 4 cores and 1000 resamples.

::::::::::::::::::::::::::::::::::::: solution

```python
import numpy as np, os
from multiprocessing import Pool

def bootstrap_mean(args):
    data, seed = args
    np.random.seed(seed)
    return np.mean(np.random.choice(data, size=len(data), replace=True))

if __name__ == "__main__":
    data = np.random.randn(10000)
    n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 4))
    with Pool(n_cores) as pool:
        means = pool.map(bootstrap_mean, [(data, i) for i in range(1000)])
    print(f"95% CI: [{np.percentile(means, 2.5):.4f}, {np.percentile(means, 97.5):.4f}]")
```

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

**Diagnose a Slow Pool**

A colleague's `Pool(64)` script runs no faster than `Pool(8)`. Their function `process(x)` does heavy NumPy linear algebra. List two likely causes and how to verify each.

::::::::::::::::::::::::::::::::::::: solution

Cause 1: NumPy is using its own internal threading (BLAS), so 8 Python workers x 16 BLAS threads each = 128 threads competing for 128 cores. Verify: check `OMP_NUM_THREADS` and `OPENBLAS_NUM_THREADS` env vars. Set them to 1 inside the script before importing NumPy.

Cause 2: input pickling dominates. Each call passes a large array; with 64 workers all receiving copies, the pickling cost exceeds the parallel speedup. Verify: time `pickle.dumps(x)` for a representative input. Fix: use shared memory or a memory-mapped array.

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Threading is right for I/O-bound work in Python; multiprocessing for CPU-bound
- R's parallel package provides parLapply for multi-core computation
- Use `clusterExport` to share function definitions with R workers
- Common patterns: file processing, bootstrap resampling, parameter sweeps
- Match process count to SLURM_CPUS_PER_TASK, not the node's physical cores
- Avoid pickling large data; use memory-mapped files for shared inputs
- Profile before parallelizing; the gain depends on what dominates runtime
::::::::::::::::::::::::::::::::::::::::::::::::
