# Storage Errors

## Disk Quota Exceeded

### Symptoms
```
write error: Disk quota exceeded
rsync: write failed: Disk quota exceeded
```

### Check Usage
```bash
$ storage-info
$ du -sh /scratch/mylab/$USER/*
```

### Solution
Clean up scratch:
```bash
$ cd /scratch/mylab/$USER
$ rm -rf old-project/
$ rm -f *.tmp
```

## Permission Denied

### Check Permissions
```bash
$ ls -lh filename
```

### Common Issues
- File owned by someone else
- Directory not readable/writable
- Wrong group permissions

### Solution
```bash
$ chmod 755 directory/
$ chmod 644 file.txt
```

## Slow I/O

### Cause
Using LTS for active computation.

### Solution
Use scratch for job I/O. Jobs can read from LTS but cannot write to it.

```bash
# In job script: read from LTS, write to scratch
cd /scratch/mylab/$USER/
./process.sh /lts/lab/data/ results/
```

After job completes, copy results from login node:
```bash
rsync -av /scratch/mylab/$USER/results/ /lts/lab/results/
```

--8<-- "includes/getting-help.md"
