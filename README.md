# Introduction
This repository contains re-usable playbooks and templates to deploy OpenShift and related infrastructure using Ansible automation.

# Getting Started

## Installation Process
To work with this repository locally you will need clone it using https or ssh authentication.

For https:
```
git clone https://git.company.com/kubernetes/openshift-cluster-provisioning.git
```

For ssh:
```
git clone git@git.company.com:kubernetes/openshift-cluster-provisioning.git
```

## Assertions and Assumptions

This automation assumes the following conditions have been met.

vSphere:
- [ ] Service Account with proper permissions created
- [ ] Service Account login information stored in vault files
- [ ] Static DNS Records pre-created


# How to Install Prerequisites
### Software dependencies (for cluster deployment)

There are a handful of tools that you'll need to be able to effectively work with this repository. Please make sure you have the following things installed. 
Any commands provided are for linux systems.

- [Python 3+](https://www.python.org/downloads/) Installed
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) Installed
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) Installed

> NOTE: Ansible will need python 3 in order to use the required modules and libraries.  To install Ansible please contact the Unix team and have the Ansible repository added to your Linux machine.  Then use the following commands to install Ansible
>
> ```
> $ yum install ansible
> ```
> You can then confirm which python version is being used by running:
> ```
> $ ansible --version | grep "python version"
> ```
> Example output:
> ```
>   python version = 3.11.2 (main, Jun  6 2023, 07:39:01) [GCC 8.5.0 20210514 (Red Hat 8.5.0-18)] (/usr/bin/python3.11)
> ```

The utilites in the list below are also required, but can be installed automatically by executing the Ansible playbook listed below, once Ansible is installed.

- [OpenShift CLI](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html) Installed
- [OpenShift Installer CLI](https://cloud.redhat.com/openshift/install/aws/installer-provisioned) Installed
- [Python Modules](https://docs.python.org/3/installing/index.html):
  - [kubernetes](https://pypi.org/project/kubernetes/)
  - [openshift](https://pypi.org/project/openshift/)
  - [boto3](https://pypi.org/project/boto3/)
  
They can be automatically installed by running the following command: 

```
$ ansible-playbook playbooks/setup-prereqs.yaml --ask-become-pass
```

> **Notes:**
>
> *Arguments*
>
> `--ask-become-pass` This will prompt for your sudo password, so required python libraries can be installed.

You can force the OpenShift binaries to update (if you already have oc and openshift-install on your system), by using the following command:
```
$ ansible-playbook playbooks/setup-prereqs.yaml --ask-become-pass -e force_update=true
```

> **Notes:**
>
> *Arguments*
>
> `force_update` This will tell Ansible to delete the existing oc and openshift-install binaries, forcing new versions to be installed.

### Ansible Vault Password File
Ansible Vault is used to encrypt sensitive data in the repository, and in order to simplify automation for users, a
password file needs to be created. 

This can be done by running the following playbook:
```
$ ansible-playbook playbooks/create-passfile.yaml
```

You will be prompted to input the Ansible Vault password. You can get this password from a team member that already knows it.

> **DO NOT COMMIT THIS PASSWORD TO ANY REPOSITORY, EVER.**

# How to Deploy an OpenShift Cluster on vSphere
An OpenShift Container Platform cluster can be created using the following command: 

---

> **IMPORTANT**: Be sure to capture the install configs as described below, so you can use them to easily destroy the 
> hub cluster when needed.

---

```
$ ansible-playbook playbooks/deploy-cluster.yaml \
  -i <inventory file> \
  -e cluster_name=<cluster name> \
  -e '@./ocp_automation_vault.yaml' \
  -e env=<environment> \
  -e base_domain=<base domain> \
  -e install_type=<install type> \
  -e api_vip=<api vip> \
  -e ingress_vip=<ingress vip> \
  -e network=<network> \
  -e install_config.workers=yes \
  -vvv \
  --ask-vault-pass \
  --ask-become-pass

  
```
> **Notes:**
> 
> *Variables*
>
> `cluster_name` The desired name for the hub cluster.  
> `env` The desired environment for the cluster.  
> `base_domain` The root of the domain you are installing OpenShift into (e.g. dev.company.com).  
> `install_type` The type of install (online or offline).  
> `api_vip` The ipv4 wildcard address for the API.  
> `ingress_vip` The ipv4 address for the entry point to the cluster.  
> `network` The name of the network you are deploying to in vCenter.  
>
> *Ansible Vault*
>
> The vault password is stored in CyberArk in the KUB_AppSafe under the name `ocp_automation_vault`

#### Capture Install Configs for Cluster Destruction

After the hub installation is complete, we want to save the install config to a secure location so we can access it later,
when we are ready to cleanly destroy the hub cluster and remove all resources from vSphere.

When the cluster install is complete, copy the directory and all of its contents from `/tmp/install/<cluster-name>` to a secure
location.

> *Variables*
>
> `cluster_name` The name of the cluster

# How to Destroy a Cluster
You will need to use the original install config that you saved when the cluster was created in order to easily 
destroy the cluster and all related resources in vSphere.

To destroy the cluster, copy the contents of the install config directory you saved locally, and execute the following command:

```
$ openshift-install destroy cluster --dir=<path-to-install-dir> --log-level=info
```

# Repository Structure
This repository follows a simple Ansible repository structure. At the top level there are playbooks and roles. 
Playbooks are used to repeatably configure and deploy infrastructure. Roles are smaller modules of automation that can be called by playbooks. 

For more information on each directory, please take a look at the README in the respective directories.

# Contribute
To contribute, please follow these steps:

1. Clone repository locally.
2. Create branch for your code or configuration change.
3. Test your changes against the sandbox cluster (Please communicate with team if it will be disruptive).
4. Create a Pull Request to merge into main. Please make sure the PR is associated with your User Story or Task. This makes tracking far easier.
5. The Pull Request will be reviewed by 1 other engineer or architect, who will either approve it or leave feedback for enhancements.
6. The Pull Request will be merged once a peer has approved it.

