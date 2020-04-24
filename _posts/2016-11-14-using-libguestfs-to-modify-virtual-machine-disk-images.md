---
layout: single
permalink: /using-libguestfs-to-modify-virtual-machine-disk-images/
title: "Using libguestfs to modify virtual machine disk images"
tags: virt-tools
comments: true
---
There are many cases when you want to modify an image.
For example - most of the cloud images can be accessed only using a key. What happens when you want to use the console to access your virtual machine?
For debugging purpose you can change the image so it will also allow access by supplying password.

Another example is to install packages that don't exist in the image. And yet another example is that the image is too small and you want to resize it.
When using OpenStack and first two examples can be probably be done by using cloud-init but this should be done for every instance we launch.

I'll give some examples for using libguestfs to modify images.

First we need to install libguestfs-tools
```bash
$ sudo dnf -y install libguestfs-tools
```
You may need to install libvirt and start libvirtd
```bash
$ sudo dnf -y install libvirt && sudo systemctl start libvirtd
```
We'll download a Fedora image but you can use your own image:
```bash
$ cd /tmp
$ wget https://download.fedoraproject.org/pub/fedora/linux/releases/24/CloudImages/x86_64/images/Fedora-Cloud-Base-24-1.2.x86_64.qcow2
```
### Listing a directory contents
```bash
$ virt-ls -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 /etc
.pwd.lock
DIR_COLORS
DIR_COLORS.256color
DIR_COLORS.lightbgcolor
GREP_COLORS
.. (output truncated)
```

### Editing a file
If we want to change a specific file we can run the following command:
```bash
$ virt-edit -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 /etc/bashrc
```
We then use our editor to change the file. We can also use virt-edit non-interactively to change the content of a file. Suppose we have file /etc/dummy with a content of the word _dummy_ and we want to change it to _foo_
```bash
$ virt-edit -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 -e 's/dummy/foo/' /etc/dummy
```

### Display a file
We can display the content of a file by running:
```bash
$ virt-cat -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 /etc/bashrc
```

### Installing packages
To install vim and git packages run:
```bash
$ virt-customize -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 --install vim,git
```
### Copy a file/directory to an image
For example copy a file named 'dummy' to /etc
```bash
virt-copy-in -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 dummy /etc
```
The same can be done by using virt-customize
```bash
$ virt-customize -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 --upload dummy:/etc
```
### Running a shell command
We can remove our file by running:
```bash
$ virt-customize -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2 --run-command 'rm /etc/dummy'
```

### Resizing an image
Sometimes our image is too small for our needs. Let's see how to enlarge it. Let's the view the image metadata:
```bash
$ qemu-img info Fedora-Cloud-Base-24-1.2.x86_64.qcow2
image: Fedora-Cloud-Base-24-1.2.x86_64.qcow2
file format: qcow2
virtual size: 3.0G (3221225472 bytes)
disk size: 510M
cluster_size: 65536
Format specific information:
compat: 0.10
refcount bits: 16
```
The image is 3G and for now only 510M are used.
```bash
$ virt-filesystems --long -h --all -a Fedora-Cloud-Base-24-1.2.x86_64.qcow2
Name Type VFS Label MBR Size Parent
/dev/sda1 filesystem ext4 - - 3.0G -
/dev/sda1 partition - - 83 3.0G /dev/sda
/dev/sda device - - - 3.0G -
```
We see that we have one partition called /dev/sda1.
We need to create another empty image with the size we want (10G in our case) and then use it to move the content of the original image:
```bash
$ qemu-img create -f qcow2 fedora.qcow2 10G
Formatting 'fedora.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16
$ virt-resize --expand /dev/sda1 Fedora-Cloud-Base-24-1.2.x86_64.qcow2 fedora.qcow2
[0.0] Examining Fedora-Cloud-Base-24-1.2.x86_64.qcow2
**********
Summary of changes:

/dev/sda1: This partition will be resized from 3.0G to 10.0G. The
filesystem ext4 on /dev/sda1 will be expanded using the 'resize2fs' method.

**********
[3.4] Setting up initial partition table on fedora.qcow2
[3.5] Copying /dev/sda1
100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[11.0] Expanding /dev/sda1 using the 'resize2fs' method

Resize operation completed with no errors. Before deleting the old disk,
carefully check that the resized disk boots and works correctly.

$ virt-filesystems --long -h --all -a fedora.qcow2
Name Type VFS Label MBR Size Parent
/dev/sda1 filesystem ext4 - - 10G -
/dev/sda1 partition - - 83 10G /dev/sda
/dev/sda device - - - 10G -
```
We now have an image called fedora.qcow2 which has the content of the previous image but has a size of 10G.

### Changing the root password of the image
To change the root password of the image to 'mypassword':
```bash
$ virt-customize -a fedora.qcow2 --root-password password:mypassword
```

### Enabling SSH for root for cloud images
When using cloud images in OpenStack - connecting to an instance as a root user using a password is disabled by cloud-init. To remove this restriction run:
```bash
$ virt-edit -a fedora.qcow2 -e 's/^disable_root: 1/disable_root: 0/' /etc/cloud/cloud.cfg
$ virt-edit -a fedora.qcow2 -e 's/^ssh_pwauth:\s+0/ssh_pwauth: 1/' /etc/cloud/cloud.cfg
```
**Note:** Allowing SSH connection using a password and especially as root is not recommended. &nbsp; &nbsp;
