# SPACK

**Please NOTE:**
Installation of software on the login node is unsupported.  Please build/install software from an interactive job.

## Tutorial

See [the official spack tutorial](https://spack-tutorial.readthedocs.io/en/latest/)

## Spack Initialization

To create a lab instance of the spack package manager:

1.  Download and untar [a spack release](https://github.com/spack/spack/releases) into `/ref/<lab_name>/software`

2.  Rename (or make a symlink from) `/ref/<lab_name>/software/spack-VERSION` to `/ref/<lab_name>/software/spack`.

3.  Logout and log back in.  This will ensure the spack command is available in the PATH.


## Install Software Using Spack

see [the official spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#cmd-spack-install)


## Using software built by spack (ie. preparing the environment)

Once spack has built software, the bash shell needs to have the proper environment variables set to access the software.
This is accomplished using the `spack load` command.

### Using spack to set the environment

Spack packages can be "loaded" similar to the way modules are loaded.

Given a [spec](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies), a spack command can be used to generate the appropriate environment variables to "load" spack-installed software.

To view the environment variables that would be set:

```
$ spack load --sh <spec>
```

To actually set the environment varibles:

```
$ eval $( spack load --sh <spec> )
```

This command is similar in function to the older `module load <spec>` commands.

See [the official spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies) for more information on specs.

These commands can be placed in an sbatch file to be used in a job.

```
#!/bin/bash

eval $( spack load --sh <spec> )

```

## Example: install biom-format

```
$ spack list biom
==> 7 packages.
microbiomeutil  py-biomine    r-biomart   r-biomformat
py-biom-format  r-biom-utils  r-biomartr

$ spack versions py-biom-format
==> Safe versions (already checksummed):
  2.1.10  2.1.9  2.1.7  2.1.6
...

$ spack install py-biom-format@2.1.10
...

$ eval $( spack load --sh py-biom-format@2.1.10 )
```
