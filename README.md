# Parallel Computing Fundamentals - Workshop 18

A comprehensive Carpentries Workbench workshop on parallel computing for researchers and students.

## Quick Start

This workshop teaches how to scale research computations from single-core to multi-core and multi-node environments.

### For Students
1. [Start here: Why Go Parallel?](episodes/01-why-parallel.md)
2. Follow episodes in order
3. Complete hands-on exercises
4. Parallelize your own code

### For Instructors
1. See [Instructor Notes](instructors/instructor-notes.md) for detailed teaching guidance
2. Use [Learner Profiles](learners/learner-profiles.md) to understand your audience
3. Review [Setup](setup.md) for classroom preparation
4. Check [Reference](reference.md) for quick syntax lookups

## Workshop Contents

### Episodes (7 total, ~12 hours)

1. **[Why Go Parallel?](episodes/01-why-parallel.md)**: Understanding speedup, efficiency, and Amdahl's Law
2. **[Types of Parallelism](episodes/02-types-of-parallelism.md)**: Shared-memory vs distributed-memory vs embarrassingly parallel
3. **[Embarrassingly Parallel Problems](episodes/03-embarrassingly-parallel.md)**: Parameter sweeps, Monte Carlo, batch processing
4. **[Running Job Arrays in SLURM](episodes/04-slurm-job-arrays.md)**: Practical job arrays with --array flag
5. **[Shared-Memory Parallelism](episodes/05-shared-memory.md)**: Python multiprocessing and R parallel
6. **[Distributed Computing with MPI](episodes/06-distributed-computing.md)**: Multi-node computing basics
7. **[Choosing Your Strategy](episodes/07-choosing-approach.md)**: Decision framework and best practices

### Supporting Materials

- **[Setup Guide](setup.md)**: Getting started on Sagehen
- **[Quick Reference](reference.md)**: Syntax and command reference
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

**Contact:** Andrew Wilson (awilson@pomona.edu)

## Acknowledgments

Developed with support from Pomona College High-Performance Computing team and based on the Carpentries teaching methodology.

---

**Ready to start?** Begin with [Episode 1: Why Go Parallel?](episodes/01-why-parallel.md)
