# Reference Storage (/ref)

Each lab or group with accounts on the HTCF is given reference space in which to store [software](#software) and/or [reference data](#data).

`/ref` is available from any node in the HTCF.  While there is no automated cleaning of files in `/ref`, *there is no guaranteed backup of data in `/ref`*.

Therefore, **Any data in `/ref` that cannot be recreated should be copied to long term storage (LTS) for safe-keeping.**


## Structure

```
/ref
├── <lab>
    ├── data
    ├── modulefiles
    │   ├── custom
    │   └── spack
    └── software
```

### /data

The data directory is well suited for modestly sized reference data such as [NCBI blast databases](ftp://ftp.ncbi.nlm.nih.gov/blast/db/).
Larger datasets (> 500GB) are probably better suited for [LTS Object Storage](/storage/os.md)

### /software

### /modulefiles
