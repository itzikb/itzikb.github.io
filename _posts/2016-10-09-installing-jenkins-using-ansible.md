---
layout: single
permalink: /installing-jenkins-using-ansible/
title: "Installing Jenkins using Ansible"
tags: ansible jenkins
comments: true
---
[**Jenkins**](https://jenkins.io/) is an automation server that can be used for&nbsp;Continuous Integration and Continuous Delivery.  
**[Ansible](https://www.ansible.com/)&nbsp;**is an agent-less configuration management tool using YAML for its playbooks. Installing a Jenkins server is easy with geerlingguy.jenkins role.  

**Note:** I'm using Fedora in the following example. If you are using another distro then you'll have to change the commands for the packages installation.
### Create the roles directory
```bash
    $ mkdir roles
```
### Install Ansible and python2-dnf packages
```bash
    $ sudo dnf -y install ansible python2-dnf
```
### Install the&nbsp;geerlingguy.jenkins role
```bash
    $ ansible-galaxy install -p roles geerlingguy.jenkins
```
### Create a file plugins.yml with all the plugins you need
You can use [The following YAML file](https://github.com/itzikb/jenkins/blob/master/ansible/plugins.yml) as a reference    
### Create a playbook jenkins.yml
```yaml
    - name: Installing Jenkins
      hosts: localhost
      pre_tasks:
        - include_vars: plugins.yml
      roles:
         - { role: geerlingguy.jenkins, jenkins_plugins: "{{ plugins }}" }
```
Change hosts:localhost based on your environment
### Run the playbook
```bash
    $ ansible-playbook -b --ask-sudo-pass jenkins.yml
```
You should be able to access the Jenkins server at http://\<Jenkins Server IP\>:8080 with User admin and password admin
