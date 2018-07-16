---
title: "VSphere Storage for Docker 2.2.1 (VMware) Solution Brief for Docker Enterprise Edition 17.06"
summary: "Docker Solution Briefs enable you to integrate the Docker container platform with popular 3rd party ecosystem solutions. This Solution Brief describes how to install and test the VMware Vsphere Storage for Docker plugin for Docker volumes."
type: guide
author: khudgins
visibleto: loggedinleads
campaign: "docker-certified-infrastructure"
product:
  - ee
testedon:
  - "ee-17.06.2-ee-7"
  - "ucp-2.2.5"
  - "dtr-2.4.2"
platform:
  - linux
tags:
  - "solution-brief"
  - storage
dateModified: "2018-04-24T13:24:21+00:00"
dateModified_unix: 1524576261
uniqueid: KB000669
---

## Overview

Docker Solution Briefs enable you to integrate the Docker Enterprise Edition (EE) container platform with popular 3rd party ecosystem solutions for networking, load balancing, storage, logging and monitoring, access management, and more.

This Solution Brief describes how to install and test the VMware Vsphere Storage for Docker plugin for Docker volumes.

> Information on vSphere Storage is provided by Docker as a known, working configuration for Docker 17.06.2-ee-. Docker does not support vSphere Storage.  Please contact the vendor approved support methods if you have any questions or problems with them.

## vSphere Storage Overview

vSphere Storage for Docker enables customers to address persistent storage requirements for Docker containers in vSphere environments. This service is integrated with [Docker Volume Plugin framework](https://docs.docker.com/engine/extend/). Docker users can now consume vSphere Storage (vSAN, VMFS, NFS, VVol) to stateful containers using Docker.

vSphere Storage for Docker is Docker Certified to use with Docker Enterprise Edition and is available in [Docker Store]().

vSphere Docker Volume Service (vDVS) technology enables running stateful containers backed by storage technology of choice in a vSphere environment. vDVS which works with proven storage technologies from VMware and is easy to install and use from end user perspective while retaining visibility and control in hands of a vSphere administrator.


## Prerequisites

- [Docker EE 17.06 Platform on VMware vSphere](https://success.docker.com/article/certified-infrastructures-vmware-vsphere)
- VSphere 6.0

## Installation and Configuration

### On ESX

vSphere Docker Volume Service technology enables stateful containers to access the storage volumes backed by vSphere volume services. To use Docker volumes using the vSphere driver, you must first install the latest release of the vSphere Docker Volume Service (vDVS) driver, which is available as a vSphere Installation Bundle (VIB). To perform this operation, log into each of the ESXi hosts in turn and then download and install the latest release of vDVS driver.

> **Note:** If there’s an incremental build of the VIB that’s been released, version numbers of the filenames are likely to change, so adjust as needed.

1. Download the VIB from to your local machine from <https://bintray.com/vmware/vDVS/VIB/0.21.1#files>.

2. Then, on your local machine, upload the VIB to each ESX server in your Docker cluster:

    ```
    % scp VDVS_driver-0.21.1-offline_bundle-7812185.zip root@vsphere01.example.com:/tmp/
    ```

3. Once the VIB is on each box, log into each system in turn as root and run:

    ```
    # cd /tmp
    # unzip VDVS_driver-0.21.1-offline_bundle-7812185.zip
    # esxcli software vib install -v /tmp/vib20/esx-vmdkops-service/VMWare_bootbank_esx-vmdkops-service_0.21.c420818-0.0.1.vib  --no-sig-check
    ```

4. You should see the following output:

    ```
    Installation Result
       Message: Operation finished successfully.
       Reboot Required: false
       VIBs Installed: VMWare_bootbank_esx-vmdkops-service_0.21.c420818-0.0.1
       VIBs Removed:
       VIBs Skipped:
    ```

### On Each Docker Host

The Docker volume plugin requires Docker engine to be installed as a prerequisite. On each Docker host, run (with root privileges, through sudo or as root):

```
# docker plugin install --grant-all-permissions --alias vsphere vmware/vsphere-storage-for-docker:latest
```

## Verifying the Install

The fastest way to verify the installation is to run the cert tests at <https://github.com/docker/vol-test> if Linux hosts, and/or bash is installed on Windows hosts.

To install the `vol-test` tools:

1. Install BATS:

    ```
    % git clone https://github.com/sstephenson/bats.git
    % cd bats
    % sudo ./install.sh /usr/local
    % cd ..
    % git clone https://github.com/docker/vol-test
    % cd vol-test
    % git submodule init
    % git submodule update --recursive --remote
    ```

2. Run the tests:

    Refer to the readme for the `vol-test` package to tailor the configuration variables for your environment. Edit the singlenode and secondnode BATS files to uncomment the `skip` command in the plugin install test since that’s already been done. This is an example run:

    ```
    % export PREFIX=”ssh node1”
    % export PREFIX2=”ssh node2”
    % export VOLDRIVER=”vsphere”
    % bats singlenode.bats secondnode.bats
    khudgins@jump:~/vol-test$ bats singlenode.bats secondnode.bats
     - Test: Install plugin for driver (vsphere) (skipped: Faster for rev during development without it - leave driver installed)
     ✓ Test: Create volume using driver (vsphere)
     ✓ Test: Confirm volume is created (volume ls) using driver (vsphere)
     ✓ Test: Confirm docker volume inspect works using driver (vsphere)
     ✓ Start a container and mount the volume
     ✓ Write a textfile to the volume
     ✓ Confirm textfile contents on the volume
     ✓ Create a binary file
     ✓ get a checksum for that binary file
     ✓ Confirm checksum
     ✓ Destroy container
     ✓ Let's see if our data is still here
     ✓ Confirm textfile contents are still on the volume
     ✓ Confirm checksum persistence
     ✓ Destroy container
     - Test: Install plugin for driver (vsphere) on node 2 (skipped: This test works, faster for rev without it)
     ✓ Test: Confirm volume is visible on second node (volume ls) using driver (vsphere)
     ✓ Start a container and mount the volume on node 2
     ✓ Confirm textfile contents on the volume from node 2
     ✓ Confirm checksum for binary file on node 2
     ✓ Destroy container on node 2
     ✓ Remove volume
     ✓ Confirm volume is removed from docker ls

    23 tests, 0 failures, 2 skipped
    ```

## Best Practice Recommendations

None Yet

## Monitoring and Troubleshooting

Monitor the VMware driver container on each node - make sure it’s active and running.

## Further Reading

Refer to the following links for additional information:

- <https://github.com/vmware/vsphere-storage-for-docker>
- <http://vmware.github.io/vsphere-storage-for-docker/documentation/>
- <https://bintray.com/vmware/vDVS/VIB/>
