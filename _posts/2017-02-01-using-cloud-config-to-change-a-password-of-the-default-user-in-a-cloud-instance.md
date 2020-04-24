---
layout: single
permalink: /using-cloud-config-to-change-a-password-of-the-default-user-in-a-cloud-instance/
title: "Using cloud-config to change a password of the default user in a cloud instance"
tags: openstack
comments: true
---
Sometimes there is a need to customize an instance when it is booting. One example is to access the instance using SSH with a password instead of a key. One way to do it is supplying user data to the instance by a cloud-config file. Most of the cloud images have a cloud-init package that doing this customization. It has couple of modules that run in number of stages. You can use these modules to install packages, run script, start Puppet agent etc.

### Using a config-file to change the user password
First we create a YAML file:
```yaml
#cloud-config
password: 123456
ssh_pwauth: True
chpasswd:
  expire: false
```

Then we can start the instance (We are using OpenStack in our example):
```shell
    $ nova boot --flavor m1.small --image fedora --user-data mydata.yaml --nic net-id=6e730ccb-bb42-4cb9-8126-b1582abeec47 vm1
```
**--user-data mydata.yaml** is the way specifying our user data. Out cloud-config file is called mydata.yaml.  
**ssh\_pwauth** is used to allow us to be able to use a password instead of a key when accessing the instance using SSH.  
**chpasswd** is the change password policy.We specify expire to be false so that we won't need to change the password on our first login. Now you can ssh to the instance (vm1 in our example) using the default user for the image (e.g. fedora for the Fedora cloud image) with the password you supplied (123456 in our example).  

### Using Configuration Driver in OpenStack
The user data relies on a metadata service. You can use write metadata to a configuration drive instead in case it's not available. Add the --config-drive true to the nova boot command:
```shell
$ nova boot --flavor m1.small --image fedora --user-data mydata.yaml --nic net-id=6e730ccb-bb42-4cb9-8126-b1582abeec47 --config-drive true vm1
```

After the instance started you can look at the user data supplied:
```shell
$ sudo mkdir -p /mnt/config
$ sudo mount /dev/disk/by-label/config-2 /mnt/config
$ cat /mnt/config/openstack/latest/user_data
```

