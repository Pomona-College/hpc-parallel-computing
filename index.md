---
title: Parallel Computing Fundamentals
subtitle: Scaling your research from single-core to multi-core and multi-node computing
type: standard
---

## Welcome

This workshop introduces you to parallel computing concepts and practical techniques for scaling your research computations from single-core to multi-core and multi-node environments. Whether you're running parameter sweeps, processing large datasets, or solving computationally intensive problems, this material will help you understand when and how to parallelize your work.

### What You'll Learn

By the end of this workshop, you will:

- Understand the fundamental concepts of parallel computing and when parallelism helps your research
- Distinguish between different types of parallelism (embarrassingly parallel, shared-memory, and distributed-memory)
- Write and submit SLURM job arrays for parameter sweeps and batch processing
- Use shared-memory parallelism with Python and R on a single node
- Understand distributed computing concepts and when MPI is appropriate
- Choose the right parallelization strategy for your specific problem

### Target Audience

This workshop is designed for researchers and students who:

- Currently run mostly single-core jobs
- Want to scale up their computations
- Have basic Linux command-line knowledge
- Are interested in practical, hands-on applications

No prior parallel computing experience is required.

### Workshop Structure

This workshop consists of 7 episodes covering:

1. **Why Go Parallel?**: Understanding speedup, efficiency, and when parallelism helps
2. **Types of Parallelism**: Shared-memory vs distributed-memory vs embarrassingly parallel
3. **Embarrassingly Parallel Problems**: Recognizing and exploiting independent tasks
4. **Running Job Arrays in SLURM**: Practical parameter sweeps and batch processing
5. **Shared-Memory Parallelism**: Threading and multiprocessing on a single node
6. **Distributed Computing with MPI**: Multi-node computing and message passing basics
7. **Choosing Your Strategy**: Decision-making framework and avoiding premature optimization

::::: prereq

### Prerequisites

- Access to Sagehen HPC cluster (or equivalent SLURM-based system)
- SSH client for terminal access
- Text editor of your choice
- Basic Linux command-line experience (cd, ls, cat, mkdir, etc.)
- Familiarity with submitting simple batch jobs (optional but helpful)

::::::

### The Sagehen Cluster

Examples in this workshop use the **Sagehen** HPC cluster at Pomona College:

- **12 AMD EPYC nodes** with 128 cores each (1,536 total cores)
- **InfiniBand 100Gb/s** interconnect for fast communication between nodes
- **SLURM** job scheduler for resource allocation and job management
- **Lustre** parallel file system for high-performance I/O

**Concepts are universal:** While we use Sagehen for examples, the parallel computing principles and SLURM syntax apply to nearly all modern HPC clusters. Adapt syntax and job parameters to your specific cluster's configuration.

### Open Science Commitment

This workshop is **open and community-contributable**. We encourage:

- Bug reports and feature requests
- Content improvements and clarifications
- Additional examples and use cases
- Translations and adaptations for other clusters

See [Contributing](#contributing) below.

### Quick Start

1. Log into Sagehen: `ssh <username>@sagehen.hpc.pomona.edu`
2. Go through the episodes in order
3. Complete the exercises and code-alongs
4. Refer to the reference section for syntax reference

### Support

- **Email:** its-hpc@pomona.edu
- **Issues:** https://github.com/Pomona-College/hpc-parallel-computing/issues
- **Discussions:** https://github.com/Pomona-College/hpc-parallel-computing/discussions

### Acknowledgments

This workshop was developed with support from the High-Performance Computing team at Pomona College. Special thanks to the Carpentries community for providing the workshop template and pedagogical framework.

### License

This work is licensed under a Creative Commons Attribution 4.0 International License ([CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/)).

You are free to:
- Share: copy and redistribute the material
- Adapt: remix, transform, and build upon the material

For any use, you must give appropriate credit and indicate if changes were made.

---

### Contributing

We welcome contributions! To contribute:

1. Fork the repository: https://github.com/Pomona-College/hpc-parallel-computing
2. Create a branch for your changes
3. Make edits and test locally
4. Submit a pull request with a clear description of changes

See the [Instructor Notes](instructors/instructor-notes.md) for guidance on workshop structure and pedagogy.

---

**Current Status:** Pre-alpha  
**Last Updated:** March 6, 2026  
**Contact:** Andrew Wilson (andrew.wilson@pomona.edu)

## Acknowledgments

Developed by **Andrew Wilson**, Director of Research Computing and Digital
Scholarship at Pomona College, with **Andrei Motchenko**, who tested, edited
and produced screenshots for the workshop series.
