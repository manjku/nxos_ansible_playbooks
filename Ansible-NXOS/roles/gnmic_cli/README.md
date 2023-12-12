gnmic_cli
=========
This role can be used to execute gnmic client cli on linux hosts to retreive capabilites and get data from routers based on yang models.
`gnmi_output` variable is set by this role with the gnmic cli output fetched from the router 

Details of gnmic client installatin and usage can be found:

https://aristanetworks.github.io/openmgmt/examples/gnmi-clients/gnmic/ 

https://gnmic.openconfig.net/user_guide/configuration_intro/

gnmic cli can be invoked in two ways:
- `passing parameters directly to gnmic cli`: 

       gnmic -a host:50051 -u <username> -p <password> --skip-verify  --tls-ca  ~/ansible_dir/nxos_ansible_playbooks/Ansible-NXOS/certificate-gnmi-eve-ng-lab/gnmi.pem get --path /System/name --path /System/ch-items/model --path /System/showversion-items

- `using gnmi config file with gnmic cli`: 

       gnmic --config ~/ansible_dir/nxos_ansible_playbooks/Ansible-NXOS/gnmi_configs/nxos_gnmi_config.yaml get --path /System/ch-items/model --path /System/showversion-items


Requirements
------------
- `grpc must be enabled on the router and .pem file must be present on the linux host corresponding to gnmi.pfx file in nxos bootflash` 
   Guide to configure certificate on nxos:
   
   https://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/white-paper-c11-744191.html#CertificatesingNMI
   
- `gnmic client` must be installed on the linux host from which gnmic cli is invoked: https://gnmic.openconfig.net/install/
- `passwordless ssh` must be setup to ssh into linux host based on rsa public/private keypair: https://www.redhat.com/sysadmin/passwordless-ssh
- `gnmi_config_file` configuration file for gnmi must be present on linux host if gnmi_config_file option is slected in the role: https://gnmic.openconfig.net/user_guide/configuration_file/

       Sample config file for NXOS gnmic:
           ---
           username: <router username>
           password: <router password>
           port: 50051
           timeout: 5s
           skip-verify: true
           targets:
                   1.1.x.x:
                     timeout: 2s
                     tls-ca: ~/ansible_dir/nxos_ansible_playbooks/Ansible-NXOS/gnmi_configs/gnmi.pem

Role Variables
--------------
Please see vars/main.yaml for variable initialisation

| Variable                | Required | Default        | Choice                 |   Description                                                                |
|-------------------------|----------|----------------|------------------------|------------------------------------------------------------------------------|
| gnmi_target             | yes      |                |                        |  router hostname/ip to send grpc requests                                    |
| gnmi_port               | no       | 50051          |                        |  port on which grpc server listening on router                               |
| gnmi_user               | yes      |                |                        |  username on router                                                          |
| gnmi_pass               | yes      |                |                        |  password on router                                                          |
| skip_verify             | no       | true           |  true/false            |  skip certificate verification received from device                          |
| tls_ca_dir              | yes      |                |                        |  complete path to gnmi.pem file on linux host                                |
| gnmi_config_file        | no       | false          |                        |  complete path to gnmi config file to be used in gnmic cli                   |
| action                  | yes      |                |  [capabilities, get]   |  grpc action to be taken                                                     |
| path                    | no       | /System/name   |                        |  list of yang model xpath to fetch from router : /System/ch-items/model      |
| debug                   | no       | false          |  true/false            |  gnmic debug messages                                                        |
| network_os              | no       | nxos           |                        |  router network os such as nxos, catalyst, cumulus                           |
| gnmic_linux_host        | yes      |                |                        |  Linux host where gnmic client is installed and gnmic cli is executed        |

