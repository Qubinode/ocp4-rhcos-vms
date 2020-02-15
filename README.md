ocp4-kvm-deployer
=================

This role will deploy OCP4 on a kvm host.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

Additional Details About The Role
----------------------------------
The role has three main tasks file:

* pre_tasks.yml
* tear_down.yml
* deploy.yml

### **TASK: pre_tasks.yml**

- Sets up global variables for load balancer contaier
- Ensure the jq command is installed

### **TASK: tear_down.yml**

The tasks is responsible for tearing down the OCP cluster. It will remove all DNS entries, remove the VMs, remove the containers and any other files created by the role.

Example Usage:

```sh
# Teardown the entire cluster
ansible-playbook rhcos.yml -e "tear_down=true"

# Remove just the webserver container
ansible-playbook rhcos.yml -e "tear_down=true" -t webserver

```

Dependancy roles:
  - openshift-4-loadbalancer
  - ocp4-bootstrap-webserver

### **TASK: deploy.yml**

1. check for ocp4 pull secret and fail if it's not found
2. ensure the kvm host firewall ports are open
3. install_podman.yml: ensure the kvm host is setup to run podman containers
4. ocp4_tools.yml: ensure the ocp4 tools such as the oc command and the openshift installer are installed
5. create_ignitions.yml: generate ignitions files based on the number of masters and workers specified for this cluster
6. deploy the container load balancer
7. deploy the container webserver that servers up the files for bootstrap 
8. download_rhcos_files.yml: download the files required to bootstrap rhcos
9. create directory to store the generated virt-install scripts for rhcos
10. deploy a generated treeinfo that defines the location of the kernel and ramdisks
11. deploy_libvirt_network.yml: create libvirt nat network for rhcos vms
12. configure_dns_entries.yml: populate Red Hat IdM with the dns entries for OCP4
13. build_cluster_nodes_profile.yml: generate the virt-install scripts for building the rhcos VMs
14. build_vm_list.yml: create the variable *ocp4_nodes* with a list of all the ocp4 required for the cluster
15. deploy the ocp4 rhcos node vms
16. wait_for_vm_shutdown.yml: OCP nodes shutdown after the initial boostrap, this tasks waits for all the VMs to shutdown
17. start up the RHCOS vms to continue the OCP deployment
18. run /usr/local/bin/openshift-install wait-for bootstrap-complete
19. contine install if bootstrap returns `It is now safe to remove the bootstrap resources`
20. get all running VMS
21. shutdown the bootstrap node
22. wait_for_vm_shutdown.yml: wait for the bootstrap node to shutdown
23. destroy the bootstrap node

pull_secret_request: https://cloud.redhat.com/openshift/install/metal/user-provisioned

Example Usage:

```
ansible-playbook rhcos.yml -t setup
ansible-playbook rhcos.yml -t tools
ansible-playbook rhcos.yml -t podman
ansible-playbook rhcos.yml -t podman --skip-tags pkg
ansible-playbook rhcos.yml -t ignitions
ansible-playbook rhcos.yml -t lb
ansible-playbook rhcos.yml -t webserver
ansible-playbook rhcos.yml -t download
ansible-playbook rhcos.yml -t libvirt_net
ansible-playbook rhcos.yml -t node_profile
ansible-playbook rhcos.yml -t idm
ansible-playbook rhcos.yml -t rhcos

ansible-playbook rhcos.yml -t webserver
ansible-playbook rhcos.yml -t lb
ansible-playbook rhcos.yml -t libvirt_net
ansible-playbook rhcos.yml -t deploy_vms
```
Dependancy roles:
  - openshift-4-loadbalancer

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
