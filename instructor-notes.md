---
title: Instructor Notes
---

## Teaching Philosophy

This workshop is designed for researchers who want to scale up their research computations. Key principles:

1. **Practical over theoretical**: Focus on what researchers actually need to do
2. **Start simple**: Job arrays before shared memory before MPI
3. **Hands-on learning**: Code along with students, work through problems together
4. **Don't overload**: Better to cover less and do it well than rush through content

## Workshop Overview

- **Total duration:** 12 hours (typically delivered as 6 x 2-hour sessions or 3 full days)
- **Format:** Interactive lecture + hands-on exercises
- **Audience:** Researchers who run mostly single-core jobs; no parallel computing experience assumed
- **Outcomes:** Students should be able to identify and parallelize embarrassingly parallel problems using job arrays

## Detailed Episode Notes

### Episode 1: Why Go Parallel? (90 minutes teaching + 45 min exercises)

**Learning Objectives:**
- Understand speedup and efficiency
- Apply Amdahl's Law
- Recognize when parallelism helps

**Key Points:**
- Speedup = T1/Tp (total time on 1 core / total time on p cores)
- Efficiency = Speedup/p (should be > 50%)
- Amdahl's Law shows serial fraction limits speedup
- Even 5% serial code limits speedup to 20x on many cores

