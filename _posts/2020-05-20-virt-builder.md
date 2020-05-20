---
layout: single
permalink: /virt-builder/
title: "Building virtual machines images using virt-builder"
tags: virt-tools
comments: true
---
**virt-builder** is a tool that ease the process of downloading virtual machines images and customizing them.

## Listing the available images  
```
$ virt-builder -l
opensuse-13.1            x86_64     openSUSE 13.1
opensuse-13.2            x86_64     openSUSE 13.2
opensuse-42.1            x86_64     openSUSE Leap 42.1
opensuse-tumbleweed      x86_64     openSUSE Tumbleweed
centos-6                 x86_64     CentOS 6.6
centos-7.0               x86_64     CentOS 7.0
centos-7.1               x86_64     CentOS 7.1
centos-7.2               aarch64    CentOS 7.2 (aarch64)
centos-7.2               x86_64     CentOS 7.2
centos-7.3               x86_64     CentOS 7.3
centos-7.4               x86_64     CentOS 7.4
centos-7.5               x86_64     CentOS 7.5
centos-7.6               x86_64     CentOS 7.6
centos-7.7               x86_64     CentOS 7.7
...
(output trancated)  
```  

## Building and customizing an image  
We'll demostate the usage of the tool by building a Fedora 29 image
```
$ virt-builder fedora-29  -o vm1.img --format qcow2 \
  --size 20G --root-password file:/tmp/rootp \
  --hostname myhostname --install 'vim,tmux' \
  --update --run-command 'echo hello > /tmp/hello'
[   1.9] Downloading: http://builder.libguestfs.org/fedora-29.xz
[   2.5] Planning how to build this image
[   2.5] Uncompressing
[   8.6] Resizing (using virt-resize) to expand the disk to 20.0G
[  35.9] Opening the new disk
[  41.2] Setting a random seed
[  41.3] Setting the hostname: myhostname
[  41.3] Installing packages: vim tmux
[ 152.1] Updating packages
[ 339.6] Running: echo hello > /tmp/hello
[ 339.7] Setting passwords
[ 341.5] Finishing off
                   Output file: vm1.img
                   Output size: 20.0G
                 Output format: qcow2
            Total usable space: 19.3G
                    Free space: 17.7G (91%)
```
**-o vm1.img** The default image name is the the name of the OS e.g. fedora-29.img  
**--format qcow2** We are using qcow2 type instead of raw  
**--size 20G** Resize the image to 20G  
**--root-password file:/tmp/rootp** Set the root password to the one stored at /tmp/rootp, otherwise a ramdomized password will be output to the console
**--hostanem myhostname** Set the hostname to myhostname  
**--install 'vim,tmux'** Install the packages **vim** and **tmux**  
**--updte** Update the system with the latest versions of the packages  
**--run-command 'echo hello > /tmp/hello'** Run a shell command 

