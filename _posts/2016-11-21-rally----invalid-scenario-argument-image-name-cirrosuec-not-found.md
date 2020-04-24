---
layout: single
permalink: /rally-invalid-scenario-argument-image-name-cirros-uec-not-found/
title: "Rally -  Invalid scenario argument: 'Image '{'name': '^cirros.*uec$'}' not found'`"
tags: openstack rally
comments: true
---
From Rally's documentation:_Rally is a benchmarking tool that automates and unifies multi-node OpenStack deployment, cloud verification, benchmarking & profiling_When following the Rally documentation - there is an example how to run a task.
```shell
 $ rally task start rally/samples/tasks/scenarios/nova/boot-and-delete.json
```
You may encounter the following error:

    Task config is invalid: `Input task is invalid!
    
    Subtask NovaServers.boot_and_delete_server[0] has wrong configuration\Subtask configuration:
    {'runner': {'type': 'constant', 'concurrency': 2, 'times': 10}, 'args': {'force_delete': False, 'flavor': {'name': 'm1.tiny'}, 'image': {'name': '^cirros.*uec$'}}, 'context': {'users': {'users_per_tenant': 2, 'tenants': 3}}}
    
    Reason:
     Invalid scenario argument: 'Image '{'name': '^cirros.*uec$'}' not found'`

There can be 2 reasons for that: 
1. You don't have an image starting with cirros and ending with uec
2. You have an image with the above name but it's private If you don't have the image you can download it using the following(0.3.4 is the latest version for now): 
```shell
    $ wget http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
    $ openstack image create --disk-format=qcow2 --file cirros-0.3.4-x86_64-disk.img --public cirros-uec
```
If you have a CirrOS image but it has the name cirros you can change the name in the task file or the name of the image. To change the name of the image:
```shell
    $openstack image set --name cirros-uec cirros
```
You can check if your image is private and change it to public:
```shell
$ openstack image show cirros-uec
+------------------+------------------------------------------------------+
| Field | Value |
+------------------+------------------------------------------------------+
| checksum | ee1eca47dc88f4879d8a229cc70a07c6 |
| container_format | bare |
| created_at | 2016-11-22T03:27:41Z |
| disk_format | qcow2 |
| file | /v2/images/15d13659-d6d2-4d4b-b6cc-819d03fdba7f/file |
| id | 15d13659-d6d2-4d4b-b6cc-819d03fdba7f |
| min_disk | 0 |
| min_ram | 0 |
| name | cirros-uec |
| owner | 7b0c5118f93b4399a12f5cf60c7b426e |
| protected | False |
| schema | /v2/schemas/image |
| size | 13287936 |
| status | active |
| tags | |
| updated_at | 2016-11-22T03:31:23Z |
| virtual_size | None |
| visibility | private |
+------------------+------------------------------------------------------+
$ openstack image set --public cirros-uec
```
