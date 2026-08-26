# Parallel Computing Fundamentals - Workshop 18

A comprehensive Carpentries Workbench workshop on parallel computing for researchers and students.

## Quick Start

This workshop teaches how to scale research computations from single-core to multi-core and multi-node environments.

### For Students
1. [Start here: Why Parallel Computing?](episodes/01-why-parallel-computing.md)
2. Follow episodes in order
3. Complete hands-on exercises
4. Parallelize your own code

### For Instructors
1. See [Instructor Notes](instructors/instructor-notes.md) for detailed teaching guidance
2. Use [Learner Profiles](learners/learner-profiles.md) to understand your audience
3. Review [Setup](learners/setup.md) for classroom preparation
4. Check [Reference](learners/reference.md) for quick syntax lookups

## Workshop Contents

### Episodes (12 total, ~6 hours)

1. **[Why Parallel Computing?](episodes/01-why-parallel-computing.md)**: Speedup, efficiency, and when parallelism pays off
2. **[Amdahl's Law and Speedup](episodes/02-amdahls-law.md)**: Amdahl's Law and the limits of speedup
3. **[Types of Parallelism](episodes/03-types-of-parallelism.md)**: Shared-memory vs distributed-memory vs embarrassingly parallel
4. **[Embarrassingly Parallel Problems](episodes/04-embarrassingly-parallel.md)**: Parameter sweeps, Monte Carlo, batch processing
5. **[GNU Parallel and Shell Parallelism](episodes/05-gnu-parallel.md)**: GNU Parallel and shell-level parallelism
6. **[SLURM Job Arrays](episodes/06-slurm-job-arrays.md)**: Practical job arrays with the --array flag
7. **[Advanced Job Arrays](episodes/07-advanced-job-arrays.md)**: Throttling, dependencies, and array patterns
8. **[Shared Memory with OpenMP](episodes/08-shared-memory-openmp.md)**: OpenMP and multi-core work on one node
9. **[Threading in Python and R Parallelism](episodes/09-threading-python.md)**: Python multiprocessing and R parallel
10. **[Distributed Computing Concepts](episodes/10-distributed-concepts.md)**: Multi-node computing concepts
11. **[MPI Basics](episodes/11-mpi-basics.md)**: MPI fundamentals on Sagehen HPC
12. **[Choosing the Right Approach](episodes/12-choosing-approach.md)**: Decision framework and best practices

### Supporting Materials

- **[Setup Guide](learners/setup.md)**: Getting started on Sagehen
- **[Quick Reference](learners/reference.md)**: Syntax and command reference
- **[Instructor Notes](instructors/instructor-notes.md)**: Detailed teaching guidance
- **[Learner Profiles](learners/learner-profiles.md)**: Who we're teaching

## Cluster Information

This workshop uses **Sagehen** HPC cluster at Pomona College:

- **12 AMD EPYC nodes** with 128 cores each
- **512GB memory** per node
- **InfiniBand 100Gb/s** interconnect
- **SLURM** job scheduler
- **Lustre** parallel filesystem

Concepts are universal and apply to most HPC clusters.

## Learning Outcomes

By the end of this workshop, you will:

- Understand fundamental parallel computing concepts
- Identify which problems benefit from parallelization
- Use SLURM job arrays for embarrassingly parallel tasks
- Implement shared-memory parallelism on a single node
- Know when distributed computing (MPI) is appropriate
- Choose the right parallelization approach for your problem
- Measure and optimize parallel performance

## Prerequisites

- Access to Sagehen or equivalent HPC cluster
- SSH client for terminal access
- Basic Linux command-line knowledge
- Text editor
- Familiarity with submitting batch jobs (helpful but not required)

## Getting Help

- **Email:** its-hpc@pomona.edu
- **Issues:** https://github.com/Pomona-College/hpc-parallel-computing/issues
- **Discussions:** https://github.com/Pomona-College/hpc-parallel-computing/discussions

## Contributing

This is an open, community-contributable workshop. We welcome:

- Bug reports and feature requests
- Content improvements and clarifications
- Additional examples from your research domain
- Translations and adaptations for other clusters

See [Contributing](CONTRIBUTING.md) (if present) or submit issues on GitHub.

## Citation

If you use this workshop, please cite:

```
Wilson, A. (2026). Parallel Computing Fundamentals: Workshop 18.
Pomona College High-Performance Computing Team.
https://github.com/Pomona-College/hpc-parallel-computing
```

## License

This work is licensed under a Creative Commons Attribution 4.0 International License (CC-BY 4.0).

You are free to:
- Share and redistribute the material
- Adapt and build upon it

For any use, you must give appropriate credit.

## Status

**Current Phase:** Pre-alpha (actively developing based on feedback)

**Last Updated:** March 6, 2026

**Contact:** Andrew Wilson (andrew.wilson@pomona.edu)

## Acknowledgments

**Andrew Wilson** — Director of Research Computing and Digital Scholarship,
Pomona College. Workshop design and development.

**Andrei Motchenko** — testing, editing, cleanup and screenshots across the
Pomona College HPC Workshop Series.

Developed with support from Pomona College High-Performance Computing team and based on the Carpentries teaching methodology.

---

**Ready to start?** Begin with [Episode 1: Why Parallel Computing?](episodes/01-why-parallel-computing.md)
