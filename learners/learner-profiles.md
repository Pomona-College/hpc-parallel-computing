---
title: Learner Profiles
---

## Typical Participants

### Profile 1: Research-Focused Postdoc
**Name:** Alex  
**Background:** 3 years postdoctoral research in chemistry

**Motivation:**
- Running quantum chemistry simulations that take 5+ hours each
- Need to test many molecular configurations
- Wants faster turnaround for publication

**Experience:**
- Writes code in Python and MATLAB
- Submits basic SLURM jobs (single-core)
- No parallel programming experience

**Pain Point:**
- "I have 100 different molecules to test"
- "Running them serially would take 500 hours"

**Expected Outcome:**
- Use job arrays to test 100 molecules simultaneously
- Reduce 500 hours to 5 hours

---

### Profile 2: Graduate Student
**Name:** Jordan  
**Background:** 2nd year graduate student in biology

**Motivation:**
- Master's thesis: analyze 500 genetic sequences
- Each sequence takes 15 minutes
- Wants to parallelize processing

**Experience:**
- Writes R and Python code
- Familiar with basic SLURM
- No parallel programming experience

**Pain Point:**
- "Processing all 500 sequences would take 125 hours serially"
- "Advisor said 'just parallelize it' but I don't know how"

**Expected Outcome:**
- Convert R script to process one sequence per job
- Use job arrays for all 500 sequences
- Reduce runtime from 125 hours to ~1 hour

---

### Profile 3: Undergraduate Researcher
**Name:** Morgan  
**Background:** Senior undergrad, first research experience

**Motivation:**
- Physics research: Monte Carlo simulations
- Currently running single simulation per night
- Needs multiple runs to publish

**Experience:**
- Recently learned Python
- New to HPC
- Limited systems programming experience

**Pain Point:**
- "I need 100 simulations. That's 1000 hours!"
- "I don't know what parallelization means"

**Expected Outcome:**
- Understand parallel computing concepts
- Recognize Monte Carlo as embarrassingly parallel
- Use job arrays to run 100 simulations simultaneously
- Reduce runtime to ~10 hours

---

### Profile 4: Data Scientist
**Name:** Casey  
**Background:** Computational sociology, large datasets

**Motivation:**
- Processing 10,000 surveys with complex statistical models
- Each model takes 2 hours
- Run different configurations in parallel

**Experience:**
- Strong in R and statistical methods
- Used parallel package locally (8 cores max)
- Want to scale to 128-core node

**Pain Point:**
- "How do I use all 128 cores on Sagehen?"

**Expected Outcome:**
- Use R's parallel package with SLURM
- Run multiple models simultaneously
- Reduce runtime significantly

---

### Profile 5: Systems-Oriented Researcher
**Name:** Riley  
**Background:** Physics PhD, computational materials

**Motivation:**
- Large-scale molecular dynamics simulations
- Problem has tight coupling
- Needs multiple nodes

**Experience:**
- Familiar with compiled languages (C++)
- Some MPI exposure from coursework
- Wants to scale to 8+ nodes

**Pain Point:**
- "Job arrays are for independent tasks"
- "My simulation has tight coupling"
- "MPI documentation is overwhelming"

**Expected Outcome:**
- Understand when MPI is necessary
- See MPI example on Sagehen
- Know where to go for deeper learning

---

## Common Characteristics

### What They Know
- Basic Linux command line
- Running batch jobs on HPC
- Writing code
- Time pressure

### What They Don't Know
- How to parallelize
- Choosing between approaches
- Measuring speedup
- SLURM arrays, multiprocessing, or MPI

### What They Value
- Simplicity
- Speed to results
- Reliability
- Practical examples

### What They Fear
- "Parallelization is too complicated"
- "I'll waste time debugging"
- "My results will be wrong"
- "I need a new language"

---

**Designed for:** Researchers across science and engineering scaling up computations
