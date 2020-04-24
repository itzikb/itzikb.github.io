---
layout: single
permalink: /running-jenkins-job-builder-using-docker/
title: "Running Jenkins Job Builder using Docker"
tags: docker jenkins
comments: true
---
[**Jenkins**](https://jenkins.io/) is an automation server that can be used for&nbsp;Continuous Integration and Continuous Delivery.  
[Docker](https://www.docker.com/) is a platform for running applications in containers. The Mantra that says if you do something more than one time than you must automate it.  
Sometimes there is so much to do that this simple mantra is forgotten. I just started to work with Jenkins Job Builder (JJB) and then it occurred to me that I'll probably move to other things and then forget all about it. So I figured one way to do it is to build a Docker image. It's probably an overkill for such a task that probably can be done by running a script to build a virtualenv but nevertheless an enjoyable way to do it ..  

Assuming there is a directory **jobs&nbsp;** that holds 2 files:&nbsp; **myjenkins.conf** and **myjob.yml**  
**myjenkins.conf** is a JJB configuration file (Refer to&nbsp;http://docs.openstack.org/infra/jenkins-job-builder/execution.html# )  
**myjob.yml** is a YAML file that&nbsp;describes&nbsp;one or more jobs.  

Create a directory called jjb
  {% highlight bash %}
    $ mkdir jjb
  {% endhighlight %}

In the directory jjb create a file named **Dockerfile** with the following content:

  {% highlight bash %}
    FROM centos:7
    ENV VERSION 1.4.0
    RUN yum -y localinstall https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-7.noarch.rpm
    RUN yum -y install git python-pip
    RUN git clone https://github.com/openstack-infra/jenkins-job-builder.git
    WORKDIR jenkins-job-builder
  {% endhighlight %}

Register to Dockerhub at&nbsp;https://hub.docker.com/ if you don't already have a user Login to Docker hub, build and push the image to Docker hub (replace user with your user name)

  {% highlight bash %}
    $ sudo docker login 
    $ sudo docker build -t user/jjb:latest jjb/
    $ sudo docker push user/jjb
  {% endhighlight %}

**Check your job definition**

  {% highlight bash %}
    $ sudo docker run \
      --rm --volume "$PWD":/opt/jenkins-job \
      --workdir /opt/jenkins-job user/jjb  jenkins-jobs \
      --conf  jobs/myjenkins.conf test jobs/myjob.yml
  {% endhighlight %}

**--rm** flag is telling Docker to&nbsp;automatically clean up the container and remove the file system when the container exits  
**--volume** is for defining a mount point in the container- in our case it will mount our current directory as /opt/jenkins-job in the container  
**--workdir** is the default directory the container will move to when it starts  
**user/jjb** is the image name (you have to replace user with your own name)  
**jenkins-job&nbsp;** is the JJB program and the rest is it's argumentsUpload(/Update) the job  

  {% highlight bash %}
    $ sudo docker run \
      --rm --volume "$PWD":/opt/jenkins-job \
      --workdir /opt/jenkins-job user/jjb  jenkins-jobs \
      --conf  jobs/myjenkins.conf update jobs/myjob.yml
  {% endhighlight %}

Once you have your image at Docker hub you can &nbsp;run the JJB commands without building it first.  
**Note:** You can use my image instead of building your own - replace the user with itzikb.

### **Troubleshooting**

#### Permission denied
You may encounter permission denied issue when accessing the volumes. You may either choose to disable SELinux temporarily by running

  {% highlight bash %}
    $ sudo setenforce 0
  {% endhighlight %}

or persisent

  {% highlight bash %}
    $ sudo chcon -Rt svirt_sandbox_file_t .
  {% endhighlight %}

#### docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?
Your probably need to start the docker daemon by running:

  ``` shell
    $ sudo systemctl start docker
  ```

