---
layout: single
permalink: /using-puppet-defined-types/
title: "Using Puppet defined types"
tags: puppet
comments: true
---
Puppet is a configuration management tool. Defined type is a way to create a new type of resource to make our life easier. For example, we can use a new defined type localuser which can add a user and it's home directory.



{% highlight puppet %}
define localuser ($ensure='present'){
  user {$name:
     ensure => $ensure,
     home => "/home/${name}"
  }
  if $ensure == 'present' {
    file { "/home/${name}":
      ensure => 'directory',
      owner => "$name",
    }
  } else {
    file { "/home/${name}":
      ensure => 'absent',
      force => 'true',
    }
  }
}

$localensure = 'present'
localuser {'alex': ensure => $localensure}
localuser {'david': ensure => $localensure}
localuser {'tom': ensure => $localensure}
{% endhighlight %}

In the above manifest we defining a new resource type and then using it to define 3 users and their home directories. The $name variable is derived from the title of the resource. For the first user the $name is 'alex'. We are using this variable in creating the user and his home directory. The three last lines can be also written as:
```puppet
localuser {['alex', 'david', 'tom']:
     ensure => $localensure
}
```
This time we are using an array of user names instead of repeating 3 times. If we want to reuse this type we can create a module named localusers (replace itzikb with your username or any prefix you wish):
```shell
$ puppet module generate itzikb-localusers --skip-interview
```

Then we add the following manifests:_localusers/manifests/users.pp_ 
```puppet
class localusers::users(){
  define localuser ($ensure='present'){
    user {$name:
       ensure => $ensure,
       home => "/home/${name}"
    }
    if $ensure == 'present' {
       file { "/home/${name}":
         ensure => 'directory',
         owner => "$name",
    }
    } else {
        file { "/home/${name}":
          ensure => 'absent',
          force => 'true',
        }
   }
  }

_localusers/manifests/init.pp_ 

class localusers($listusers=[],$ensure='present') {
  localusers::users::localuser{$listusers:
    ensure => $ensure,
  }
}
```
We then build our module
```shell
$ puppet module build localusers
```

Installing the module:
```shell
$ puppet module install
```

We can the create a manifest myusers.pp
```shell
class {"localusers":
    listusers => ['moo','foo','bar'],
}
```
And apply the manifest:
```shell
$ puppet apply myusers.pp
```
If we want to remove all the users and their home directories we can pass ensure =\> 'absent' to the class:
```puppet
class {"localusers":
    listusers => ['moo','foo','bar'],
    ensure => 'absent',
}
```

Suppose we have two hosts : node1.example.com and node2.example.com and we want to have the users 'moo','foo' and 'bar' on node1 but just 'moo' and 'foo' on node2.  

We'll write the following /etc/puppet/manifests/site.pp:
```puppet
node 'node1.example.com' {
  class {"localusers":
    listusers => ['moo','foo','bar'],
  }
}
node 'node2.example.com' {
  class {"localusers":
    listusers => ['moo','foo'],
  }
}
```