**Common Student Confusion:**
- "If I have 4 cores, I get 4x speedup" (not true! Amdahl's Law)
- CPU time vs wall-clock time (clarify: CPU time = sum of all work; wall-clock = what the user waits)
- "My code runs 10x but uses 4 cores" (efficiency is 2.5x, calculate correctly)

**Teaching Tips:**
- Use visuals for speedup/efficiency (show bars, graphs)
- Have students calculate Amdahl's Law for their own problems
- Emphasize: "Profile first, don't parallelize prematurely"
- Real example: A researcher profiling their code and finding serialization they didn't expect

**Exercise Tips:**
- Answers for Amdahl's Law calculations are provided in the episode
- Have students discuss their codes: Which parts are parallel? Estimate serial fraction.
- Don't worry if students don't get exact numbers; focus on understanding the concept

### Episode 2: Types of Parallelism (90 minutes)

**Learning Objectives:**
- Distinguish shared-memory vs distributed-memory
- Recognize embarrassingly parallel problems
- Understand data parallelism vs task parallelism

**Key Points:**
- **Shared memory:** Multiple cores, same memory (fast, limited to one node)
- **Distributed memory:** Multiple nodes, separate memory (scalable, complex)
- **Embarrassingly parallel:** No communication needed (easiest!)
- **Data parallel:** Same operation, different data
- **Task parallel:** Different operations, can be independent or dependent

**Common Student Confusion:**
- "Shared memory means all cores see everything instantly" (not quite; there's cache coherency complexity)
- "Embarrassingly parallel is a bad thing" (it's actually great!)
- "Why not always use MPI?" (communication overhead, complexity)

**Teaching Tips:**
- Draw diagrams of memory architectures
- Give examples from their domain (biology, chemistry, etc.)
- Emphasize embarrassingly parallel as "free parallelism"
- Show the decision table: "Job arrays are your first choice"

**Exercise Tips:**
- Have students classify their own problems
- Discuss trade-offs: simplicity vs. scalability
- Don't go into MPI theory here; save for Episode 6

### Episode 3: Embarrassingly Parallel Problems (120 minutes)

**Learning Objectives:**
- Recognize embarrassingly parallel patterns
- Convert serial code to embarrassingly parallel
- Structure code for independent tasks

**Key Points:**
- Parameter sweeps: Same code, different inputs
- Monte Carlo: Many independent random trials
- Batch processing: Process many files independently
- Resampling: Bootstrap, permutation tests

**Common Student Confusion:**
- "How do I handle dependencies?" (You don't:that's not embarrassingly parallel!)
- "What if the tasks take different times?" (Load balancing: job scheduler handles it)
- "Can I use MPI for this?" (You could, but job arrays are simpler)

**Teaching Tips:**
- Work through parameter sweep example step-by-step
- Show how to convert serial code (loop) to parallel (array indices)
- Emphasize unique output filenames
- Show aggregation (combining results at end)

**Live Coding:**
- Have students write a simple embarrassingly parallel script
- Test it serially first, then submit as job array
- Monitor with `squeue` and show job statistics

**Exercise Tips:**
- Start with simple Monte Carlo pi estimation
- Have students work in pairs
- Provide starter code if needed; focus on structure, not algorithm

### Episode 4: SLURM Job Arrays (120 minutes)

**Learning Objectives:**
- Submit job arrays with --array flag
- Use SLURM_ARRAY_TASK_ID in scripts
- Map indices to parameters
- Monitor and manage job arrays

**Key Points:**
- `sbatch --array=1-100 script.sh` submits 100 jobs
- Each job knows its index via `$SLURM_ARRAY_TASK_ID`
- Use task ID to select parameter/input file
- Save output with unique filenames
- Monitor with `squeue -j <job_id>`

**Demonstration:**
1. Show job array submission
2. Show squeue output (job array notation)
3. Show individual job logs
4. Cancel part of array, requeue failed tasks
5. Show sacct for performance statistics

**Common Student Mistakes:**
1. Forgetting `--array` flag (submits one job)
2. All jobs writing to same output file (last one wins!)
3. Not using `$SLURM_ARRAY_TASK_ID` (jobs are identical)
4. Off-by-one error (array IDs are 1-indexed, but many languages use 0-indexing)

**Live Demo:**
- Submit a simple job array while students watch
- Show real-time monitoring
- Explain output notation (12345_[1-100], 12345_5, etc.)

**Exercises:**
1. Simple job array: Generate random numbers with different seeds
2. Parameter sweep: Run simulation with varying parameters
3. Monitoring: Check job status, cancel, requeue

**Advanced Topics (if time):**
- Dependency chains: `--dependency=afterok:12345`
- Limit concurrent jobs: `--array=1-1000%10` (max 10 at a time)
- Custom mapping: Using parameter files instead of array indices

### Episode 5: Shared-Memory Parallelism (120 minutes)

**Learning Objectives:**
- Use Python multiprocessing.Pool
- Use R parallel functions
- Request multiple cores with SLURM
- Understand when shared-memory is appropriate

**Key Points:**
- Use Pool(n) for n parallel processes
- Map function over data with pool.map()
- Request cores with `#SBATCH --cpus-per-task=N`
- Avoid shared mutable state (don't modify same variable)
- Good for single-node parallelism

**Common Student Confusion:**
- Threads vs. processes (Python threading has GIL; use processes)
- How to set number of processes (use $SLURM_CPUS_PER_TASK)
- Race conditions (don't modify shared variables)
- When to use shared-memory vs. job arrays (shared-memory for data-parallel on one node)

**Demonstration:**
1. Simple pool example: Square numbers
2. File processing example: Read multiple files in parallel
3. Show speedup with different pool sizes
4. Show SLURM resource request and verification

**Live Coding:**
```python
from multiprocessing import Pool
import os

def work(item):
    return item * 2

if __name__ == "__main__":
    n_cores = int(os.environ.get('SLURM_CPUS_PER_TASK', 1))
    with Pool(n_cores) as p:
        results = p.map(work, range(1000))
    print(f"Processed {len(results)} items with {n_cores} cores")
```

**Exercises:**
1. Parallelize file processing (read 100 files)
2. Parallelize Monte Carlo simulation (run 1000 samples in parallel)
3. Compare to serial and to job array approach

**Discussion Points:**
- When is shared-memory better than job arrays? (When you need to communicate within a single batch of work)
- When is it worse? (When tasks are embarrassingly parallel)
- Communication patterns: What data sharing looks like

### Episode 6: Distributed Computing (90 minutes)

**Learning Objectives:**
- Understand MPI concepts conceptually
- Know when MPI is appropriate
- Run simple MPI program with srun
- Understand mpi4py basics

**Key Points:**
- MPI for tightly-coupled multi-node problems
- Each process (rank) has its own memory
- Explicit message passing (send/recv)
- Launch with `srun` not `mpirun`
- InfiniBand makes MPI practical on Sagehen

**Important Notes:**
- **This is conceptual introduction**: Not a full MPI programming course
- Focus on when MPI is needed, not deep programming
- mpi4py examples show syntax, not complex algorithms

**Common Student Confusion:**
- "Why not always use MPI?" (communication overhead, complexity)
- "How is MPI different from job arrays?" (MPI has tight coupling and shared memory; job arrays are loose)
- "Does my problem need MPI?" (Usually not! Only if frequent communication across nodes)

**Demonstration:**
1. MPI hello world: Show how processes are numbered (rank)
2. Simple communication: Send/receive between ranks
3. Collective operations: Broadcast, reduce
4. Show performance limitation: Communication overhead grows with node count

**MPI Programs Provided:**
- hello.py: Print rank information
- distributed_sum.py: Distribute computation across ranks
- Integration example: Domain decomposition

**Assessment:**
- Can students identify when MPI is needed?
- Do they understand rank/communicator concepts?
- Can they launch an MPI program?

**Avoid:**
- Deep MPI programming (too complex)
- Performance optimization of MPI
- Advanced features (non-blocking communication, custom datatypes)

### Episode 7: Choosing Your Strategy (120 minutes)

**Learning Objectives:**
- Apply decision framework to real problems
- Profile and benchmark code
- Understand scaling tests
- Recognize premature optimization

**Key Points:**
- Decision tree: Profile first, then choose approach
- Job arrays > shared-memory > MPI (in order of preference)
- Don't parallelize what you can't measure
- Scaling tests validate approach before large runs
- Efficiency > speedup (diminishing returns)

**Teaching Approach:**
- Give realistic problems, have students decide approach
- Discuss pros/cons of each choice
- Show decision tree
- Emphasize: "Profile first, optimize later"

**Case Studies:**
1. **Parameter sweep (100 simulations, 30s each)** → Job arrays (obvious!)
2. **Large matrix operations on one node** → Use optimized libraries (numpy), then shared-memory if needed
3. **Molecular dynamics (1M particles, 1000 steps)** → Shared-memory on one node
4. **Climate simulation (multi-node)** → MPI or use existing package

**Exercises:**
1. Given a code snippet and timing, what's the bottleneck?
2. Measure speedup from toy parallel program
3. Design a scaling test for their own problem

**Live Examples:**
- Profile a simple program: `cProfile` in Python
- Run benchmark: 1 core, 2 cores, 4 cores, 8 cores
- Calculate efficiency at each step
- Plot and discuss

**Final Discussion:**
- What's the simplest approach that will work?
- How do you validate it?
- When to ask for help (its-hpc@pomona.edu)
- Where to go for deeper learning

---

## Facilitation Tips

### Creating Safe Space
- No stupid questions in parallel computing!
- Emphasize: "We all started here"
- Share own learning struggles
- If you don't know answer: "Let's find out together"

### Pacing
- Watch for confusion, slow down if needed
- Don't get lost in mathematical details (Amdahl's Law is concept, not equation)
- If ahead of schedule: Dig deeper with interested students
- If behind: Skip advanced topics, focus on core concepts

### Hands-On Work
- Live code along with students (makes mistakes visible and normal)
- Have students type commands, don't just watch
- Pause frequently: "Try this yourself, I'll wait 2 minutes"
- Have TAs/helpers ready to help stuck students

### Troubleshooting Common Issues

**"My job is in queue but not running"**
- Check node availability: `sinfo`
- Check memory/core requests: Are they reasonable?
- Job priority: Check with `sprio`
- Contact HPC team if cluster is down

**"My parallel code is slower than serial"**
- Likely communication overhead exceeds benefit
- Measure: Is overhead actually the bottleneck?
- Try larger problem size (overhead becomes smaller fraction)
- Accept: Not everything should be parallelized

**"I get different results parallel vs. serial"**
- Check random seeds (must be different per task)
- Floating point rounding (slightly different order of operations)
- True bugs: Shared mutable state, race conditions
- Have students debug step-by-step

**"Scheduler is full, my job is waiting"**
- This is normal! Remind students: "Shared resource"
- Use smaller job array for testing
- Check priority: `sprio -j <job_id>`
- Suggest: Come back later to check results

### Assessment

Formative assessment (throughout):
- "Can you sketch a decision tree?"
- "What would you parallelize first?"
- "How would you measure speedup?"
- Live exercises with immediate feedback

Summative assessment (end of workshop):
- Student chooses their own code to parallelize
- Profiles it, identifies bottleneck
- Implements parallelization
- Measures speedup and efficiency
- Presents approach and results

---

## Timing Guide (for 6 x 2-hour sessions)

### Session 1 (Episodes 1-2): Why Parallel? + Types
- 0:00-0:15: Welcome, objectives
- 0:15-1:00: Episode 1 (lecture + interaction)
- 1:00-1:30: Episode 1 exercises
- 1:30-2:00: Episode 2 (lecture + interaction)

*Homework:* Identify own problem type

### Session 2 (Episode 3): Embarrassingly Parallel
- 0:00-0:10: Review from Session 1
- 0:10-1:30: Episode 3 (lecture + live coding)
- 1:30-2:00: Exercises

*Homework:* Sketch code structure

### Session 3 (Episode 4): Job Arrays
- 0:00-0:10: Review
- 0:10-1:00: Episode 4 (lecture + demo)
- 1:00-1:30: Live submission + monitoring
- 1:30-2:00: Exercises

*Homework:* Submit job array to Sagehen

### Session 4 (Episode 5): Shared Memory
- 0:00-0:10: Review
- 0:10-1:00: Episode 5 (lecture + live coding)
- 1:00-1:30: Demonstrate Pool and scaling
- 1:30-2:00: Exercises

*Homework:* Write multiprocessing script

### Session 5 (Episode 6): MPI
- 0:00-0:10: Review
- 0:10-0:50: Episode 6 (conceptual, demos)
- 0:50-1:20: Show MPI on Sagehen
- 1:20-2:00: Exercises (simple examples)

*Homework:* Understand when MPI applies

### Session 6 (Episode 7): Choosing Approach
- 0:00-0:10: Review all episodes
- 0:10-1:00: Decision framework, case studies
- 1:00-1:30: Student presentations (optional)
- 1:30-2:00: Q&A, next steps

---

## Resources for Instructors

### Setting Up Lab Environment
- Create shared directory with example scripts
- Pre-generate sample data (to avoid I/O delays)
- Test all examples before workshop
- Have backup plans (demo slides if cluster is down)

### Recommended Demo Scripts
- Monte Carlo pi estimation (shows reproducibility, random seeds)
- Parameter sweep (shows job array use)
- File processing (batch parallelism)
- Scaling benchmark (shows efficiency degradation)

### Further Reading
- "Introduction to HPC" (XSEDE tutorials)
- "Performance Tuning of Scientific Applications" (Chapman, Hanxleden)
- HPC documentation for your cluster
- Research papers from your students' fields

### Getting Help
- SLURM documentation: https://slurm.schedmd.com/
- Supercomputing centers: XSEDE, NERSC
- Local HPC team: its-hpc@pomona.edu

---

## Feedback and Iteration

This is a **pre-alpha workshop**. Your feedback improves it!

### What to Watch For
- Which concepts confuse students most?
- Which examples resonate best?
- How much time does each episode actually take?
- What terminology is unclear?
- What's missing?

### How to Contribute
- Submit issues: https://github.com/Pomona-College/hpc-parallel-computing/issues
- Pull requests welcome
- Contact: andrew.wilson@pomona.edu

---

**Last Updated:** March 6, 2026
**Contact:** Andrew Wilson (andrew.wilson@pomona.edu)
