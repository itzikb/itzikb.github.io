---
layout: single
permalink: /how-to-import-plugins-from-one-jenkis-server-to-another-one/
title: "How to import plugins from one Jenkins server to another one"
tags: jenkins
comments: true
---
[**Jenkins**](https://jenkins.io/) is an automation server that can be used for&nbsp;Continuous Integration and Continuous Delivery. Jenkins has a nice [python wrapper for the Jenkins REST API](https://python-jenkins.readthedocs.io/en/latest/).  
I needed it one time when I wanted to have the same plugins on my Jenkins server as another Jenkins server. Here's a simple python script that does the work:
``` python
    import jenkins
    
    server1 = jenkins.Jenkins(username='myuser',
                              password='mypassword',
                              url='http://192.168.100.100:8080/')
    server2 = jenkins.Jenkins(username='myuser',
                              password='mypassword',
                              url='http://192.168.100.101:8080/')
    
    plugins = server1.get_plugins_info()
    active_plugins=[str(p['shortName']) for p in plugins if p['active'] is True]
    
    for p in active_plugins:
        server2.install_plugin(p)
```
I haven't find a way to restart the server through this wrapper so you can just go to http://server:port/restart and it will do the job.
