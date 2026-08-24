---
title: "Shared Memory with OpenMP"
teaching: 25
exercises: 15
---

::::::::::::::::::::::::::::::::::::: questions
- How do I use multiple cores on a single node?
- What is the difference between threads and processes?
- How do I request multiple cores in SLURM?
- Why does Python's threading module not give parallelism?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand shared-memory parallelism concepts
- Learn the difference between threads and processes in Python
- Use Python's multiprocessing module for true parallelism
- Set SLURM directives for multi-core jobs
- Avoid race conditions through stateless task design
::::::::::::::::::::::::::::::::::::::::::::::::

## Shared-Memory Parallelism in Context

Sagehen's `amd` nodes have 128 cores and 512 GB of RAM each. Shared-memory parallelism uses multiple cores on one node, all reading and writing to the same memory pool. This is in contrast to distributed parallelism (next two episodes), which uses many nodes coordinating through network messages.

Shared memory is simpler: no networking, no message-passing protocols, no failure-mode complexity from a flaky link between machines. The cost is that you cannot scale past one node. For most research workflows on Sagehen, that is plenty: 128 cores is enormous compared to a laptop's 8.

## Threads vs. Processes

### Threads

All threads share the same memory space and process ID. Communication between threads is just reading and writing shared variables.

**Disadvantage in Python:** The Global Interpreter Lock (GIL) allows only one thread to execute Python bytecode at a time. Threads can wait for I/O concurrently, but they cannot run pure-Python computation in parallel. For numerical work in Python, threads give zero speedup.

The GIL is released when calling into C extensions like NumPy, so heavy numeric work that spends most of its time inside `np.dot` or similar can benefit from threading. But pure Python loops cannot.

### Processes

Each process has its own memory space and Python interpreter. There is no GIL coordination between them, so processes give true parallelism. The cost is that data must be explicitly serialized to communicate between processes (pickled and sent through pipes), which is slower than reading shared memory.

For most Python work on Sagehen, processes via the `multiprocessing` module are the right answer.

::::::::::::::::::::::::::::::::::::: callout

**Python Parallelism: Use Multiprocessing**

In Python, always use multiprocessing (processes) for compute-intensive work. Threads are limited by the GIL. Multiprocessing gives true parallelism.

The exception is I/O-bound work: web requests, database calls, file downloads. Threads parallelize those well because the GIL is released while waiting for I/O. For computation, use multiprocessing or move the inner loop to NumPy / Cython / C.

::::::::::::::::::::::::::::::::::::::::::::::::

## Python Multiprocessing

### Simple Example

```python
from multiprocessing import Pool
import os

def process_file(filename):
    with open(filename) as f:
        data = f.read()
    return filename, len(data)

if __name__ == "__main__":
    files = [f for f in os.listdir("data") if f.endswith(".txt")]

    with Pool(4) as p:
        results = p.map(process_file, files)

    for filename, result in results:
        print(f"{filename}: {result}")
```

The `Pool(4)` creates 4 worker processes. `p.map(fn, items)` distributes `items` across workers, each calling `fn` on its share. The results come back in input order. The `if __name__ == "__main__":` guard is mandatory: without it, child processes re-import the script and recursively spawn more workers.

### Monte Carlo with Pool

```python
#!/usr/bin/env python3
import numpy as np
from multiprocessing import Pool

def estimate_pi(n_samples):
    inside = 0
    for _ in range(n_samples):
        x, y = np.random.random(), np.random.random()
        if x**2 + y**2 <= 1:
            inside += 1
    return 4 * inside / n_samples

if __name__ == "__main__":
    with Pool(processes=10) as pool:
        estimates = pool.map(estimate_pi, [10_000_000] * 10)

    print(f"pi approx {np.mean(estimates):.6f} +/- {np.std(estimates):.6f}")
```

Each worker independently runs the Monte Carlo computation, then the parent process aggregates the results. Because each worker has its own NumPy random state, the seeds happen to be independent without explicit seeding. For reproducibility, seed inside `estimate_pi` based on a worker-specific value.

## SLURM Directives for Multi-Core Jobs

```bash
#!/bin/bash
#SBATCH --job-name=multicore
#SBATCH --ntasks=1              # One task (no MPI)
#SBATCH --cpus-per-task=8       # 8 CPU cores
#SBATCH --mem=16G
#SBATCH --time=01:00:00

python3 my_parallel_script.py
```

The two SLURM directives that matter for shared-memory work:

`--ntasks=1`: one task, not multiple. Multiprocessing creates child processes inside one task; SLURM does not need to know about them.

`--cpus-per-task=N`: how many cores SLURM allocates to that task. This is the number you should pass to `Pool()`.

