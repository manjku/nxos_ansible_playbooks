nxos_copy_file_bootflash
=========
This role can be used to copy the files from Ansible controller node to nxos bootflash.  

Requirements
------------

Files must be present on the controller nodes.

Role Variables
--------------

| Variable                | Required | Default | Comments                               |   Description                                     |
|-------------------------|----------|---------|----------------------------------------|---------------------------------------------------|
| local_file_location     | yes      |         | local_file_location: /tmp/gnmi.pfx     |  location of the file on Ansible node             |
| nxos_file               | yes      |         | nxos_file: gnmi.pfx                    |  destination name of the file in nxos bootflash   |
| nxos_delete_file        | no       |  true   |                                        |  delete nxos_file in nxos before copying          |
| verify_file_copy        | no       |  true   |                                        |  Verify file is copied in nxos                    |

Tags
----
Following tags can be used at the task level while including this role to execute particular function:

- `copy_file`: Copy only file from Ansible controller node to nxos bootflash
- `delete_file`: delete nxos_file from the nxos bootflash
- `verify_copy_file`: Only Verify nxos_file is present after copying nxos_file to bootflash


Example Playbook
----------------

    - hosts: N1
      #gather_facts: no
      serial: 1
      tasks: 
        - name: Copy certificates from server to the router
          include_role: 
            name: nxos_copy_file_bootflash
          vars:
            local_file_location: "{{inventory_dir}}/certificate-gnmi-eve-ng-lab/gnmi.pfx"
            nxos_file: gnmi.pfx
          tags: [copy_file, verify_copy_file, delete_file]
          # role will be in included with any of the listed tags, however only the tasks(within role) with the tag supplied during ansible-playbook cli
          # get executed. 
          # ansible-playbook test.yml --tags copy_file will only copy the file task in the role
