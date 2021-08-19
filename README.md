# Singularity HPC (SHPC) documentation
_This documentation is intended as an overview for facility support staff and does not replace the official SHPC documentation._

**What is SHPC and why is it useful?**
SHPC is a utility that allows the installation of containers as modules. This approach is largely automated and reduces the complexity of deploying and using containers. It also has the benefit of hiding the Singularity syntax for these ‘modular containers’, which reduces the complexity for users. 
When a container is loaded as a module, one can invoke the container with just name of the tool. For example, one could use `module load blast+` and then call blast+ with `blastn` rather than `singularity exec blastn <container_name>`.
For containerised applications that are already available in the SHPC registry, installing and using them via SHPC is much simpler than using Singularity itself. For applications that are not yet in the registry, writing a custom container recipe may still be more time effective than learning how to use Singularity.
Pulling modules for use
SHPC has a registry of commonly used software which are available to download. For example, to download BWA

```bash
$ module load shpc/0.0.27  # load SHPC module
 
$ shpc show -f bwa  # search for a package in SHPC registry (string search)
biocontainers/bwa
ghcr.io/autamus/bwa
 
$ shpc show biocontainers/bwa  # inspect specific container recipe
docker: biocontainers/bwa
url: https://hub.docker.com/r/biocontainers/bwa
maintainer: '@vsoch'
description: BWA is a software package for mapping low-divergent sequences against
  a large reference genome, such as the human genome.
latest:
  0.7.15: sha256:6f76c11a816b10440fd9d2c64c7183a31cc104a729f31a373c9b2b068138305e
tags:
  0.7.15: sha256:6f76c11a816b10440fd9d2c64c7183a31cc104a729f31a373c9b2b068138305e
  v0.7.17_cv1: sha256:9479b73e108ded3c12cb88bb4e918a5bf720d7861d6d8cdbb46d78a972b6ff1b
aliases:
  bwa: /opt/conda/bin/bwa
#### Example install
$ shpc install biocontainers/bwa:0.7.15
singularity pull --name /group/pawsey0001/mdelapierre/software/shpc/containers/biocontainers/bwa/0.7.15/biocontainers-bwa-0.7.15-sha256:6f76c11a816b10440fd9d2c64c7183a31cc104a729f31a373c9b2b068138305e.sif docker://biocontainers/bwa@sha256:6f76c11a816b10440fd9d2c64c7183a31cc104a729f31a373c9b2b068138305e
INFO:    Converting OCI blobs to SIF format
INFO:    Starting build...
Getting image source signatures
 
[..]
 
INFO:    Creating SIF file...
/group/pawsey0001/mdelapierre/software/shpc/containers/biocontainers/bwa/0.7.15/biocontainers-bwa-0.7.15-sha256:6f76c11a816b10440fd9d2c64c7183a31cc104a729f31a373c9b2b068138305e.sif
Module biocontainers/bwa:0.7.15 was created.
```

The module can then be loaded. 

```bash
$ module avail bwa  # search module
 
-------------------------------------------------------- /group/pawsey0001/mdelapierre/software/shpc/modules ---------------------------------------------------------
   biocontainers/bwa/0.7.15/module
 
$ module load biocontainers/bwa/0.7.15  # load module
 
$ bwa  # test command
```


**Making your own modules**
What if a software container is not in the SHPC registry? In this case, you can write your own container recipe for SHPC. This is essentially a YAML file with instructions of where to look for the desired container online. This YAML needs to reside in the `registry` directory of SHPC. Biocontainers is a good default source of well-curated containers, on quay.io. 

An example with the Velvet software:

```bash
$ shpc config get registry  # get registry location
registry                       /group/pawsey0001/mdelapierre/software/shpc/registry
 
# create directory tree for desired Velvet container recipe
$ mkdir -p /group/pawsey0001/mdelapierre/software/shpc/registry/quay.io/biocontainers/velvet
 
# create a new YAML container recipe in the new path (using vi as text editor here)
$ vi /group/pawsey0001/mdelapierre/software/shpc/registry/quay.io/biocontainers/velvet/container.yaml
```

Let’s look at the contents of the yaml file for Velvet.

```bash
docker: quay.io/biocontainers/velvet
 
latest:
  "1.2.10--h5bf99c6_4": "sha256:7fc2606a1431883dcd0acf830abcfeddb975677733d110a085da0f07782f5a27"
tags:
  "1.2.10--h5bf99c6_4": "sha256:7fc2606a1431883dcd0acf830abcfeddb975677733d110a085da0f07782f5a27"
  "1.2.10--hed695b0_3": "sha256:b17fd98d802c1e78dde5fd2c5431efc1969db35a279f3a5ca7afcb46efc66e4a"
 
maintainer: "@marcodelapierre"
 
# these are optional
description: "Velvet is a sequence assembler for short reads."
url: https://quay.io/repository/biocontainers/velvet
 
aliases:
  velvetg: /usr/local/bin/velvetg
  velveth: /usr/local/bin/velveth
```

Let's comment on the key components of this YAML file:
•	docker is the repository path for the container, without version tags
•	tags is a list of container tags (versions) with the corresponding SHA message digest (shasum); these need to be manually collected from the repository website, in this case https://quay.io/repository/biocontainers/velvet?tab=tags 
•	latest is a copy-paste of the tag from above, to be used as "latest" version
•	maintainer is the Github username of the creator (required to contribute the recipe back to the Github repository of SHPC; put any name if you don't have one)
•	aliases is a list of command names that will be made available by the SHPC module, with the corresponding commands from inside the container; these need to be manually provided, either by reading through the documentation of the package, or by downloading and inspecting the container
Then you can install Velvet just as we did with BWA previously. 

