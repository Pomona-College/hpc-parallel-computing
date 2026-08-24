---
title: Learner Profiles
---

## Typical Participants

### Profile 1: Computational Physicist Seeking Speedup
**Name:** Dr. Alex Chen
**Background:** 3 years postdoctoral research in quantum chemistry

**Motivation:**
- Running density functional theory (DFT) simulations on molecular structures
- Each simulation takes 6-8 hours
- Has 50 different molecular configurations to test for publication
- Current approach: 400+ hours of compute time (running serially overnight)

**Experience:**
- Proficient in Python and some Fortran
- Submits SLURM jobs but only single-core batches
- No experience with parallelization
- Familiar with Linux command line

**Pain Point:**
"I have 50 molecules to test, and running them serially would take 2+ weeks. My advisor wants results by next month."

**Expected Outcome After Workshop:**
- Recognize that 50 independent simulations are embarrassingly parallel
- Use SLURM job arrays to run 25 simulations simultaneously
- Reduce total runtime from 400 hours to 16 hours (running parallel)
- Understand how to monitor and manage job arrays on Sagehen

**Preference:** Will use command-line SSH with job arrays (prefers shell scripts)

---

### Profile 2: Bioinformatics Pipeline Developer
**Name:** Jordan Martinez
**Background:** 2nd year graduate student in computational biology

**Motivation:**
- Processing 500 genomic sequences for a genome-wide association study
- Current pipeline: 15 minutes per sequence = 125 hours total
- Need to parallelize for dissertation timeline
- May have to process 1000+ sequences in future

**Experience:**
- Strong Python and R programmer
- Recently learned SLURM basics
- Built processing pipeline locally (runs on laptop)
- No experience scaling to HPC resources

**Pain Point:**
"My advisor told me to just 'parallelize it,' but I don't know how. Do I need a completely different language?"

**Expected Outcome After Workshop:**
- Understand that not every parallel problem needs rewriting
- Adapt existing pipeline for SLURM job arrays or multiprocessing
- Process 500 sequences in ~1-2 hours instead of 125 hours
- Learn how to test and validate parallel pipeline produces correct results

**Preference:** Will use job arrays with Python scripts, may experiment with R parallel package

---

### Profile 3: Data Scientist with Embarrassingly Parallel Workload
**Name:** Casey Brooks
**Background:** Research scientist in computational sociology

**Motivation:**
- Statistical analysis on 10,000 survey responses
- Each analysis model takes 30 minutes to 2 hours
- Wants to run multiple model configurations in parallel
- Goal: Complete all models in 1 week instead of several months

**Experience:**
- Excellent with R and statistical methods
- Used R's `parallel` package locally on 8-core laptop
- Submits basic SLURM jobs
- New to large-scale HPC

**Pain Point:**
"I know how to parallelize on my laptop, but how do I scale to 128 cores on Sagehen without rewriting everything?"

**Expected Outcome After Workshop:**
- Understand how to scale R's parallel package to a full Sagehen node (128 cores)
- Respect SLURM CPU allocation using `$SLURM_CPUS_PER_TASK`
- Run multiple statistical models simultaneously
- Monitor and optimize CPU usage on a multi-core node

**Preference:** Will use shared-memory parallelism with R, may also use Python multiprocessing

---

### Profile 4: Computer Science Student Learning HPC
**Name:** Morgan Lee
**Background:** Senior undergrad, first HPC research project

**Motivation:**
- Writing a parallel implementation of a Monte Carlo algorithm for independent study
- Wants to understand parallel computing theory and practice
- Plans to learn systems programming and HPC for graduate school
- Need hands-on experience with real HPC infrastructure

**Experience:**
- Comfortable with C/C++ and Python
- Took algorithms course (knows Big O notation)
- Just completed intro HPC workshop
- No parallel programming experience

**Pain Point:**
"I learned SLURM basics, but parallel computing is abstract. I need to see it work on real hardware."

**Expected Outcome After Workshop:**
- Understand three main parallelization strategies
- Recognize when each strategy is appropriate
- Write and debug simple MPI program on Sagehen
- Measure speedup and understand Amdahl's Law
- Know where to go for deeper HPC study

**Preference:** Will use job arrays, Python multiprocessing, and simple MPI examples

---

### Profile 5: Materials Science Researcher Needing Multi-Node Scaling
**Name:** Riley Patel
**Background:** PhD student in computational materials science

**Motivation:**
- Molecular dynamics (MD) simulations of crystal structures
- Tightly-coupled simulations requiring inter-process communication
- Problem size requires 4-8 compute nodes (512-1024 cores)
- Currently running on desktop (2 cores, impractical)

**Experience:**
- Proficient in C++ and some Python
- Some MPI exposure from HPC seminar
- Comfortable with compilation and debugging
- Has run SLURM jobs but only single-node

**Pain Point:**
"MPI documentation is overwhelming. Job arrays won't work for my simulation because processes need to communicate. How do I scale to multiple nodes?"

**Expected Outcome After Workshop:**
- Understand why MPI is necessary for their problem (vs. job arrays)
- See practical MPI example on Sagehen
- Know how to structure SLURM job for multi-node MPI
- Understand synchronization barriers and collective operations
- Know where to get advanced MPI help (documentation, support, HPC center)

**Preference:** Will focus on MPI + multi-node SLURM job configuration

---

## Common Learner Characteristics

### What They Know
- Basic Unix/Linux command line
- How to submit and monitor SLURM jobs
- How to write scripts or code in at least one language
- What parallelization means in general

### What They Don't Know
- How to choose between job arrays, multiprocessing, and MPI
- How to measure speedup and scaling efficiency
- How to adapt their existing code for parallelization
- Amdahl's Law and its practical implications
- How to avoid common parallel programming bugs

### What They Value
- **Practical Results:** Want to solve their research problem, not just learn theory
- **Speed to Productivity:** Can't afford weeks of learning, need working code soon
- **Reliability:** Results must be correct and reproducible
- **Examples:** Learn by doing, not just lecture

### What They Worry About
- **"Is this too complicated?"** "Will I understand this?"
- **"Will my code be fast?"** "What if parallelization doesn't help?"
- **"Will my results be correct?"** "How do I verify parallel results match serial?"
- **"Do I need to rewrite everything?"** "Can I use my existing code?"

---

## Learner Diversity

### By Research Field
- Physics, Chemistry, Materials Science (simulation-heavy, often MPI)
- Biology, Bioinformatics (parameter sweeps, often job arrays)
- Data Science, Statistics (statistical models, often multiprocessing)
- Computer Science (variety of approaches)

### By Career Stage
- Undergraduates (learning first parallel systems)
- Graduate students (need to parallelize dissertation research)
- Postdocs (scaling up research for publication)
- Faculty (optimizing lab workflows)

### By Programming Preference
- Python (most common)
- R (bioinformatics, statistics)
- C/C++/Fortran (physics, materials)
- MATLAB (some engineering)

---

## Design Philosophy

This workshop is designed for **researchers who have working code and want to make it faster**, not for those learning programming basics.

Key assumptions:
- Participants can write and debug code
- Participants have SLURM basics (from intro workshop)
- Participants have real parallelization needs
- Participants value practical examples over theory

The workshop emphasizes:
- **"How do I parallelize MY code?"** not "what is parallelization?"
- **"Which approach is right for MY problem?"** not "three approaches exist"
- **"Is it actually faster?"** via speedup measurement
- **"What can go wrong?"** via common pitfalls

By day's end, participants should have either:
1. A working parallel version of their research code, OR
2. Clear understanding of what approach to use for their specific problem

