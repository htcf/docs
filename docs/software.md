# Software 
* * *
### Modules
>Lmod is a Lua based module system that easily handles the MODULEPATH Hierarchical problem. Environment Modules provide a convenient way to dynamically change the users' environment through modulefiles. This includes easily adding or removing directories to the PATH environment variable. Modulefiles for Library packages provide environment variables that specify where the library and header files can be found.

Software is handled using [lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod).  There should be minimal need to modify your .bashrc or .profile unless you're installing software locally to test.


####Basics
Lmod is managed using the command *module*, using this command without options will show you a list of all available subcommands.

~~~~
~$ module

Modules based on Lua: Version 6.6  2016-10-13 13:28 -05:00
    by Robert McLay mclay@tacc.utexas.edu

module [options] sub-command [args ...]

Help sub-commands:
------------------
  help                              prints this message
  help                module [...]  print help message from module(s)

~~~~


A list of software available the command:

~~~~
~$ module avail
---------------------------------------------- /opt/apps/modules ----------------------------------------------
   GD/2.1.1                            genometools/1.5.8                  pindel/0.2.5b6
   R/2.15.3                            ghostscript/9.19                   pmap/11-25-2010
   R/3.1.2                             glimmer/3.02b                      poretools/0.6.0
~~~~

To load the latest (default) version of module:

~~~~
module load ncbi-blast
~~~~

To specifiy which version of the module you would like to use:

~~~~
module load ncbi-blast/2.2.30+
~~~~

Be sure to specify which version of the sofware you'd like to use in your scripts to ensure consistent results, as software updates may break pipelines.

Software with prerequisites are loaded dynamically, for example:

~~~~
~$ module load prokka
~$ ml

Currently Loaded Modules:
  1) aragorn/1.2.36   3) prodigal/2.6.2       5) tbl2asn/24.2       7) hmmer/3.1b1   9) prokka/1.11
  2) infernal/1.1.1   4) ncbi-blast/2.2.31+   6) bio-perl/1.6.923   8) barrnap/0.6
~~~~

### GUI Software

As the HTCF primarily a batch queuing system for high-throughput processing of large amounts of data,  GUI application are not directly supported by the HTCF.  GUI application installation and setup on the HTCF are left to the end user.

