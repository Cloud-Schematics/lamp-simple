# LAMP Stack
-------------------------------------------
Use this role to deploy LAMP stack components on IBM cloud VSI by using Ansible or IBM Cloud Schematics. 

    Linux
    Apache
    MySQL (mariadb)
    PHP

These playbooks are meant to be a reference and starter's guide to building
Ansible Playbooks. These playbooks were tested on CentOS 7.x so we recommend
that you use CentOS or RHEL to test these modules. In this deployment we had 
used [simple web application]() which greets with an Hello msg.

This playbook is deployed/tested against IBM Cloud Multitier VPC Bastion Host. 
To provision Multitier VPC Bastion Host on IBM cloud follow the steps [here](https://github.com/akhiljain23/multitier-lamp-bastion-vpc)

## Prerequisites

 - Ansible 1.2.
 - [Multitier VPC Bastion Host](https://github.com/Cloud-Schematics/multitier-vpc-bastion-host)
 - Manager service access role for IBM Cloud Schematics
 - SSH Key on IBM Cloud

## Variables

| Variable Name | Description |	Default Value |
| ----- | ----- | ----- |
| vpc_name | Name of the VPC | |
| ssh_key_name| SSH key name to connect to VSI | |


## Running the playbook
 The stack can be deployed using the following
- command:
    - ansible-playbook -i hosts site.yml --extra_vars vpc_name=<VPC_NAME> --extra_vars ssh_key_name=<SSH_KEY_NAME>


## Outputs

Once done, you can check the results by browsing to http://localhost/index.php.
You should see a simple test page and a list of databases retrieved from the
database server.

- IBM Cloud Schematics
    - User can use IBM Cloud Schematics to run this playbook. For complete information on IBM CLoud schematics actions refer to the document [here]().

## Notes

- For MySQL installation we are using this [url](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)
