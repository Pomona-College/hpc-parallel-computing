---
title: Reference
---

## Parallel Computing Concepts

### Key Definitions

**Speedup:** How much faster parallel code runs compared to serial
```
Speedup = T_serial / T_parallel
```

**Efficiency:** How well you're using allocated resources
```
Efficiency = Speedup / number_of_processors
```
Target: 60-80% efficiency. >50% is acceptable.

**Amdahl's Law:** Theoretical maximum speedup when parallelizing a program
```
Speedup = 1 / (f_s + (1 - f_s) / p)
```
Where:
- `f_s` = fraction of code that is serial (0 to 1)
- `p` = number of processors

Example: If 10% of your code is serial and you use 16 cores:
```
Speedup = 1 / (0.1 + 0.9/16) = 1 / 0.1563 ≈ 6.4x
```

**Strong Scaling:** How performance improves when you add more processors to the same problem

**Weak Scaling:** How performance scales when you increase problem size proportionally with processor count

### Parallelization Strategies

#### 1. Job Arrays (Embarrassingly Parallel)

Best for: Independent tasks, parameter sweeps, Monte Carlo runs

```bash
#!/bin/bash
#SBATCH --job-name=sweep
#SBATCH --array=1-100
#SBATCH --time=00:30:00
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1

# Each array task runs independently
TASK_ID=$SLURM_ARRAY_TASK_ID
echo "Running simulation $TASK_ID"
python my_simulation.py --config config_$TASK_ID.yml
```

Key environment variables:
- `$SLURM_ARRAY_TASK_ID` - Unique ID for this task (1-100)
- `$SLURM_ARRAY_JOB_ID` - Master job ID

Submit: `sbatch my_array.sbatch`

Expected speedup: Near-linear (tasks run independently)

#### 2. Shared-Memory Parallelism (Single Node)

Best for: Data parallelism on one machine, shared data structures

**Python multiprocessing:**
```python
from multiprocessing import Pool
import os

def process_item(item):
    return item * 2

if __name__ == '__main__':
    # Respect SLURM allocation
    n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 1))

    data = list(range(1000))
    with Pool(n_cores) as p:
        results = p.map(process_item, data)
```

SLURM script:
```bash
#!/bin/bash
#SBATCH --job-name=multiproc
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=01:00:00

module load miniconda3
python my_parallel_script.py
```

**R parallel package:**
```r
library(parallel)

# Auto-detect available cores (respects SLURM allocation)
n_cores <- as.numeric(Sys.getenv("SLURM_CPUS_PER_TASK", 4))

cl <- makeCluster(n_cores)
results <- parLapply(cl, 1:1000, function(x) x * 2)
stopCluster(cl)

# Alternative: mclapply (forking, simpler)
results <- mclapply(1:1000, function(x) x * 2, mc.cores=n_cores)
```

SLURM script: Same as Python above

Expected speedup: 60-90% efficiency on single node (communication overhead is low)

#### 3. Distributed Computing with MPI (Multi-Node)

Best for: Tightly coupled problems, simulations requiring inter-node communication

**Python with mpi4py:**
```python
from mpi4py import MPI
import numpy as np

comm = MPI.COMM_WORLD
rank = comm.Get_rank()
size = comm.Get_size()

if rank == 0:
    data = np.arange(size) * 10
else:
    data = None

# Distribute data from rank 0 to all ranks
my_data = comm.scatter(data, root=0)
print(f"Rank {rank} received {my_data}")

# Each rank does work
result = my_data * 2

# Gather results back to rank 0
results = comm.gather(result, root=0)

if rank == 0:
    print(f"All results: {results}")

comm.Barrier()  # Synchronize all ranks
```

SLURM script:
```bash
#!/bin/bash
#SBATCH --job-name=mpi_job
#SBATCH --ntasks=8
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4
#SBATCH --time=01:00:00

module load openmpi
module load miniconda3

# srun launches MPI-aware job
srun python my_mpi_program.py
```

**MPI Collectives Reference:**
```python
# Broadcast: rank 0 sends to all
data = comm.bcast(data, root=0)

# Scatter: divide array among ranks
my_chunk = comm.scatter(data, root=0)

# Gather: collect results from all ranks
results = comm.gather(my_result, root=0)

# Reduce: apply operation (SUM, MAX, MIN, etc)
total = comm.reduce(my_value, op=MPI.SUM, root=0)

# Allreduce: reduce and broadcast to all
total = comm.allreduce(my_value, op=MPI.SUM)

# Barrier: synchronize all ranks
comm.Barrier()
```

Expected speedup: Varies significantly (limited by communication overhead and network latency)

## SLURM Job Submission Reference

### Common SBATCH Directives

```bash
#!/bin/bash

# Job identification
#SBATCH --job-name=MyJob          # Job name
#SBATCH --output=job_%j.log       # Output file (%j = job ID)
#SBATCH --error=job_%j.err        # Error file

# Time and resource limits
#SBATCH --time=01:30:00           # Time limit (HH:MM:SS)
#SBATCH --partition=amd           # Partition (amd, gpu, short)

# Processor allocation
#SBATCH --ntasks=4                # Total number of tasks (MPI processes)
#SBATCH --cpus-per-task=2         # Cores per task
#SBATCH --nodes=2                 # Number of nodes
#SBATCH --ntasks-per-node=2       # Tasks per node

# Memory allocation
#SBATCH --mem=64G                 # Total memory per node
#SBATCH --mem-per-cpu=8G          # Memory per core

# Job arrays
#SBATCH --array=1-100             # Array range
#SBATCH --array=1-100%10          # Array with max 10 jobs running

# GPU resources
#SBATCH --gpus=2                  # Total GPUs
#SBATCH --gpus-per-task=1         # GPUs per task
#SBATCH --gres=gpu:a100:2         # Specific GPU type (a100, l40s, rtxpro6000)
```

