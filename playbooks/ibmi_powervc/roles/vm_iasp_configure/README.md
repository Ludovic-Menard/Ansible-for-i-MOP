vm_iasp_configure
=================

Configure not-configured disks into iASP. If the specified iASP doesn't exist, create one. 

Role Variables
--------------

| Variable                | Type          | Description                                |
|-------------------------|---------------|--------------------------------------------|
| `iasp_info `            | list of dicts | The configuration setting of iASP.         |


Example Playbook
----------------
```
- name: IBM i VM initalization
  hosts: new_vm 
  roles:
    - role: vm_iasp_configure
      vars:
        iasp_info:
          - {'iasp_name': 'demoiasp1', 'iasp_capacity': 0.5}
          - {'iasp_name': 'demoiasp2', 'iasp_capacity': 1}
```

License
-------

Apache-2.0