Role Output Variable
--------------------
- `gnmi_output` variable is set with the gnmic cli output from the device and becomes part of hostvars. It can be used to match substrings for validation. 

       Example usage:
       ---
       - hosts: N1
         #gather_facts: no
         serial: 1
         tasks:         
           - name: Verify capabilites on configured router from linux host using gnmic and use gnmi config file
             include_role: 
               name: gnmic_cli
             vars:
               action: capabilities
               gnmi_config_file: ~/ansible_dir/nxos_ansible_playbooks/Ansible-NXOS/gnmi_configs/nxos_gnmi_config.yaml
       
           - name: Output from gnmic capabilites fetch
             debug:
               var: gnmi_output.stdout
           
           - name: Verify fetched capabilities
             assert:
               that:
                 - "'Cisco Systems' in gnmi_output.stdout"
               fail_msg: "FAIL: Capabilites NOT received from {{ inventory_hostname}}"
               success_msg: "PASS: Capabilites received from {{ inventory_hostname}}"

       Sample output from the role:
          ok: [N1] => {
          "gnmi_output.stdout": "gNMI version: 0.8.0\nsupported models:\n  - Cisco-NX-OS-device, Cisco Systems, Inc., 2023-08-16\n  - DME, Cisco Systems, Inc., \n  - Cisco-NX-OS-Syslog-oper, Cisco Systems, Inc., 2019-08-15\nsupported encodings:\n  - JSON\n  - PROTO"
          }
       
          =====

          ok: [N1] => {
          "gnmi_output.stdout": [
              {
                  "source": "172.20.59.1:50051",
                  "time": "2023-12-12T13:47:30.379779413Z",
                  "timestamp": 1702388850379779413,
                  "updates": [
                      {
                          "Path": "System/name",
                          "values": {
                              "System/name": "N1"
                          }
                      }
                  ]
              },
              {
                  "source": "172.20.59.1:50051",
                  "time": "2023-12-12T13:47:30.507858471Z",
                  "timestamp": 1702388850507858471,
                  "updates": [
                      {
                          "Path": "System/ch-items/model",
                          "values": {
                              "System/ch-items/model": "N9K-C9300v"
                          }
                      }
                  ]
              },
              {
                  "source": "172.20.59.1:50051",
                  "time": "2023-12-12T13:47:30.511683485Z",
                  "timestamp": 1702388850511683485,
                  "updates": [
                      {
                          "Path": "System/showversion-items",
                          "values": {
                              "System/showversion-items": {
                                  "biosCompileTime": "",
                                  "biosVersion": "",
                                  "bootflashSize": 4358984,
                                  "copyRight": "Cisco Nexus Operating System (NX-OS) Software\nTAC support: http://www.cisco.com/tac\nDocuments: http://www.cisco.com/en/US/products/ps9372/tsd_products_support_series_home.html\nCopyright (c) 2002-2023, Cisco Systems, Inc. All rights reserved.\nThe copyrights to certain works contained herein are owned by\nother third parties and are used and distributed under license.\nSome parts of this software are covered under the GNU Public\nLicense. A copy of the license is available at\nhttp://www.gnu.org/licenses/gpl.html.\n\nNexus 9000v is a demo version of the Nexus Operating System\n",
                                  "kernelUptime": "10 days(s) 21 hour(s) 19 minute(s) 17 second(s) ",
                                  "lastResetEpochTime": "0",
                                  "lastResetReason": "Unknown",
                                  "lastResetService": "",
                                  "lastResetSysVersion": "",
                                  "lastResetTime": "",
                                  "nxosCompileTime": " 7/31/2023 12:00:00",
                                  "nxosImageFile": "bootflash:///nxos64-cs.10.4.1.F.bin",
                                  "nxosVersion": "10.4(1)",
                                  "plugin": "Core Plugin, Ethernet Plugin"
                              }
                          }
                      }
                  ]
              }
          ]
      }

 


    
Example Playbook
----------------
    ---
    - hosts: N1
      #gather_facts: no
      serial: 1
      tasks:         
        - name: Verify capabilites on configured router from linux host using gnmic and use gnmi config file
          include_role: 
            name: gnmic_cli
          vars:
            action: capabilities
            gnmi_config_file: ~/ansible_dir/nxos_ansible_playbooks/Ansible-NXOS/gnmi_configs/nxos_gnmi_config.yaml
    
        - name: Output from gnmic capabilites fetch
          debug:
            var: gnmi_output.stdout
    
        - name: Verify Metric fetched from configured router using gnmic and by passing parameters directly to gnmic cli
          include_role: 
            name: gnmic_cli
          vars:
            action: get
            
        - name: Output from gnmic capabilites fetch
          debug:
            var: gnmi_output.stdout