### Common SLURM Commands

```bash
# Submit a job
sbatch my_script.sbatch
sbatch --job-name=test my_script.sbatch

# Check job status
squeue                          # All jobs
squeue -u $USER                 # Your jobs
squeue -j JOB_ID                # Specific job

# Job information
scontrol show job JOB_ID        # Full job details
sacct -j JOB_ID                 # Job accounting/statistics

# Cancel jobs
scancel JOB_ID
scancel -u $USER                # Cancel all your jobs

# Cluster information
sinfo                           # All partitions
sinfo -p amd                    # Specific partition
sinfo -N                        # Per-node details
sinfo -l                        # Long format

# Node information
sinfo -N -n node-name           # Details for specific node
```

### Environment Variables in Scripts

Inside your SBATCH script, access:

```bash
$SLURM_JOB_ID              # Job ID
$SLURM_ARRAY_TASK_ID       # Array task index (1-100 for --array=1-100)
$SLURM_ARRAY_JOB_ID        # Master job ID
$SLURM_NTASKS              # Total number of tasks
$SLURM_CPUS_PER_TASK       # Cores per task
$SLURM_NNODES              # Number of nodes
$SLURM_NODELIST            # List of allocated nodes
$SLURM_SUBMIT_DIR          # Directory job was submitted from
$SLURM_TMPDIR              # Temporary directory on node
```

## Sagehen HPC Cluster Specifications

### Hardware

| Component | Specification |
|-----------|---------------|
| **Compute Nodes** | 12× AMD EPYC with 128 cores each |
| **Total Cores** | 1,536 cores |
| **Memory per Node** | 512 GB |
| **GPU Nodes** | 10 GPUs total: 4× A100 (80 GB), 4× L40S (48 GB), 2× RTX PRO 6000 (96 GB) |
| **Interconnect** | InfiniBand 100Gb/s |
| **Filesystem** | BeeGFS parallel filesystem |

### Storage

| Mount Point | Size | Purpose |
|------------|------|---------|
| `/rhome` | 100 GB | Personal home directory (backed up) |
| `/bigdata` | 1 TB | Lab storage (backed up, shared) |
| `/scratch` | Large | SSD-backed temporary (NOT backed up) |
| `/tmpfs` | RAM-based | Ultra-fast temporary (NOT backed up) |

### Partitions

| Partition | Nodes | Use Case | Max Time |
|-----------|-------|----------|----------|
| `amd` | 12 | General purpose, CPU-heavy | 14 days |
| `gpu` | 5 | GPU workloads | 3 days |
| `short` | Any | Quick tests, debugging | 1 hour |

Default partition is `amd`. Always specify `--partition=short` for testing.

### Module System

Sagehen uses Lmod for module management:

```bash
# View available modules
module avail
module avail python
module avail gcc

# Load modules
# gcc is available system-wide on Sagehen -- no module load needed
module load miniconda3
module load openmpi

# View loaded modules
module list

# Unload modules
module unload gcc
module purge              # Unload all

# Create custom module collections
module save my_env
module restore my_env
```

## Performance Optimization Checklist

1. **Profile First**
   - Use `cProfile` (Python), `Rprof` (R)
   - Identify the bottleneck before parallelizing

2. **Measure Serial Baseline**
   - Run single-core version: `#SBATCH --ntasks=1 --cpus-per-task=1`
   - Record wall-clock time

3. **Estimate Speedup with Amdahl's Law**
   - What fraction is inherently serial?
   - What's the theoretical max speedup?

4. **Choose Strategy Wisely**
   - Independent tasks? Use job arrays
   - Shared data on one node? Use multiprocessing/OpenMP
   - Tightly coupled, multi-node? Use MPI

5. **Benchmark Scaling**
   - Test with 1, 2, 4, 8, 16 cores
   - Plot speedup vs cores
   - Is it linear or sublinear?

6. **Calculate Efficiency**
   - Efficiency = Speedup / cores
   - Acceptable: >50%, Good: >70%

7. **Verify Correctness**
   - Parallel results should match serial
   - Check a few sample runs

## Common Mistakes

| Mistake | Impact | Solution |
|---------|--------|----------|
| Parallelizing the wrong section | Wasted effort | Profile first! |
| Too much communication overhead | Poor scaling | Reduce sync points |
| Race conditions | Wrong results | Use locks or atomic ops |
| Not respecting SLURM allocation | Job crashes | Use `$SLURM_CPUS_PER_TASK` |
| Mixing job arrays with MPI | Deadlock/waste | Use one approach per job |
| No baseline measurement | Can't measure speedup | Time serial version first |

## Quick Reference: When to Use What

```
Is the workload embarrassingly parallel?
├─ YES → Use Job Arrays (SLURM --array)
└─ NO ─→ Can it fit on one node?
         ├─ YES → Use shared-memory (Python Pool, R parallel)
         └─ NO  → Use MPI
```

## Useful Resources

- **SLURM Documentation:** https://slurm.schedmd.com/
- **Python multiprocessing:** https://docs.python.org/3/library/multiprocessing.html
- **mpi4py:** https://mpi4py.readthedocs.io/
- **R parallel package:** https://cran.r-project.org/web/packages/parallel/
- **Amdahl's Law calculator:** https://en.wikipedia.org/wiki/Amdahl%27s_law
- **HPC Support:** its-hpc@pomona.edu
