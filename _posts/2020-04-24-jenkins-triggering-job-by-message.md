---
layout: single
permalink: /jenkins-trigger-message/
title: "Triggering Jenkins jobs using messages"
tags: jenkins
comments: true
---
[**Jenkins**](https://jenkins.io/) is an automation server that can be used for&nbsp;Continuous Integration and Continuous Delivery
[**RabbitMQ**](https://www.rabbitmq.com/) is a message broker

In this post we'll see how to trigger a job by sending a message through a message broker.
I'm assuming that RabbitMQ is installed and running.
## Creating an exchange and a queue in RabbitMQ
**First we'll create an exchange and a queue**
```
$ rabbitmqadmin declare exchange name=myexchange type=fanout
$ rabbitmqadmin declare queue name=myqueue durable=true
```

**Verify that you have the exchange and the queue are defined**
```
$ sudo rabbitmqctl list_exchanges
Listing exchanges for vhost / ...
name	type
	direct
amq.direct	direct
amq.headers	headers
amq.match	headers
amq.topic	topic
amq.fanout	fanout
myexchange	fanout
amq.rabbitmq.trace	topic

$ sudo rabbitmqctl list_queues
Timeout: 60.0 seconds ...
Listing queues for vhost / ...
name	messages
myqueue	0
```

## Install and configure the JMS Messaging Plugin

You'll want to make sure that the plugin is installed
Go to **Manage Jenkins**→**Configure System**
![manage_plugins]({{ site.url }}/assets/images/jenkins_manage_plugins.png)

Mark the JMS Messaging plugin and Use **Install without restart**
![install_plugin]({{ site.url }}/assets/images/jenkins_install_jms.png)
Go to **ManageJenkins** →**Configure System**
![configure_system]({{ site.url }}/assets/images/jenkins_configure_system.png)
Under **CI Monitor** add RabbitMQ
![jenkins_ci_monitor]({{ site.url }}/assets/images/jenkins_ci_monitor.png)

Then fill in the details:
![jenkins_ci_monitor]({{ site.url }}/assets/images/jenkins_ci_monitor_details.png)

**Name** - The name you want to call the broker  
**Hostname**  - RabbitMQ hostname  
**Port Numbe** - 5672  
**Virtual Host** - /  (Unless you created a different one)  
**Topic** - blue (Otherwise you'll need to override it in the job)    
**Exchange**  - myexchange  (In our example)  
**Queue -** myqueue (In our example)  
**Authentication Method**  - Choose Username and password (default for rabbitmq is guest:guest)  

Hit **Test Connetion** and make sure the connection to the message broker is successful and then Click **Save**
## Create a Notifier Job
Create a new Freesyle job
![jenkins_ci_monitor]({{ site.url }}/assets/images/create_notifier_job.png)
Under **Post-build Actions** Add post-build action and choose **CI notifier**

![jenkins_ci_monitor]({{ site.url }}/assets/images/jenkins_post_builder_ci_notifier.png)
Choose the broker and fill the Content message
In our example fill {"foo":"boo"} in the Content message
![jenkins_ci_monitor]({{ site.url }}/assets/images/jenkins_ci_notifier_details.png)
## Create the job to be triggered
Create a new freestyle job

Under **Build Triggers** choose **CI event** and **Schedule a new job for every triggering message**.

Add Message Checks and fill the Field and Expected value
In our example:  
**Field:** foo  
**Expected value:** ^boo$  
  
![jenkins_ci_monitor]({{ site.url }}/assets/images/jenkins_notified_job_details.png)

## Making sure everything works
All that you need right now is to trigger the first job and then watch the second one triggered

You can change the first job's notification message to boo1 instead of boo and trigger the first job again and verify that the second job is not triggered

## Troubleshooting
The best way to figure out why something isn't working is to look at Jenkins Log (Manage Jenkins → System Log)
