---
layout: single
permalink: /using-git-completion-and-git-prompt/
title: "Using git-completion and git-prompt"
tags: git
comments: true
---
After working a little bit with Git there are two things (among others) which can help you be more productive: Commands completion and getting the status of your repository in the Shell prompt. This can be achieved by using git-completion and git-prompt.  
  
**Note:** I'm using Fedora and Bash as a shell - the location of the files may be different but the names should be the same. If you are using other shell such as Zsh you can search for the files with the appropiate suffix (e.g..zsh) . To add the above features - add the following to your ~/.bashrc file:

```shell
source /usr/share/doc/git-core-doc/contrib/completion/git-completion.bash
GIT_PS1_SHOWDIRTYSTATE=1
GIT_PS1_SHOWUNTRACKEDFILES=1
GIT_PS1_SHOWCOLORHINTS=true
PROMPT_COMMAND='__git_ps1 "\u@\h:\w" "\\\$ "'
source /usr/share/git-core/contrib/completion/git-prompt.sh
```

Some explanations about the above statements:

**source /usr/share/doc/git-core-doc/contrib/completion/git-completion.bash**

The first line is to enable git commands completion. If you type git co you'll see the two commands: commit and config.
```shell
    $ git co
    commit config
```
After you type 'm' the shell will complete the command to be 'git commit'
```shell
GIT_PS1_SHOWDIRTYSTATE=1
GIT_PS1_SHOWUNTRACKEDFILES=1
GIT_PS1_SHOWCOLORHINTS=true
```
The 3 environment files are for she git-prompt.sh script: GIT\_PS1\_SHOWDIRTYSTATE is used to show the state of repository state:
- Changing a file will be noted by '\*'  
- Adding a file to the stage are will be noted by '+'  

GIT\_PS1\_SHOWDIRTYSTATE is used to show untracked files. Adding untracked files will be noted by '%'  
GIT\_PS1\_SHOWCOLORHINTS will show the state by colors. On my machine - Untracked files and unstaged changed will be in red and staged ones will be green.  
PROMPT\_COMMAND='\_\_git\_ps1 "\u@\h:\w" "\\\$ "' is telling the shell to change the prompt using the git-prompt script. Another thing that is really important is that the current branch will be shown in parenthesis. Here is an example for using Git with the git-prompt: ![Git prompt with branch name and status in colors]({{ site.url }}/assets/images/git_propmpt_branch_and_colors.png)
