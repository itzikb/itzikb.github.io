---
layout: single
permalink: /delete-volume-error-attached/
title: "Delete a volume in error state when attached status is wrong"
tags: openstack
comments: true
---

Having the following volumes that are in error state:
```
$ openstack volume list --all

-+--------------------------------------+---------------------------------------------------------------+--------+------+---------------------------------------------------------------+
| ID                                   | Name                                                          | Status | Size | Attached to                                                   |
+--------------------------------------+---------------------------------------------------------------+--------+------+---------------------------------------------------------------+
| d212825f-2b92-4130-a0e5-644ef4c2aae9 | ostest-f8gbn-dynamic-pvc-2e8bbf81-f4d9-4da2-9629-7ae2d8f24ed0 | error  |    2 | Attached to 23a95f48-077f-4afe-9df8-6ebd08e740eb on /dev/vdb  |
| 6fc62b1a-6daf-45d5-b778-62981d57e858 | ostest-f8gbn-dynamic-pvc-efd16155-fc88-4520-ba40-4208ea957265 | error  |    2 | Attached to b8ecf79b-f7ff-4853-a5e7-22ce5108a41e on /dev/vdb  |
+--------------------------------------+---------------------------------------------------------------+--------+------+---------------------------------------------------------------+

```

Trying to detach the volumes
```
$ openstack server remove volume  23a95f48-077f-4afe-9df8-6ebd08e740eb ostest-f8gbn-dynamic-pvc-2e8bbf81-f4d9-4da2-9629-7ae2d8f24ed0 ; openstack server remove volume b8ecf79b-f7ff-4853-a5e7-22ce5108a41e ostest-f8gbn-dynamic-pvc-efd16155-fc88-4520-ba40-4208ea957265
No server with a name or ID of '23a95f48-077f-4afe-9df8-6ebd08e740eb' exists.
No server with a name or ID of 'b8ecf79b-f7ff-4853-a5e7-22ce5108a41e' exists.
```

Apparently the servers are no longer there and the attachment status is wrong  

So we need to set the Attached status to detached  
```
$ openstack volume set --detached d212825f-2b92-4130-a0e5-644ef4c2aae9                                                      
$ openstack volume set --detached 6fc62b1a-6daf-45d5-b778-62981d57e858
```

Now Running the delete command should work
