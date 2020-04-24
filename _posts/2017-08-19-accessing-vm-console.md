---
layout: single
permalink: /accessing-vm-console/
title: "Accessing a virtual machine console"
tags: virt-tools
comments: true
---
Sometimes you want to access a Virtual machine console, for example when you can't access it through SSH and you want to find out why.
The following post will show you how to access the console using virsh or vncviewer.

## Accessing the console using virsh

Connect to the Hypervisor your Virtual Machine is running on and identify it's ID or number:
```bash
$ sudo virsh list
 Id Name                 State
----------------------------------
  0 Domain-0             running
```

```bash
$ sudo virsh console <virsh id or name>
```

## Accessing the console using VNC viewer
We want to access the Virtual machine from a node that has access to the hypervisor the Virtual machine is running on.

First we install the following packages:

On Fedora:
```shell_session
$ sudo dnf -y install tigervnc xorg-x11-xauth dejavu-sans-fonts
```

On RHEL
```shell_session
$ sudo yum -y install tigervnc xorg-x11-xauth dejavu-sans-fonts
```

Then we need to figure out what are the IP and port the Virtual Machine's VNC server is running on.

There are couple of ways to get this information:

One way is to access the hypervisor the Virtual machine is running on and run:

```shell_session
$ virsh domdisplay <virtual machine name or id>
```
For example:
```shell_session
$ virsh domdisplay myvm
vnc://192.168.70.5:0
```
The IP and port after the vnc:// part is our connection information.
We can get the id or the name of the Virtual machine by listing the virtual machines on this hypervisor:

```shell_session
$ virsh list
```
We then connect to the  machine we want to access the virtual machine from:
```shell_session
$ ssh -Y -C <our machine IP>
```

And launch our VNC client:
```shell_session
$ vncviewer <Virtual machine IP: VNC port>
```
Using the example above:
```shell_session
$ vncviewer 192.168.70.5:0
```
