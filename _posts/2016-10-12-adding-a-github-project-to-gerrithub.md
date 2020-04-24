---
layout: single
permalink: /adding-a-github-project-to-gerrit/
title: "Adding a GitHub project to GerritHub"
tags: gerrit git
comments: true
---
If you have a GitHub project and want people to be able to contribute to your project by adding code and reviewing patches you can add it to [GerritHub](http://gerrithub.io/).  

**Gerrit** is a free, web-based team code collaboration tool. Software developers in a team can review each other's modifications on their source code using a Web browser and approve or reject those changes. It integrates closely with Git, a distributed version control system. (From Wikipedia).

### Adding your Project to GerritHub
1. Go to [GerritHub](http://gerrithub.io/)
2. If you haven't registered yet - Click on 'First Time Sign In' button on the upper right corner.
3. Click on the 'Sign In' Button on the upper right corner.
4. Enter your GitHub credentials (Unless you are already logged in to GitHub) &nbsp; You'll see the Dashboard: ![dashboard]({{ site.url }}/assets/images/dashboard.png)
5. Click on the bold GitHub at the top of the dashboard and then on 'Repositories'.
6. Mark the repository you want to import and click 'Import' -\> Next -\> Next

### Adding a SSH key
Before you can push your changes to Gerrit you need to add your SSH key: 
1. Click on your username (upper right corner) &nbsp;-\> Settings
2. On the next page choose SSH Public Keys . Then add your ssh public key. ![ssh_keys]({{ site.url }}/assets/images/ssh_keys-1-300x117.png)

### Submitting a change to your project
1. Go back to your Dashboard and click on Projects and search your project i.e. username/project-name
2. Click the Project and you'll see details about the project.
3. Install git and git-review For Fedora run:
```bash
    $ sudo dnf -y install git git-review
```
4. Copy the line starts with 'git clone' and run.
```bash
    $ git clone ssh://@review.gerrithub.io:29418//
```
5. Change directory to your project
```bash
    $ cd
```
6. Add Gerrithub as a remote named gerrit
```bash
    $ git remote add gerrit \
      ssh://@review.gerrithub.io:29418//
```
7. Set the gerrit remote
```bash
    $ git remote -s
```
8. Make your changes and commit 9. Show the latest commit and make sure you see a line with **Change-Id**
```bash
    $ git log -1
    
    commit 91666e02ac48f0922b48c180bf6a7eca082a940e
    Author: user <user@example.com>
    Date: Thu Oct 13 00:00:08 2016 +0000
    
    test
    
    Change-Id: I104eccd92cfda7da4744f0877f10ff15ff1ae16a
```
10. Submit your change to GerritHub
```bash
    $ git review
```
11. Go to the GerriHub Dashboard and click on My-\>Changes. You should see your patch Under the 'Outgoing reviews' ![after_change]({{ site.url }}/assets/images/after_change.png)

### Reviewing a Patch
For testing purpose you can approve the patch and see it merged in GitHub 

1. Click on the patch 
2. Click on 'Reply..' 
3. If you want to approve the patch you can choose '+2' and in the **Code-review** &nbsp;line and '+1' at the **Verified&nbsp;** line.
You should now see a submit Button ![submit.PNG]({{ site.url }}/assets/images/submit.png)
1. Click on the Submit button 
2. Go to your Github project and you should see your latest commit.

### Adding reviewers
I found a great explanation here:[https://github.com/midonet/midonet/wiki/Gerrit-Access-Rights-Policies](https://github.com/midonet/midonet/wiki/Gerrit-Access-Rights-Policies)
