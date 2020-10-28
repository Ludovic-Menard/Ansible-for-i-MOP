vm_volume_attach
=================

Create a new volume and attach it to a PowerVC VM

Role Variables
--------------

| Variable              | Type          | Description                                      |
|-----------------------|---------------|--------------------------------------------------|
| `cloud_name`          | str           | PowerVC name. Pickup one from clouds.yaml        |
| `volume_size`         | int           | Size of to-be-created volume                     |
| `volume_name`         | str           | Display name of to-be-created volume             |
| `storage_template_id` | str           | ID of PowerVC storage template                   |
| `vm`                  | str           | Name or ID of VM you want to attach a volume to  |

Example Playbook
----------------
```
- name: IBM i VM initalization
  hosts: localhost
  roles:
    - role: vm_volume_attach
      vars:
        volume_number: 4
        volume_size: 10
        volume_name: 'demo'
        storage_template_id: '70866ebf-0db5-4b12-86ed-0e838d593458'
        vm: 'newibmi1'
```
        
License
-------

Apache-2.0
