---
title: "Types of Parallelism"
teaching: 10
exercises: 10
---

::::::::::::::::::::::::::::::::::::: questions
- What is the difference between shared memory and distributed memory?
- What is data parallelism vs task parallelism?
- How does Sagehen's architecture combine both?
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives
- Understand shared-memory and distributed-memory parallelism
- Learn the difference between task parallelism and data parallelism
- Understand Sagehen's hybrid architecture
::::::::::::::::::::::::::::::::::::::::::::::::

## Shared-Memory vs. Distributed-Memory

### Shared-Memory Parallelism

Multiple processors access the **same memory space**:

```
                   Shared Memory (RAM)
                        |
         _______________|_______________
        |               |               |
      Core 0          Core 1          Core 2
```

**Advantages:** Easy communication, low latency, good for single-node work.
**Challenges:** Race conditions, limited to single-node memory.
**Examples:** OpenMP, Python multiprocessing, R parallel package.

### Distributed-Memory Parallelism

Each processor has its **own memory**. Data sharing requires explicit message passing:

```
  Node 1            Node 2            Node 3
  Memory            Memory            Memory
    |                 |                 |
  Core 0            Core 1            Core 2

  Communication: explicit send/receive messages
```

**Advantages:** Scales to many nodes, simpler synchronization.
**Challenges:** Higher latency, explicit communication code.
**Examples:** MPI, Spark, distributed databases.

### Sagehen's Hybrid Architecture

Sagehen uses both:
- **Within a node:** 128 cores sharing 512GB RAM (shared memory)
- **Between nodes:** 12 nodes connected by InfiniBand (distributed memory)

## Data Parallelism vs. Task Parallelism

### Data Parallelism

Same operation applied to different pieces of data:

```python
# Serial
for image in images:
    processed = apply_filter(image)

# Parallel: same operation, different data
with Pool(4) as p:
    results = p.map(apply_filter, images)
```

### Task Parallelism

Different tasks running simultaneously:

```bash
# Run 100 different simulations with different parameters
sbatch --array=1-100 my_simulation.sh
```

## Comparison: Which Type of Parallelism?

| Scenario | Best Approach | Why |
|----------|---------------|-----|
| 100 independent simulations | Job arrays | No communication needed |
| Processing 10,000 images | Job arrays | Independent files |
| Single simulation on 4 cores | multiprocessing | Shared memory |
| Large dataset analysis | Data parallelism | Divide data, apply function |
| Tightly-coupled physics | MPI | Needs frequent communication |

::::::::::::::::::::::::::::::::::::: challenge

**Categorizing Problems**

For each, identify: is it data parallel, task parallel, or tightly coupled?

1. Analyzing 1,000 gene sequences independently
2. Simulating fluid flow where each grid point depends on neighbors
3. Training 3 different ML algorithms on the same data

::::::::::::::::::::::::::::::::::::: solution

1. **Task parallel / embarrassingly parallel** -- each sequence is independent. Use job arrays.
2. **Tightly coupled** -- spatial dependencies between grid points. Use shared-memory or MPI.
3. **Task parallel** -- each algorithm trains independently. Use job arrays (only 3 tasks though).

::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints
- Shared-memory parallelism works within a node; distributed-memory across nodes
- Data parallelism applies the same operation to different data
- Task parallelism runs different tasks in parallel
- Most research problems are embarrassingly parallel at some level
::::::::::::::::::::::::::::::::::::::::::::::::
