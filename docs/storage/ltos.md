# Long Term Object Storage


Long Term Object Storage (LTOS) is an architecture that manages data as [objects](https://en.wikipedia.org/wiki/Object_storage), as opposed to traditional LTS which uses file systems and block storage.

Access to LTOS is [REST](https://en.wikipedia.org/wiki/Representational_state_transfer)-based (HTTP).  Although LTOS uses [a subset](https://docs.ceph.com/en/latest/radosgw/s3/) of the Amazon [s3 API](https://en.wikipedia.org/wiki/Amazon_S3), **LTOS data is stored internally**, not in Amazon.

Unlike LTS, which is only accessible from the login server, LTOS is accessible from all HTCF nodes.

## Purpose

LTOS can be a good alternative to LTS any time the data in question doesn't need to be heavily manipulated or modified.

Good Candidates for LTOS:

- Raw sequence data and finished analysis data
- Archived or rarely referenced data such as alumni files.
- Data that is not often modified, once created.

Not Good Candidates for LTOS:

- scripts in use or under development
- software
- intermediate files generated during job processing

## Using LTOS

### Software Tools

#### CLI

The two most common command line tools for accessing LTOS are s3cmd and aws-cli.

##### s3cmd

First a configuration files needs to be created 

#### Workflow

### Putting files in LTOS


## Examples


--8<-- "includes/coming_soon.md"
