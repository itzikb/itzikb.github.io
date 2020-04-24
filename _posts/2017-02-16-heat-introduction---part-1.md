---
layout: single
permalink: /heat-introduction-part-1/
title: "Heat introduction - Part 1"
tags: heat openstack
comments: true
---
Heat is the Orchestration project of OpenStack. It enables to manage the life cycle of applications. We can use it to create an entire application by creating networks, servers etc. This is done by writing templates which are text files using the YAML language.  

**Note:** I assume you already have an access to an OpenStack cloud.

### Creating our first stack

Create a file called first-stack.yaml with the below content:

  ```yaml
  heat_template_version: 2016-10-14

  description: Launching a single compute instance

  resources:
    my_instance:
  type: OS::Nova::Server
  properties:
    key_name: my_key
    image: cirros
    flavor: m1.tiny
  ```

Source your credentials file (In our example it's overcloudrc):
  ```shell
  $ source overcloudrc
  ```

Create the stack
```shell
$ openstack stack create -t first-stack.yaml firststack
```
List all stacks
```shell
$ openstack stack list
```
The status of the stack should be CREATE\_IN\_PROGRESS and after a while CREATE\_COMPLETE ![]({{ site.url }}/assets/images/firststack-1.png)

Verify that the instance is running:
```shell
$ openstack server list
```
 ![]({{ site.url }}/assets/images/server-list.png)

In the above example we created one resource which we called my\_instance. The type of resource is OS::Nova::Server which is an instance. This type of resource has few properties that we can supply so Heat will send Nova when creating the instance. Each type has it's own properties - some are required and some are optional. We specified the Heat template version for the Newton release (2016-10-14).

In our example before creating the stack we should have:
1. A key named **my\_key**
2. An image named **cirros**
3. A flavor named **m1.tiny**
If any of the above doesn't exist the creation of stack fails.

### Suppling parameters
In our first example the properties of our Instance was hard coded in the template file.

### Using Parameters
We obviously want to be able to use it to create it with custom values. The following template will show how:
```yaml
    heat_template_version: 2015-04-30
    description: Simple template to deploy a single compute instance
    
    parameters:
      key_name:
        type: string
        label: Key Name
        description: Name of key-pair to be used for compute instance
      image_id:
        type: string
        label: Image ID
        description: Image to be used for compute instance
        default: m1.tiny
      instance_type:
        type: string
        label: Instance Type
        description: Type of instance (flavor) to be used
        default: m1.tiny
        constraints:
          - allowed_values: [m1.tiny, m1.small]
            description: m1.tiny or m1.small should be used
      name:
        type: string
        label: Instance name
        description: The name of the instance
        default: myserver
    
    resources:
      my_instance:
        type: OS::Nova::Server
        properties:
          key_name: { get_param: key_name }
          image: { get_param: image_id }
          flavor: { get_param: instance_type }
          name: { get_param: name }
    outputs:
      instance_ip:
        description: The IP address of the deployed instance
        value: { get_attr: [my_instance, first_address] }
```
In the above example we specified couple of parameters that the user can supply to create the server. We supplied default values for all the properties except for the key\_name which should be supplied by the user. For the instance\_type property we used a constraint to limit the flavor of the instance to be m1.tiny of m1.small We can supply the parameters by using the parameter argument (or -P):

```shell
$ openstack create -t firststack.yaml --parameter "key_name=mykey;name=myserver" firststack
```

We can also supply an environment file. The environment file is a YAML file that can be sued to override the resources implementation (more about this on a following post) and passing parameters.
```yaml
    parameters:
      key_name: mykey
      image_id: fedora
      instace_type: m1.small
      name: myserver
    
    parameter_defaults:
      KeyName: heat_key
```
### Deleting the stack
When we want to delete the stack:
```shell
    $ openstack stack delete firststack
```
**Note:** ALL the resources in this stack will be deleted! You can verify the removal of this stack by listing the stacks as above and you shouldn't see the stack in the list 

**Links:**   
[OpenStack Resource Types reference](http://docs.openstack.org/developer/heat/template_guide/openstack.html)  
[The definition of OS::Nova::Server](http://docs.openstack.org/developer/heat/template_guide/openstack.html#OS::Nova::Server)  
[Heat template version list](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#heat-template-version)  
[Heat parameter constraints](http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#hot-spec-parameters-constraints)  
