# Best Practices

## Quick Reference

### Pre-Submission Checklist
- [ ] Resource estimates based on test runs
- [ ] Using scratch for computation
- [ ] Time limit set appropriately
- [ ] Output directory exists
- [ ] Job script tested interactively

### Resource Optimization
- [Resource Estimation](resource-estimation.md) - Right-size your requests
- [Job Optimization](job-optimization.md) - Improve performance
- [Data Management](data-management.md) - Efficient storage usage

### Cluster Citizenship
- [Cluster Etiquette](etiquette.md) - Be considerate
- [Reproducibility](reproducibility.md) - Document your work

## Key Principles

**Right-size Resources**
Request what you need, not more. Smaller jobs start faster.

**Use Scratch**
Process data on scratch, not LTS.

**Test First**
Test with small jobs before scaling up.

**Clean Up**
Remove temporary files from scratch.

**Be Considerate**
Share resources fairly.

--8<-- "includes/getting-help.md"
