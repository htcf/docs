# Software on the HTCF

Software building and installation on the HTCF is primarily self-service.
Labs are free to use their [/ref](/storage/ref.md) space to install software using whatever means is most comfortable.
Use of [Spack](#spack) to install common software is recommended.


## Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

### Spack setup

To create a lab instance of the spack package manager:

1.  Download and untar spack release into `/ref/<lab>/software`

2.  Rename (or make a symlink from) `/ref/<lab>/software/spack-VERSION` to `/ref/<lab>/software/spack`.

3.  Logout and log back in.  This will ensure the `spack` command is available in the PATH.
