---
layout: single
permalink: /ansible-basics/
title: "Ansible notes"
tags: ansible
comments: true
---
**[Ansible](https://www.ansible.com/)&nbsp;**is an agent-less configuration management tool using YAML for its playbooks. 
Here are some notes about using Ansible that I find useful.  

**Configuration files precedence**  
Anisble configuration files can be read from couple of places in order or precedence:  
1. By specifying the ANSIBLE\_CONFIG environment variable - export ANSIBLE\_CONFIG=/path/to/myconfig
2. ansible.cfg at the directory from which you are running ansible
3. At the user's home directory - /home/user/.ansible.cfg
4. At /etc/ansible/ansible.cfg Only one configuration file is active - there is no merge between the files.

**List all the hosts in a group**  
If my inventory file is:  
``` ini
    [mygroup]
    server1
    server2
```
Then running the following will show all the servers in the group 'mygroup'  
```bash
    $ ansible mygroup --list-hosts
     hosts (2):
     server1
     server2
```
**Form a group from a other groups in inventory file**  
If we have 2 groups 'mygroup1' and 'mygroup2' we can have another group that will include both groups:
```ini
    [mygroup1]
    server1
    [mygroup2]
    server2
    [allgroups:children]
    mygroup1
    mygroup2
```
We can verify the inventory by running
```bash
    $ ansible mygroup1 --list-hosts
```
```ini
     hosts (1):
     server1
```    
```bash    
    $ ansible allgroups --list-hosts
```
```ini
     hosts (2):
     server1
     server2
```
**Using Adhoc commands**  
Sometimes it's useful to run a command on a remote host without using a playbook
```bash
    $ ansible server1 -m command -a 'date'
```
**server1** is the server we are running the commands on 
**-m** &nbsp;is the module we are using ( command in our case)
**-a** &nbsp;are the parameters we are passing to the module ('/bin/date' ).
You can all the ansible modules here, not just the command module.
Parameters are separated by spaces

```bash
    $ ansible localhost -m copy -a 'src=/tmp/1 dest=/tmp/2'
```

**Static/Dynamic and multiple inventory files**  
The basic inventory file is the basic one as show earlier. It's possible to have a script that will populate the inventory dynamically - the script has to be executable (more can be found [here](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)). It's possible to have several hosts files - you are using a directory. Keep in mind that the files are examined in alphabetical order. &nbsp; &nbsp; &nbsp;
