---
layout: single
permalink: /use-oc-debug-in-openshift/
title: "Use an openshift client debug option"
tags: openshift
comments: true
---
Red Hat® OpenShift® is an enterprise-ready Kubernetes container platform built for an open hybrid cloud strategy. It provides a consistent application platform to manage hybrid cloud, multicloud, and edge deployments.  

If you want to run commands on an Openshift node (A master or a worker), you can use the Openshift client debug command.  

First, we list our nodes  
```
$ oc get nodes
NAME                          STATUS   ROLES    AGE   VERSION
ostest-s4dr6-master-0         Ready    master   36m   v1.23.5+3afdacb
ostest-s4dr6-master-1         Ready    master   36m   v1.23.5+3afdacb
ostest-s4dr6-master-2         Ready    master   36m   v1.23.5+3afdacb
ostest-s4dr6-worker-0-gxcs4   Ready    worker   13m   v1.23.5+3afdacb
ostest-s4dr6-worker-0-vb6n9   Ready    worker   13m   v1.23.5+3afdacb
ostest-s4dr6-worker-0-wvwrt   Ready    worker   13m   v1.23.5+3afdacb
```

Now we can open an interactive session on one of the nodes:  
```
$ oc debug node/ostest-s4dr6-worker-0-gxcs4
Starting pod/ostest-s4dr6-worker-0-gxcs4-debug ...
To use host binaries, run `chroot /host`
Pod IP: 172.16.0.95
If you don't see a command prompt, try pressing enter.
sh-4.4# chroot /host
sh-4.4# touch /tmp/2
```

We can run a specific command:  
```
$ oc debug -q node/ostest-s4dr6-worker-0-gxcs4 -- chroot /host sudo chronyc tracking
Reference ID    : 00000000 ()
Stratum         : 0
Ref time (UTC)  : Thu Jan 01 00:00:00 1970
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 1.000000000 seconds
Root dispersion : 1.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Not synchronised
```
