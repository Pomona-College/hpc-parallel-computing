---
title: "Distributed Computing Concepts"
teaching: 10
exercises: 5
---

::::::::::::::::::::::::::::::::::::: questions
- When do I need distributed-memory computing?
- What is MPI and how does it work?
- How do I run MPI programs on Sagehen HPC?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand when MPI is appropriate
- Learn MPI core concepts: rank, communicator, send/recv
- Understand SLURM directives for MPI jobs
::::::::::::::::::::::::::::::::::::::::::::::::

## When Do You Need Distributed Computing?

::::::::::::::::::::::::::::::::::::: callout

**Use MPI (distributed computing) when ALL of these apply:**
1. Problem is too large for one node (> 128 cores or 512GB memory)
2. Tight communication loop between processors
3. Iterative algorithms with frequent synchronization
4. Computation time far exceeds communication time

**Don't use MPI if:**
- Tasks are embarrassingly parallel (use job arrays)
- Problem fits on single node (use multiprocessing)
- Communication overhead would exceed speedup benefit

::::::::::::::::::::::::::::::::::::::::::::::::

**MPI makes sense for:** Large-scale physics simulations, molecular dynamics, coupled multi-physics problems.

**MPI is overkill for:** Parameter sweeps, processing independent files, single-node analyses.

## MPI Core Concepts

**Rank:** Each process has a unique ID.
**Communicator:** Group of processes that can communicate (default: `COMM_WORLD`).
**Send/Recv:** Explicit message passing between ranks.

### Basic Operations

```python
from mpi4py import MPI

comm = MPI.COMM_WORLD
rank = comm.Get_rank()     # What's my process ID?
size = comm.Get_size()     # How many total processes?

comm.send(data, dest=1)           # Send to rank 1
data = comm.recv(source=0)        # Receive from rank 0
data = comm.bcast(data, root=0)   # Rank 0 sends to all
total = comm.reduce(my_value, op=MPI.SUM, root=0)  # Sum across all
comm.Barrier()                    # Wait for all to reach this point
```

## SLURM Directives for MPI Jobs

### Single Node, Multiple Ranks

```bash
#!/bin/bash
#SBATCH --ntasks=8           # 8 MPI processes
#SBATCH --cpus-per-task=1
#SBATCH --nodes=1
#SBATCH --mem=16G

srun python3 my_mpi_program.py
```

### Multiple Nodes

```bash
#!/bin/bash
#SBATCH --ntasks=32          # 32 MPI processes total
#SBATCH --nodes=4            # Across 4 nodes
#SBATCH --ntasks-per-node=8  # 8 processes per node
#SBATCH --mem-per-cpu=2G

srun python3 my_mpi_program.py
```

**Key:** Use `srun` to launch MPI programs (not `mpirun`). SLURM manages distribution.

## InfiniBand Advantage

Sagehen has InfiniBand 100Gb/s interconnect:
- **100 Gb/s bandwidth** (vs. 1-10 Gb/s Ethernet)
- **Sub-microsecond latency**
- MPI programs scale much better on fast interconnects

::::::::::::::::::::::::::::::::::::: challenge

**When to Use MPI?**

For each scenario, decide: job arrays, multiprocessing, or MPI?

1. 500 independent simulations, 1 hour each
2. 3D fluid simulation on a grid with neighbor dependencies, needs 256 cores
3. Processing 10,000 images independently

::::::::::::::::::::::::::::::::::::: solution

1. **Job arrays** -- embarrassingly parallel, no communication needed
2. **MPI** -- tightly coupled, needs more than one node (256 > 128 cores/node)
3. **Job arrays** -- each image is independent

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Distributed-memory systems have separate memory per node
- MPI enables explicit message passing between processes
- Use MPI only for tightly-coupled algorithms across multiple nodes
- Launch MPI with srun under SLURM
- InfiniBand makes MPI practical on Sagehen
::::::::::::::::::::::::::::::::::::::::::::::::
