# Software Errors

## Module Not Found

### Symptoms
```
Error: No package matches 'nonexistent-package'
```

### Solution
Search for package:
```bash
$ spack find -v packagename
$ spack list | grep keyword
```

## Library Errors

### Symptoms
```
error while loading shared libraries: libfoo.so.1: cannot open shared object file
```

### Solution
Load dependencies:
```bash
eval $(spack load --sh dependency-package)
eval $(spack load --sh your-package)
```

## Command Not Found

### Symptoms
```
bash: python: command not found
```

### Solution
Load module in job script:
```bash
eval $(spack load --sh python)
```

## Compilation Failures

### Check
- Compiler available
- Dependencies loaded
- Correct flags

### Solution
```bash
eval $(spack load --sh gcc)
eval $(spack load --sh dependencies)
./configure
make
```

--8<-- "includes/getting-help.md"
