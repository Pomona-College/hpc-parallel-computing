---
title: Setup
---

## Pre-Workshop Requirements

Before the workshop, please ensure you have the following set up and working.

### 1. Active Sagehen HPC Account

You must have a valid Pomona College HPC account with access to the Sagehen cluster.

**To verify your access:**
```bash
ssh <myusername>@sagehen.hpc.pomona.edu
```

If you don't have an account or can't log in, contact **its-hpc@pomona.edu**.

### 2. Authentication Setup

Sagehen uses Pomona Active Directory and DUO MFA.

::::::::::::::::::::::::::::::::::::: callout

### Setting Up Passwordless SSH (Recommended)

Generate an SSH key pair on your local machine (if you don't have one):

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_sagehen
```

Copy your public key to Sagehen:

```bash
ssh-copy-id -i ~/.ssh/id_sagehen <myusername>@sagehen.hpc.pomona.edu
```

Add to your `~/.ssh/config` to simplify logins:

```
Host sagehen
    HostName sagehen.hpc.pomona.edu
    User your_username
    IdentityFile ~/.ssh/id_sagehen
```

Then you can simply: `ssh sagehen`

:::::::::::::::::::::::::::::::::::::::::::::::

### 3. Basic SLURM Knowledge

Familiarity with basic SLURM commands is expected. If you need a refresher:

```bash
# View cluster info
sinfo

# Submit a job
sbatch my_script.sh

# Check your jobs
squeue -u $USER

# Cancel a job
scancel JOB_ID
```

If you're new to SLURM, complete the "Introduction to HPC Systems" workshop first.

### 4. Load Required Modules

The workshop uses Python, C/C++, and OpenMPI. Test that you can load these modules:

```bash
module avail python
module avail gcc
module avail openmpi
```

Load and test Python:

```bash
module load miniconda3
python --version
```

Load and test C/C++ compiler:

```bash
# gcc is available system-wide on Sagehen -- no module load needed
gcc --version
```

Load and test OpenMPI:

```bash
module load openmpi
mpicc --version
```

If you see module not found errors, contact **its-hpc@pomona.edu**.

### 5. Test MPI Environment

To verify OpenMPI works, run this simple test:

```bash
# Login to Sagehen
ssh sagehen

# Create a test script
cat > test_mpi.sbatch << 'EOF'
#!/bin/bash
#SBATCH --job-name=test_mpi
#SBATCH --ntasks=4
#SBATCH --time=00:05:00

module load openmpi

# Print node and rank info
srun hostname -s
srun echo "My MPI rank: $OMPI_COMM_WORLD_RANK"
EOF

# Submit it
sbatch test_mpi.sbatch

# Check output
squeue -u $USER
```

### 6. Cluster Resources Available

Sagehen has the following resources available during the workshop:

- **AMD Nodes:** 12 nodes with 128 cores each, 512GB RAM
- **GPU Nodes (10 GPUs total, confirmed May 2026):**
  - 4× NVIDIA A100 80 GB
  - 4× NVIDIA L40S 48 GB
  - 2× NVIDIA RTX PRO 6000 Blackwell 96 GB
- **Storage:**
  - `/rhome` (100GB personal)
  - `/bigdata` (1TB lab, shared)
  - `/scratch` (SSD temp)
  - `/tmpfs` (RAM temp)
- **Network:** InfiniBand 100Gb/s
- **Partitions:** `amd` (default), `gpu`, `short`

### 7. Optional: Install mpi4py (For Python MPI)

If you plan to use Python for MPI:

```bash
module load miniconda3
pip install mpi4py
```

### 8. Sagehen Access Methods

You can access Sagehen three ways:

1. **SSH Command Line** (recommended for this workshop)
   ```bash
   ssh sagehen.hpc.pomona.edu
   ```

2. **OnDemand Web Interface**
   https://ondemand.hpc.pomona.edu/

3. **Local Development + Remote Submission**
   Develop locally, submit jobs to Sagehen via SSH

### Getting Help

- **HPC Support:** its-hpc@pomona.edu
- **Instructor:** Andrew Wilson
- **OnDemand Docs:** https://ondemand.hpc.pomona.edu/

Once you've verified items 1-5 above, you're ready for the workshop!

<!-- highlight <labname>/<myusername> placeholders in code blocks; remove if the varnish theme handles this natively -->
<script>(function(){var CSS='.sh-placeholder{color:#c2410c;font-weight:700}[data-bs-theme="dark"] .sh-placeholder,html.dark .sh-placeholder{color:#fdba74}@media (prefers-color-scheme: dark){[data-bs-theme="auto"] .sh-placeholder{color:#fdba74}}';var RX=/<labname>|<myusername>/g;function firstMatch(el){var w=document.createTreeWalker(el,NodeFilter.SHOW_TEXT,null),nodes=[],full='';while(w.nextNode()){nodes.push({n:w.currentNode,s:full.length});full+=w.currentNode.nodeValue;}RX.lastIndex=0;var m;while((m=RX.exec(full))){var s=m.index,e=s+m[0].length,inSpan=false,parts=[];for(var j=0;j<nodes.length;j++){var ns=nodes[j].s,ne=ns+nodes[j].n.nodeValue.length;if(ne<=s||ns>=e)continue;parts.push({node:nodes[j].n,a:Math.max(s-ns,0),b:Math.min(e-ns,nodes[j].n.nodeValue.length)});var p=nodes[j].n.parentNode;while(p&&p!==el){if(p.classList&&p.classList.contains('sh-placeholder')){inSpan=true;break;}p=p.parentNode;}}if(!inSpan&&parts.length)return parts;}return null;}function wrapParts(parts){for(var i=parts.length-1;i>=0;i--){var t=parts[i].node,txt=t.nodeValue,a=parts[i].a,b=parts[i].b;var span=document.createElement('span');span.className='sh-placeholder';span.textContent=txt.slice(a,b);var f=document.createDocumentFragment();if(a>0)f.appendChild(document.createTextNode(txt.slice(0,a)));f.appendChild(span);if(b<txt.length)f.appendChild(document.createTextNode(txt.slice(b)));t.parentNode.replaceChild(f,t);}}function run(){var st=document.createElement('style');st.textContent=CSS;document.head.appendChild(st);document.querySelectorAll('pre,code').forEach(function(el){var guard=0,parts;while((parts=firstMatch(el))&&guard++<500){wrapParts(parts);}});}if(document.readyState==='loading'){document.addEventListener('DOMContentLoaded',run);}else{run();}})();</script>
