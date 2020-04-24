---
layout: single
permalink: /how-tripleo-populates-the-etchosts-in-the-overcloud-nodes/
title: "How TripleO populates the /etc/hosts in the overcloud nodes"
tags: heat openstack tripleo
comments: true
---
TripleO (OpenStack On OpenStack) is a program that let you install and upgrade OpenStack using OpenStack.  
You can find out more at the [TripleO Documentation.](https://docs.openstack.org/developer/tripleo-docs/).  
TripleO is using Heat which is the OpenStack's orchestration project.  

In this post I'll cover how TripleO populates the /etc/hosts file for the overcloud nodes. TripleO Heat templates is located at [https://github.com/openstack/tripleo-heat-templates](https://github.com/openstack/tripleo-heat-templates)  

**overcloud.j2.yaml** defines the base Template for each overcloud node. Among other things it has the following resource definition:
```yaml
hostsConfig:
    type: OS::TripleO::Hosts::SoftwareConfig
    properties:
      hosts:
        list_join:
        - "\n"
        - - if:
            - add_vips_to_etc_hosts
            - {get_attr: [VipHosts, value]}
            - ''
        -
{% for role in roles %}
          - list_join:
            - ""
            - {get_attr: [{{role.name}}, hosts_entry]}
{% endfor %}
```
It converts a list of entries to a string which is passed to OS::TripleO::Hosts::SoftwareConfig hosts property. In the second part the hosts\_entry for each role is added to the list.  

Let's look how the host\_entry for the Controller role is built: In the file **puppet/controller-role.yaml** there is:
```yaml
hosts_entry:
    description: >
      Server's IP address and hostname in the /etc/hosts format
    value:
      str_replace:
        template: |
          PRIMARYIP PRIMARYHOST.DOMAIN PRIMARYHOST
          EXTERNALIP EXTERNALHOST.DOMAIN EXTERNALHOST
          INTERNAL_APIIP INTERNAL_APIHOST.DOMAIN INTERNAL_APIHOST
          STORAGEIP STORAGEHOST.DOMAIN STORAGEHOST
          STORAGE_MGMTIP STORAGE_MGMTHOST.DOMAIN STORAGE_MGMTHOST
          TENANTIP TENANTHOST.DOMAIN TENANTHOST
          MANAGEMENTIP MANAGEMENTHOST.DOMAIN MANAGEMENTHOST
          CTLPLANEIP CTLPLANEHOST.DOMAIN CTLPLANEHOST
        params:
          PRIMARYIP: {get_attr: [NetIpMap, net_ip_map, {get_param: [ServiceNetMap, ControllerHostnameResolveNetwork]}]}
          DOMAIN: {get_param: CloudDomain}
          PRIMARYHOST: {get_attr: [Controller, name]}
          EXTERNALIP: {get_attr: [ExternalPort, ip_address]}
          EXTERNALHOST: {get_attr: [NetHostMap, value, external, short]}
          INTERNAL_APIIP: {get_attr: [InternalApiPort, ip_address]}
          INTERNAL_APIHOST: {get_attr: [NetHostMap, value, internal_api, short]}
          STORAGEIP: {get_attr: [StoragePort, ip_address]}
          STORAGEHOST: {get_attr: [NetHostMap, value, storage, short]}
          STORAGE_MGMTIP: {get_attr: [StorageMgmtPort, ip_address]}
          STORAGE_MGMTHOST: {get_attr: [NetHostMap, value, storage_mgmt, short]}
          TENANTIP: {get_attr: [TenantPort, ip_address]}
          TENANTHOST: {get_attr: [NetHostMap, value, tenant, short]}
          MANAGEMENTIP: {get_attr: [ManagementPort, ip_address]}
          MANAGEMENTHOST: {get_attr: [NetHostMap, value, management, short]}
          CTLPLANEIP: {get_attr: [Controller, networks, ctlplane, 0]}
CTLPLANEHOST: {get_attr: [NetHostMap, value, ctlplane, short]}
```
One entry is:
```
PRIMARYIP PRIMARYHOST.DOMAIN PRIMARYHOST
```
If we'll search for the PRIMARYHOST in the params section:
```
PRIMARYHOST: {get_attr: [Controller, name]}
```
So **PRIMARYHOST** is replaced by the Controller's name attribute.  
The Controller resource is defined in the same file.  
So now that we have the list of entries that should be put in the /etc/hosts of the overcloud nodes - how it's being written ? 

We saw that the hosts\_entry is passed to the resource OS::TripleO::Hosts::SoftwareConfig. This resource is defined in overcloud-resource-registry-puppet.j2.yaml
```
OS::TripleO::Hosts::SoftwareConfig: hosts-config.yaml
```
So it points us to **hosts-config.yaml**. The current content of **hosts-config.yaml** is:
```yaml
heat_template_version: ocata
description: 'All Hosts Config'

parameters:
  hosts:
    type: string

resources:
  hostsConfigImpl:
    type: OS::Heat::SoftwareConfig
    properties:
      group: script
      inputs:
        - name: hosts
          default:
            list_join:
              - ' '
              - str_split:
                - '\n'
                - {get_param: hosts}
      config: {get_file: scripts/hosts-config.sh}

outputs:
  config_id:
    description: The ID of the hostsConfigImpl resource.
    value:
      {get_resource: hostsConfigImpl}
  hosts_entries:
    description: |
      The content that should be appended to your /etc/hosts if you want to get
      hostname-based access to the deployed nodes (useful for testing without
      setting up a DNS).
    value: {get_param: hosts}
  OS::stack_id:
    description: The ID of the hostsConfigImpl resource.
    value: {get_resource: hostsConfigImpl}
```
So it get the hosts parameter and pass is as an environment variable to the hosts-config.sh script which writes to the /etc/hosts file (Among other files)
