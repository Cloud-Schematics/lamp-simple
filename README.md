# LAMP Stack
-------------------------------------------
Use this role to deploy the LAMP stack components on an IBM cloud VSI by running RedHat Ansible locally or by using IBM Cloud Schematics Actions. 

    Linux
    Apache
    MySQL (mariadb)
    PHP

These playbooks are intended to be a reference and starter's guide to building Ansible Playbooks for use with IBM Cloud and Schematics. These playbooks were tested on CentOS 7.x so we recommend that you use CentOS or RHEL to test these modules. In this deployment we deploy a [simple web application]() which greets the user with a "Hello" msg.

Schematics Actions uses SSH to configure target VSIs on IBM Cloud. To ensure that all access to the target VSIs is secured it is assumed that SSH access is configured via a bastion host/jump server om IBM Cloud. This [IBM Cloud Automation](https://github.com/Cloud-Schematics) repo contains a number of example Terraform configs that deploy a VPC Gen 2 environment with bastion host access.   

This playbook has been run and tested using VSIs in a VPC Gen2 environment, deployed using the IBM Cloud Multitier VPC Bastion LAMP example. 
To provision Multitier VPC Bastion LAMP on IBM cloud follow the steps [here](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)

## Prerequisites

 - Ansible 1.2.9
 - [Multitier VPC Bastion LAMP](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)
 - Manager service access role for IBM Cloud Schematics
 - SSH Key on IBM Cloud

## Variables

| Variable Name | Description |	Default Value |
| ----- | ----- | ----- |
| vpc_name | Name of the VPC | |
| ssh_key_name| SSH key name to connect to VSI | |


## Running the playbook locally
 To run this example locally, create a hosts file in the root of the example folder and update it with the {target-host-ip}, {jump-server-ip} and respective ssh keys. Details of how to configure the host file manually can be found in the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups). 
 The stack can be deployed using the following 
- command:
    - ansible-playbook site.yml -e "upassword={password}  dbname={database-name}  dbuser={database-user}  mysql_port=3306  httpd_port=80" -i hosts

## Running on IBM Cloud Schematics

IBM Cloud Schematics supports running Ansible playbooks natively. You can try the same example through various entry points. 

### Running Actions using the IBM Cloud Schematics API

