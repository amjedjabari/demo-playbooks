# AWS OCP Hosted Control Planes (HCP) Automation
This directory contains playbooks for setting up HCP clusters in a Red Hat AWS Open Environment on demo.redhat.com

## Prerequisites
### Software dependencies

There are a handful of tools that you'll need to be able to effectively work with this repository. Please make sure you have the following tools installed.
Any commands provided are for linux systems.

- [Python 3+](https://www.python.org/downloads/) Installed
- [pip and setuptools](https://packaging.python.org/en/latest/tutorials/installing-packages/#ensure-pip-setuptools-and-wheel-are-up-to-date) Installed
- [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) Installed
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) Installed

> NOTE: Ansible will need python 3 in order to use the required modules and libraries, so it is recommended that you use
> the following command to install Ansible:
>
> ```
> $ pip3 install ansible
> ```
> You can then confirm which python version is being used by running:
> ```
> $ ansible --version | grep "python version"
> ```
> Example output:
> ```
> python version = 3.6.8 (default, Mar 18 2021, 08:58:41) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]
> ```

The utilities in the list below are also required, but can be installed automatically by executing the Ansible playbook listed below, once Ansible is installed.

- [OpenShift CLI](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html) Installed
- [Python Modules](https://docs.python.org/3/installing/index.html):
    - [setuptools](https://pypi.org/project/setuptools/)
    - [kubernetes](https://pypi.org/project/kubernetes/)
    - [openshift](https://pypi.org/project/openshift/)
    - [boto3](https://pypi.org/project/boto3/)

## Running the Automation
### Other Required Repo
This repo references roles from another repo, and so you must have both repos checked out in the same top level directory 
in order for the relative pathing to work.

You can checkout this repo and the required repo in your desired directory using the following commands:

```
git clone https://github.com/SkylarScaling/demo-playbooks.git
git clone https://github.com/SkylarScaling/61502-automation.git
```

(An example inventory file can be found in the inventory directory.)

### Steps for Execution
1) Provision "AWS Blank Open Environment"
2) Run `aws configure` on your machine and input the AWS account values provided by the demo environment
3) Login to the aws account for your demo environment and go to `Red Hat OpenShift Service on AWS (ROSA)`
4) Click `Get Started` button
5) Click `Enable ROSA with HCP` in the ROSA enablement box
6) When complete, you will be given a redirect link to your Red Hat Hybrid Cloud console to link your account
7) After your account is linked, you will receive a ROSA login command (https://console.redhat.com/openshift/create/rosa/getstarted)
8) Copy and execute the `rosa login` command on your machine
9) Back in your AWS account, go to `VPCs`
10) Click `Create VPC`
11) Select `VPC and more` from the VPC settings under "Resources to create"
12) Change the VPC Auto-generate field under Name tag auto-generation from "Project" to `your cluster name`
13) Change the IPv4 CIDR block to `10.0.0.0/24`
14) Set Number of Availability Zones (AZs) to `3`
15) Change NAT gateways ($) to `In 1 AZ`
16) Click `Create VPC`
17) Once the VPC is finished creating, go to VPC -> `Subnets`
18) Copy the 3 private and 3 public subnet IDs into the inventory file
19) Ensure all other variables in the inventory file are properly populated
20) Provision the HCP cluster and apply Day 2 configs by running the following command:
```
ansible-playbook install-infra.yaml -i <inventory filepath>
```

If for whatever reason, the playbook fails after cluster provisioning, but before the Day 2 config completes, you can resolve
the errors for Day 2 and re-run JUST the Day 2 config playbook by doing the following:
1) Populate the `ocp_cluster_address` variable in your inventory (you won't have this until the cluster provisions, and install-infra automatically populates this)
2) Re-run the Day 2 config by running the following command:
```
ansible-playbook configure-day2-full.yaml -i <inventory filepath>
```

If you only want to provision the HCP cluster, without applying the Day 2 configuration, run the following command:
```
ansible-playbook provision-demo-hcp-cluster.yaml -i <inventory filepath>
```