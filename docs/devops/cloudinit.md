# Cloud Init

[Cloud-init](https://cloudinit.readthedocs.io/en/latest/index.html) 
provides tools for initialising compute resources deployed acoss various 
cloud platforms. Cloud images typically come with cloud-init pre-installed and 
configured to run on first boot. 

Cloud providers also provide an IMDS - Instance MetaData Service. The IMDS is 
used to expose cloud-init configuration to the instance. During early boot, 
cloud-init connects to the IMDS webserver to collect configuration.

## Configuration Sources

Cloud-init builds a single configuration from multiple sources:

1. Base Config - consists of multiple sources (lowest to highest priority):
    a. Hard Coded Config: Config that is part of cloud-init and cannot be changed
    b. Configuration Directory: anything defined in `/etc/cloud/cloud.cfg` and `/etc/cloud/cloud.cfg.d/`
    c. Runtime Config: anything defined in `/run/cloud-init/cloud.cfg`
    d. Kernel Command Line: anything found between `cc` and `end_cc` on the kernel command line 
2. Vendor and User Data: provided by the datasource and added to the Base Config
3. Network Configuration: occurs independantly of other cloud-init configuration

End Users can pass configuration is specified as User Data. Distro Providers will modify
the Base Config and Cloud Providers will modify Vendor Data.

Cloud-init functionality is integrated into the five boot stages:

1. Generator
2. Local
3. Network
    - also runs disk_setup and mounts modules, to configure disks, partitions and mount points. On Azure, this stage is used to setup mounts necessary for cloud-init to work. Additional disks configuration should be added after this stage completes
4. Config
    - runs config modules only
5. Final
    - runs as late in the boot process as possible. Typically this stage is for running commands that would otherwise be run by a user logging-in to the system to run the commands. This will include package installation and post-installation scripts. 

## Determining First Boot

Cloud-init configuration is divided into per-instance and per-boot configuration. Per-instance configuration is only run on the first boot: per-boot configuration runs at every boot. 
Cloud-init stores a cache for use across stages and boots. If the cache is already present, 
then cloud-init has already run on this instance. 

However, a cache will exist if the instance is built from an image that has 
previously been launched. To determine if the presence of a cache represents
a first boot for an instance, cloud-init runs a **check** to compare the Instance ID in the 
cache to the Instance ID found at runtime: if they do not match, then this is determined
to be a first-boot for the instance. 
This behaviour can be over-ridden by using a **trust** behaviour: which
tells cloud-init not to compare the Instance ID and assume that this is not a 
first boot if a cloud-init cache exists. Using the **trust** behaviour, cloud-init 
per-instance configuration will only run if no cache is found. 

## Re-Running Cloud-Init Configuration

You can re-run the config and final stages of a cloud-init configuration by logging-on
to the instance and executing:

```bash
sudo cloud-init clean --logs
sudo cloud-init init --local
sudo cloud-init init
sudo cloud-init modules --mode=config
sudo cloud-init modules --mode=final
```

## User Data

User Data can be provided in a variety of formats:

1. Cloud Config: yaml format. Begins with either:
    - `#cloud-config` or
    - `Content-Type: text/cloud-config`
2. User Data: used to execute shell scripts. Begins with either:
    - `#!` or 
    - `Content-Type: text/x-shellscript`
3. Gzip-compressed Data: user data is limited to 16,384 bytes. Gzipped data can be used where scripts would exceed these limits
4. Mime Multipart File: Can be used to specify configuration using any combination of the other formats available. 
5. Include File: contains a list of URLs, one per line. Each URL is retrieved and processed in order. Begins with:
    - `#include` or 
    - `Content-Type: text/x-include-url`
6. Cloud-boothook: boothook data is stored in /var/lib/cloud and executed immeadiately. Begins with `#cloud-boothook`
7. Part-handler: Python code. Begins with:
    - `#part-handler` or 
    - `Content-Type: text/part-handler`

## Debugging Cloud-Init

To check the status of your cloud-init on a booted instance, run: 

```
cloud-init status --wait
```

Cloud-init log files are kept in `/var/log/cloud-init.log`, `/var/log/cloud-init-output.log`
and `/run/cloud-init`.

Cloud-init configuration files are found in `/etc/cloud/cloud.cfg` and `/etc/cloud/cloud.cfg.d/*.cfg`.

The `/var/lib/cloud/instance` directory points to the most recently used instance-id directory and
contains the the data received from the datasources including user and vendor data. It also contains
a datasource file with information about the datasource used. You can also run the `cloud-id` command
to find out which datasource is being used. 

The `/var/lib/cloud/data` directory contains information from the previous boot. 