1. Create a file containing the Schematics Action payload. The Action payload is a json object. Start with `{}` and add the required fields. 

    - Add basic information for action like `name`, `description`, the Schematics `location` you want the Action to be placed in, `resource_group` and tag for identification. Ensure that the Action `name` is unique.  

    ```
    "name": "Example-1",
    "description": "This Action install LAMP stack on VSI using ansible",
    "location": "us-east",
    "resource_group": "Default",
    "tags": [
      "string"
    ]
    ```

    - Add the source repo for importing the Ansible playbooks. 
    ```
    "source_type": "GitHub", 
    "source": {
         "source_type" : "git",
         "git" : {
              "git_repo_url": "https://github.com/Cloud-Schematics/lamp-simple"
         }
    }
    ```
    - Add the playbook which should run by default when `run` is triggered. Any playbook from the source repo can be selected. 
    ```
    "command_parameter": "site.yml"
    ```

    - Add the Credentials required o run the configuration. For deploying code to VSI's these will be the SSH private keys for the bastion host and target VSIs. All credentials should be added as `name` and `value` and will be referred in the target section with the same name.
    ```
    "credentials": [
      {
        "name": "ssh_key",
        "value": "< SSH_KEY >",
        "metadata": {
          "type": "string",
          "default_value": "",
          "secure": true
        }
      }
    ]
    ```
    - Add Bastion host information. Refer to the credentials provided in the above step in `cred_ref`.
    ```
    "bastion_ref": {
      "name": "bastionhost",
      "type": "string",
      "description": "string",
      "resource_query": "< BASITION_HOST_IP_ADDRESS >",
      "credential_ref": "ssh_key"
    }
    ```
    - Add inventory information. Refer the credentials provided in above step in `cred_ref`. Multiple groups with multiple host can be provided.The following inventory file 
    ```
    [webserverhost]
    FIRST_WEB_SERVER_IP_ADDRESS
    SECOND_WEB_SERVER_IP_ADDRESS

    [dbhost]
    FIRST_DATABASE_SERVER_IP_ADDRESS
    SECOND_DATABASE_SERVER_IP_ADDRESS
    ```
    can be represented as 
    ```
    "targets_ref": [
      {
        "name": "webserverhost",
        "description": "Group of webservers",
        "credential_ref": "ssh_key",
        "bastion_ref": "bastionhost",
        "target_resources": [
          {
            "resource_id": "< FIRST_WEB_SERVER_IP_ADDRESS >"
          },
           {
            "resource_id": "< SECOND_WEB_SERVER_IP_ADDRESS >"
          }
        ]
      },
      {
        "name": "dbhost",
        "description": "Group of database servers",
        "credential_ref": "ssh_key",
        "bastion_ref": "bastionhost",
        "target_resources": [
          {
            "resource_id": "< FIRST_DATABASE_SERVER_IP_ADDRESS >"
          },
           {
            "resource_id": "< SECOND_DATABASE_SERVER_IP_ADDRESS >"
          }
        ]
      }
    ]
    ```
    - Add inputs variables required by the playbook. Keys can be marked `secure`.
    ```
    "inputs": [
      {
        "name": "upassword",
        "value": "Abc@123abc",
        "metadata": {
          "type": "string",
          "secure": true,
          "default_value": "Abc@123abc"
        }
      },
      {
        "name": "dbname",
        "value": "foodb",
        "metadata": {
          "type": "string",
          "default_value": "foodb"
        }
      },
      {
        "name": "dbuser",
        "value": "root",
        "metadata": {
          "type": "string",
          "default_value": "root"
        }
      },
      {
        "name": "mysql_port",
        "value": "3306",
        "metadata": {
          "type": "string",
          "default_value": "3306"
        }
      },
      {
        "name": "httpd_port",
        "value": "80",
        "metadata": {
          "type": "string",
          "secure": false,
          "default_value": "80"
        }
      }
    ]
    ```

2. Create action by making a http request.
    - Headers: 
    Authorization : < Bearer ...>
    - Method: POST
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/actions`


3. Note the `ID` from step 1 and check the status of action by making http request. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/actions/<ID>`

4. Verify if the action is in `normal` state. 
    ```
    "state": {  
        "status_code": "normal",
        "status_message": "Action is normal and ready for execution"
    }
    ```
5. Create Job with a http request. Modify the payload with the `ID` received in step 1. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/jobs`

    ```
    {
        "command_object": "action",
        "command_object_id": "< ACTION_ID >",
        "command_name": "ansible_playbook_run"
    }
    ```

6. Check logs with a http request. Use the `ID` from step 4. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/jobs/<JOB-ID>/logs`

## Outputs

Check the job logs for of TASK: `Display Index page content`. The content should give List of Databases and the host name if everything completed successfully.

```
ASK [Display Index page content] *****************************************************
 2021/01/12 10:01:18 ansible-playbook run | ok: [dbhost0] => {
 2021/01/12 10:01:18 ansible-playbook run |     "output.content": "<html>\n <head>\n  <title>Ansible Application</title>\n </head>\n <body>\n </br>\n  <a href=http://172.22.192.6/index.html>Homepage</a>\n </br>\n </br>\n  <a href=http://172.22.192.6/about.html>About Us</a>\n </br>\nHello, World! I am a web server configured using Ansible and I am : vsi2</BR>List of Databases: </BR>foodb\nhelp\ninformation_schema\nmysql\nperformance_schema\nsys\n</body>\n</html>\n\n"
 2021/01/12 10:01:18 ansible-playbook run | }
```
## References

- For MySQL installation we are using this [url](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)
