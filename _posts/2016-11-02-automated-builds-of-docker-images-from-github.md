---
layout: single
permalink: /automated-builds-of-docker-images-from-github/
title: "Automated builds of Docker images from GitHub"
tags: docker
comments: true
---
[Docker](https://www.docker.com/) is a platform for running applications in containers.  
You can specify how to build an image by Writing a Dockerfile file. Once your have your Dockerfile in a GitHub (Or Bitbucket) repository it's possible to trigger DockerHub to build a Docker image for you automatically so when you update your Dockerfile the Docker image is being updated in DockerHub as well.  

The process of doing an automated build is covered [here](https://docs.docker.com/docker-hub/github/#/creating-an-automated-build)&nbsp;so I'll just add&nbsp;one point that may not be clear.  

When you create an automated build in DockerHub the image is only being built when you update your GitHub repository.  
You can trigger a build when going to Dashboard-\> Your repository -\> Build Settings and then Click on Trigger as shown below

![trigger_build]({{ site.url }}/assets/images/trigger_build.png)  

If you go to the **Build Details** Tab you should see that the build is in **Queued** status.  
![docker_queued]({{ site.url }}/assets/images/docker_queued.png)&nbsp;  
It can take couple of minutes until the build is finished (You need to refresh the page to see the new status). &nbsp;  

![docker_success]({{ site.url }}/assets/images/docker_success.png)  
