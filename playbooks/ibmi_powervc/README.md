This example creates a IBMi.

## Requirements

The Ansible OpenStack [os_server][1] module is used to create and delete
PowerVC VM instances. This requires:

* openstacksdk >= 0.12.0

This can be installed in the Ansible Python environment using pip:
`pip install "openstacksdk>=0.12.0"`

Authentication is handled by openstacksdk, which can be configured in many
ways. See [Configuring OpenStack SDK Applications][2] for details.

## Requisites
Before run playbook, configure PowerVC into the clouds.yml firstly.  
* `auth_url` (str): https://powervc:5000/v3
* `username` (str): User name of PowerVC
* `password` (str): Password of PowerVC 
* `project_name` (str): PowerVC project name
* `project_domain_name` (str): PowerVC domain name
* `user_domain_name` (str): User domain name
* `verify` (bool): When the access to PowerVC is done via a secure connection, openstacksdk will always verify the SSL cert by default. This can be disabled by setting verify to False

## Variables of create_vm.yml

The following variables are set by the user:
* `cloud_name` (str): PowerVC name. Please pickup one from clouds.yaml
* `vm_name` (str): PowerVC VM name
* `vm_hostname` (str): VM hostname
* `image` (str): OS image name. Please pickup one from the Image list in PowerVC.
* `nics` (list of dicts): List of network interface definitions. Please pickup one from the Network list in PowerVC
* `compute_template` (str): Name or ID of VM flavor, please pickup one from the list of flavor which can be found as "Compute templates" in PowerVC
* `user_data` (str): The userdata passed to cloud-init to initialize the instance

## Variables of configure_iasp.yml

The following variables are set by the user:
* `cloud_name` (str): PowerVC name. Please pickup one from clouds.yaml
* `vm_name` (str): PowerVC VM name
* `vm_hostname` (str): VM hostname
* `image` (str): OS image name. Please pickup one from the Image list in PowerVC.
* `nics` (list of dicts): List of network interface definitions. Please pickup one from the Network list in PowerVC
* `compute_template` (str): Name or ID of VM flavor, please pickup one from the list of flavor which can be found as "Compute templates" in PowerVC
* `user_data` (str): The userdata passed to cloud-init to initialize the instance
* `volume_name_prefix` (str): Name prefix of new volume
* `storage_template_id` (str): ID of PowerVC storage template
* `volume_size` (int): Size of the new volume
* `volume_number` (int): Number of volumes to be created
* `iasp_info` (list of dicts): List of iasp info.

License
-------

Apache-2.0

[1]: https://docs.ansible.com/ansible/latest/modules/os_server_module.html
[2]: https://docs.openstack.org/openstacksdk/latest/user/config/configuration.html

