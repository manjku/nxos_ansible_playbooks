configure_grpc
===================
This role can be used to configure and unconfigure GRPC on nxos

Requirements
------------
- `gnmi.pfx` file must be present on bootflash needed for importing trustpoint configs. 

Role Variables
--------------

| Variable                | Required | Default | Choice       |   Description                     |
|-------------------------|----------|---------|--------------|-----------------------------------|
| configure_grpc_config   | no       | true    | true/false   |  true: Configure GRPC on nxos     |
|                         |          |         |              |  false: Unconfigre GRPC on nxos   |
| pfx_file                | yes      |         |              |  pfx file such as gnmi.pfx        |
| pfx_passphrase          | yes      |         |              |  pfx_passphrase for pfx_file      |
| trustpoint              | yes      |         |              |  Name of the trustpoint on nxos   |

Tags
----
Following tags can be used at the task level while including this role to execute particular function:

- `configure_grpc_config`: Only configure GRPC on nxos
- `verify_pfx_file_present`: Only verify pfx_file is present in nxos bootflash
- `verify_gprc_enabled`: Only Verify GRPC server running on nxos
- `unconfigure_grpc_config`: Unconfigure GRPC 
- `verify_gprc_disabled_nxos`: Verify GRPC server is disabled on nxos


Example Playbook
----------------
    - hosts: N1
      #gather_facts: no
      serial: 1
      tasks: 
        - name: Configure GRPC on router
          include_role: 
            name: configure_grpc
          vars:
            pfx_file: gnmi.pfx
            pfx_passphrase: test_pass
            trustpoint: gnmi
          when: ansible_net_system == "nxos" 
          tags: [verify_pfx_file_present, configure_grpc_config, verify_gprc_enabled, unconfigure_grpc_config]
          # Use any of the tag listed in tags during playbook execution cli using --tags option to execute only task with that tag
