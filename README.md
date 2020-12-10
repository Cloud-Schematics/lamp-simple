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
To provision Multitier VPC Bastion Host on IBM cloud follow the steps [here](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)

## Prerequisites

 - Ansible 1.2.
 - [Multitier VPC Bastion Host](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)
 - Manager service access role for IBM Cloud Schematics
 - SSH Key on IBM Cloud

## Variables

| Variable Name | Description |	Default Value |
| ----- | ----- | ----- |
| vpc_name | Name of the VPC | |
| ssh_key_name| SSH key name to connect to VSI | |


## Running the playbook
 In hosts file update {target-host-ip}, {jump-server-ip} and respective ssh keys.
 The stack can be deployed using the following
- command:
    - ansible-playbook site.yml -e "upassword={password}  dbname={database-name}  dbuser={database-user}  mysql_port=3306  httpd_port=80" -i hosts


## Outputs

Successful output should be something like:
```
PLAY RECAP *********************************************************************************
ws01 : ok=22   changed=18   unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
```
Once done, you can check the results by doing "curl http://localhost/index.php" in that machine.
You should see a simple test page and a list of databases retrieved from the
database server.

- IBM Cloud Schematics
    - User can use IBM Cloud Schematics to run this playbook. For complete information on IBM CLoud schematics actions refer to the document here.

## References

- For MySQL installation we are using this [url](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)
