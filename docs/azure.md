# Installation of OpenShift on Microsoft Azure

* Requirements
* Troubleshooting

## Requirements

In order to successfully create an OpenShift installation on Azure, be sure to have
* **Azure Account**: A valid account for your to setup your credentials. 
* **Azure Subscription**: This allows you to create any virtual machines on demand.
* **Red Hat Subscription**: OpenShift requires a valid subscription to be able to install packages from Red Hat's repository.


Besides that, it's important to have available
* Ansible (latest: version 2.6.3)
* Ansible's Module for Azure 
```bash
$ pip install ansible[azure]
```

## Setup file: openshift.config

This file contains all the necessary information you might need to have a basic cluster. Here is a example of the contents of this file:
```yaml
cloud_provider: azure

name: openshift
region: eastus
labels:
   host: openshift

admin_username: demo
admin_ssh_publickey: /home/demo/.ssh/id_rsa.pub

openshift_users: { 'demo': '$apr1$ucdyfv6b$EJFanTIfig6LR8cszOnKV0' }

redhat_subscription_username: my_redhat_subscription_username
redhat_subscription_password: my_redhat_subscription_password

azure_subscription_id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
azure_tenant: yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy
azure_client_id: zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz
azure_secret: wwwwwwww-wwww-wwww-wwww-wwwwwwwwwwww

number_of_masters: 1
number_of_infras: 1
number_of_nodes: 1
```

## Step #1: Networking 

Now, all you need to do, it's to run the Ansible's playbooks that it will create everything automatically. The first step will create all the necessary networking capabilities on Azure:

```bash
$  ./step1_create_network.yaml
```

## Step #2: Storage Account 

Once the networking is ready, you may proceed with the creation of a storage account and added in roles/ocp_bastion_preparation/templates/ansible_hosts.j2

```bash
$ az storage account create --name openshiftregistry1 --resource-group openshift --location eastus --sku Standard_LRS
$ az storage account keys list --account-name openshiftregistry1 --resource-group openshift --output table
$ openshift_hosted_registry_storage_azure_blob_accountkey='myKey1'
```

## Step #4: Computing 

Once the storage account is ready, you may proceed with the creation of all the necessary computing capabilities in order to deploy the whole cluster

```bash
$ ./step2_create_computing.yaml
```

## Step #5: OpenShift Installation

The last step, will start the installation of OpenShift 

```bash
$ ./step3_install_openshift.sh
```
## Deleting All

After you've done using your OpenShift Container Platform, you can get rid of everything by running

```bash
$ ansible-playbook delete_all.yaml
```

This will delete all the instances, storage and networking that was associated with your cluster
