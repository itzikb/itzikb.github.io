---
layout: single
permalink: /resize-disk-openstack-virt/
title: "Resize an existing disk of a compute in a virt setup"
tags: openstack
comments: true
---

The scenario we are addressing is a case where Openstack setup is a virtualized setup where the undercloud,controllers and compute nodes are all on the same hypervisor and our vms can't be launched because the compute nodes were created with small disks. We are going to shut down the compute nodes, resize the disks and bring them up again. Obviously this is for a non-production environment

## Shut down the compute nodes

We have 3 compute nodes

```bash
$ . stackrc; for i in 845ecb4e-86ef-47cb-889a-b4576d13ea22 d8feaee7-c8db-4d1b-a108-5c7a601d6ccf c1f6677c-d5c1-4429-ab04-6259cbd22591; do openstack server stop $i;done
```

## Go to the images directory on the hypervisor

```bash
# cd /var/lib/libvirt/images/
```

## Create a larger disk file

In our case it's 130G. We show here just one compute , do the same (with different name) for each compute

```bash
# qemu-img create -f qcow2 compute-0-disk2.qcow2  130G
Formatting 'compute-0-disk2.qcow2', fmt=qcow2 size=139586437120 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```

## Check the file system of the current compute's disk

Here we see that we need to expand sda2

```bash
# virt-filesystems --long -h --all -a compute-0-disk1.qcow2
Name       Type        VFS      Label       MBR  Size  Parent
/dev/sda1  filesystem  iso9660  config-2    -    1.0M  -
/dev/sda2  filesystem  xfs      img-rootfs  -    80G   -
/dev/sda1  partition   -        -           83   1.0M  /dev/sda
/dev/sda2  partition   -        -           83   80G   /dev/sda
/dev/sda   device      -        -           -    80G   -
```

## Resize the disk

```bash
# virt-resize --expand /dev/sda2 compute-0-disk1.qcow2 compute-0-disk2.qcow2 
[   0.0] Examining compute-0-disk1.qcow2
**********

Summary of changes:

/dev/sda1: This partition will be left alone.

/dev/sda2: This partition will be resized from 80.0G to 130.0G.  The 
filesystem xfs on /dev/sda2 will be expanded using the ‘xfs_growfs’ 
method.

**********
[   2.5] Setting up initial partition table on compute-0-disk2.qcow2
[   3.4] Copying /dev/sda1
[   3.4] Copying /dev/sda2
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[ 126.4] Expanding /dev/sda2 using the ‘xfs_growfs’ method

Resize operation completed with no errors.  Before deleting the old disk, 
carefully check that the resized disk boots and works correctly.
```

## Check the file system of the new disk

```bash
# virt-filesystems --long -h --all -a compute-0-disk2.qcow2 
Name       Type        VFS      Label       MBR  Size  Parent
/dev/sda1  filesystem  iso9660  config-2    -    1.0M  -
/dev/sda2  filesystem  xfs      img-rootfs  -    130G  -
/dev/sda1  partition   -        -           83   1.0M  /dev/sda
/dev/sda2  partition   -        -           83   130G  /dev/sda
/dev/sda   device      -        -           -    130G  -
```

## Backup the old disk and rename the new one

We backup the old disk. It can be removed after the compute nodes are properly booted up

```bash
# mv compute-0-disk1.qcow2{,.bak}
# mv compute-0-disk2.qcow2 compute-0-disk1.qcow2
# chown qemu:qemu  *
```

## Start the compute nodes

```bash
# . stackrc ; for i in 845ecb4e-86ef-47cb-889a-b4576d13ea22 d8feaee7-c8db-4d1b-a108-5c7a601d6ccf c1f6677c-d5c1-4429-ab04-6259cbd22591; do openstack server start $i;done
$ openstack server list
+--------------------------------------+--------------+--------+------------------------+----------------+------------+
| ID                                   | Name         | Status | Networks               | Image          | Flavor     |
+--------------------------------------+--------------+--------+------------------------+----------------+------------+
| d27c05db-c566-443b-ac4a-6f0b16a2afd4 | controller-0 | ACTIVE | ctlplane=192.168.24.41 | overcloud-full | controller |
| d8feaee7-c8db-4d1b-a108-5c7a601d6ccf | compute-1    | ACTIVE | ctlplane=192.168.24.21 | overcloud-full | compute    |
| 845ecb4e-86ef-47cb-889a-b4576d13ea22 | compute-0    | ACTIVE | ctlplane=192.168.24.24 | overcloud-full | compute    |
| c1f6677c-d5c1-4429-ab04-6259cbd22591 | compute-2    | ACTIVE | ctlplane=192.168.24.54 | overcloud-full | compute    |
+--------------------------------------+--------------+--------+------------------------+----------------+------------+
```

All the compute nodes are up and running and we can now create are virtual machines!
