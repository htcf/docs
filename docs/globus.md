
**To register a personal HTCF endpoint with Globus.  (only needs to be run once)**

```
$ globusconnectpersonal -setup
```

Open the link presented in a web browser and follow the on screen instructions.

Copy the auth code given at the end of the registration and paste it in the terminal where prompted.

**To start the endpoint:**

```
$ globusconnectpersonal -start -restrict-paths /location/you/want/to/access/files
```

For example, if transferring data to/from /scratch/mylab/mydir/project

```
$ globusconnectpersonal -start -restrict-paths /scratch/mylab/mydir/project
```

For long running transfers, `globusconnectpersonal` can be run in a `screen` session or an `sbatch` job.

Once the endpoint is started, transfers can begin.  They can be initiated from the globus website, or the globus command line tool (module load py-globus-cli).

Please see the Globus documentation to understand both of these methods.
