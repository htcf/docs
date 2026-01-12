# Data Management

## Standard Workflow

--8<-- "includes/storage-workflow.md"

## Storage Guidelines

### LTS (Long-Term Storage)
- Raw data
- Final results
- Important files
- Backed up

### Scratch (HTS)
- Active processing
- Temporary files
- High performance
- **Auto-deleted after 60 days**

## Best Practices

**Organize Data**
```
/lts/lab/
├── raw_data/
├── results/
├── references/
└── archived/
```

**Use rsync**
```bash
rsync -av --progress source/ destination/
```

**Clean Regularly**
```bash
# Find old files
find /scratch/mylab/$USER -mtime +30 -ls

# Remove old temp files
rm -rf /scratch/mylab/$USER/old-project/
```

**Archive Small Files**
```bash
tar czf archive.tar.gz many-small-files/
```

## See Also

- [Storage Overview](../storage/index.md) - Detailed information on all storage types
- [Storage Comparison](../storage/comparison.md) - Compare storage options
- [Data Workflow Tutorial](../tutorials/data-workflow.md) - Hands-on data management walkthrough

--8<-- "includes/getting-help.md"
