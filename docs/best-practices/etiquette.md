# Cluster Etiquette

## Core Principles

**Shared Resource**
The cluster serves many users. Be considerate.

**Fair Share**
Request only what you need.

**Community Standards**
Help maintain a good environment for all.

## Login Node Usage

!!! warning "Login Node Policy"
    **DO NOT** run computational work on login nodes.
    Use interactive or batch jobs instead.

**OK on Login Node:**
- File management (cp, mv, ls)
- Job submission (sbatch)
- Text editing
- Quick file viewing

**NOT OK on Login Node:**
- Running analysis scripts
- Heavy I/O operations
- Long-running processes
- Computational work

## Resource Requests

**Request What You Need**
```bash
# Bad - over-requesting
#SBATCH --cpus-per-task=32
#SBATCH --mem=256G

# Good - right-sized
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
```

**Don't Monopolize**
- Limit concurrent jobs if many running
- Use array job throttling: `--array=1-100%20`

**Release Unused Resources**
Cancel jobs you don't need:
```bash
scancel <JOBID>
```

## Storage Etiquette

**Clean Up Scratch**
Don't wait for auto-deletion:
```bash
rm -rf /scratch/mylab/$USER/old-project/
```

**Respect Quotas**
Your usage affects your lab's quota.

**Shared Directories**
- Use appropriate permissions
- Organize files logically
- Don't clutter shared spaces

## Interactive Jobs

!!! reminder
    Interactive jobs are for development, not production.
    - Exit sessions when not actively using them
    - Don't leave sessions running overnight
    - Don't run batch workloads interactively

## Being a Good Citizen

**Test Before Scaling**
Test with 1-2 jobs before submitting 1000.

**Monitor Your Jobs**
Check if jobs are running efficiently.

**Report Issues**
Help improve the cluster for everyone.

**Help Others**
Share knowledge and solutions.

--8<-- "includes/getting-help.md"