Then in your script, respect the allocation:

```python
import os
n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 1))
with Pool(processes=n_cores) as pool:
    results = pool.map(process_function, data)
```

This pattern lets the same script run with any allocation by reading `SLURM_CPUS_PER_TASK` instead of hardcoding a number. Sagehen `amd` nodes have 128 cores and 512 GB RAM, so you can request up to that on a single node.

::::::::::::::::::::::::::::::::::::: callout

**How many cores should I request?**

For pure CPU-bound work, request as many cores as your script can keep busy. Profile with a small allocation first to see where you stop scaling.

For mixed CPU/I/O work, requesting too many cores wastes scheduling priority because most cores idle waiting for I/O. 16 to 32 cores is often a sweet spot.

For ML preprocessing with Dask, 16 to 32 workers x 4 threads each (so 64-128 effective cores on a 128-core node) usually balances throughput against per-worker memory.

Avoid the temptation to always request 128. SLURM scheduling priority depends on how much of your past requested allocation you actually used.

::::::::::::::::::::::::::::::::::::::::::::::::

## Race Conditions

### The Problem

```python
# BAD: Multiple processes increment shared counter simultaneously
counter.value += 1  # Read-modify-write is NOT atomic
```

Even with multiprocessing's "shared memory primitives" (`Value`, `Array`), increments are not atomic. Two processes can both read the same value, both increment locally, and both write back the same value+1; the actual count is one less than expected. Locks fix this but slow things down.

### The Solution

Avoid shared mutable state entirely:

```python
# BEST: No shared state needed
def process_item(x):
    return x * 2

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(process_item, range(100))
    total = sum(results)  # Combine after all processes finish
```

The map-reduce pattern (workers process independently, parent aggregates) sidesteps race conditions entirely. If your task feels like it needs shared state, look for a way to restructure it as map-reduce. Almost everything embarrassingly parallel can be expressed this way.

::::::::::::::::::::::::::::::::::::: challenge

**Parallelize a Script**

Convert this serial analysis to use multiprocessing.Pool:

```python
import time, numpy as np
results = []
for i in range(10):
    np.random.seed(i)
    data = np.random.randn(1000000)
    time.sleep(1)  # Simulate computation
    results.append({'id': i, 'mean': np.mean(data)})
```

::::::::::::::::::::::::::::::::::::: solution

```python
import time, numpy as np, os
from multiprocessing import Pool

def analyze_dataset(dataset_id):
    np.random.seed(dataset_id)
    data = np.random.randn(1000000)
    time.sleep(1)
    return {'id': dataset_id, 'mean': np.mean(data)}

if __name__ == "__main__":
    n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 4))
    with Pool(processes=n_cores) as pool:
        results = pool.map(analyze_dataset, range(10))
    for r in results:
        print(f"Dataset {r['id']}: mean={r['mean']:.3f}")
```

With 4 cores, runs in ~3 seconds instead of ~10 seconds. With 10 cores, runs in ~1 second. Scaling is near-perfect because tasks are independent and computation dominates over communication overhead.

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

**Choose the Right Tool**

For each scenario, pick: shared-memory `multiprocessing.Pool`, SLURM job arrays, or threading. Justify briefly.

A. Download and parse 1,000 web pages.

B. Process 1,000 medical images, each takes 30 seconds, total work fits on one node.

C. Run 1,000 instances of a 4-hour simulation, each needing 64 GB RAM.

::::::::::::::::::::::::::::::::::::: solution

A. Threading. I/O-bound work releases the GIL while waiting on the network. `concurrent.futures.ThreadPoolExecutor` is the cleanest API.

B. multiprocessing.Pool on one node. 1,000 images x 30 seconds = ~8 hours serial; with 32 cores it is ~15 minutes. Fits on one Sagehen `amd` node.

C. SLURM job arrays. 1,000 x 64 GB = 64 TB total memory commitment, which is across many nodes. Each task is long enough that SLURM overhead is irrelevant. `--array=1-1000%50` would be the typical submission.

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- In Python, use multiprocessing (not threading) for true parallelism on CPU-bound work
- Use threading for I/O-bound work (downloads, database calls)
- Request cores with `#SBATCH --cpus-per-task=N`
- Read `os.environ['SLURM_CPUS_PER_TASK']` so scripts adapt to allocation
- Use `Pool.map()` to parallelize independent tasks
- Avoid race conditions by not sharing mutable state; use map-reduce
- Sagehen amd nodes have 128 cores and 512 GB; a single node is often enough
- Shared-memory parallelism is limited to a single node; arrays scale further
::::::::::::::::::::::::::::::::::::::::::::::::
