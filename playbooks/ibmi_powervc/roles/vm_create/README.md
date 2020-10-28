vm_create
=================

Create a PowerVC VM

Role Variables
--------------

| Variable                | Type          | Description                                |
|-------------------------|---------------|--------------------------------------------|
| `cloud_name`            | str           | PowerVC name. Pickup one from clouds.yaml  |
| `vm_name`               | str           | PowerVC VM name                            |
| `vm_hostname`           | str           | VM hostname                                |
| `vm_group`              | str           | VM group                                   |
| `image`                 | str           | The name or id of the image                |
| `nics`                  | list of dicts | List of network interface definitions      |
| `compute_template`      | str           | Name or ID of VM flavor                    |
| `user_data`             | str           | Userdata passed to cloud-init              |


Example Playbook
----------------
```
- name: IBM i VM initalization
  hosts: localhost
  roles:
    - role: vm_create
      vars:
        cloud_name: 'pvc254'
        vm_name: 'newibmi'
        vm_hostname: 'neo143' 
        image: '74-svc'
        nics:
          - net-name: Net-pub
        vm_group: "Demo-ibmi"
        user_data: |
          {%- raw -%}#!/bin/sh
          system "chgusrprf usrprf(qciuser) password(passw0rd) status(*enabled) usrcls(*secofr) spcaut(*allobj *iosyscfg *jobctl *secadm *audit *service *splctl *savsys)"
          system "chgtcpsvr svrsrcval(*sshd) autostart(*yes)"  
          system "strtcpsvr *sshd"
          {% endraw %}
        compute_template: 'lj' 
```

License
-------

Apache-2.0
