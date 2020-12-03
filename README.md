# LAMP Stack
-------------------------------------------
Use this role to deploy LAMP stack components on IBM cloud VSI by using Ansible or IBM Cloud Schematics. 

    Linux
    Apache
    MySQL (mariadb)
    PHP

These playbooks are meant to be a reference and starter's guide to building
Ansible Playbooks. These playbooks were tested on CentOS 6.x so we recommend
that you use CentOS or RHEL to test these modules.

This playbook is deployed/tested against IBM Cloud Multitier VPC Bastion Host. 
To provision Multitier VPC Bastion Host on IBM cloud follow the steps [here](https://github.com/akhiljain23/multitier-lamp-bastion-vpc)

## PreRequisites

 - Ansible 1.2.
 - [Multitier VPC Bastion Host](https://github.com/Cloud-Schematics/multitier-vpc-bastion-host)
 - Manager service access role for IBM Cloud Schematics

## Variables

| Variable Name | Description |	Default Value |
| ----- | ----- | ----- |
| vpc_name | Name of the VPC | |
| ssh_key_name| SSH key name to connect to VSI | |


## Running the playbook
 The stack can be deployed using the following
- command:
    - ansible-playbook -i hosts site.yml

- IBM Cloud Schematics
    - Create IBM Cloud Schematics actions with <Name>, <Location>, <Rsource_group> and <description>
    - Update the action with the repo URL pointing to this repository
    - Update the inventory with bastion host, target host and credentials
    - Now the action is ready for execution. Trigger the execution by running a job
    - Track the progress of job either by querying for logs or tracking the status of Job
    - Once the Job is finished is successfully, the lamp stack is successfully deployed on IBM Cloud Schematics
For complete information on IBM Cloud Schematics action refer here.


## Outputs

Once done, you can check the results by browsing to http://localhost/index.php.
You should see a simple test page and a list of databases retrieved from the
database server.
