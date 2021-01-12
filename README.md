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

## Running on IBM Cloud Schematics

IBM Cloud schematics supports running ansible playbooks natively. You can try the same example through various entry points. 

### Running by IBM Cloud Schematics API

1. Create action payload. Action payload is a json object. Start with `{}` and add required fields. 

    - Add basic information for action like `name`, `description`, `location` you want the action to be in, `resource_group` and tag for identification. Make sure`name` is unique.  

    ```
    "name": "Example-1",
    "description": "This Action install LAMP stack on VSI using ansible",
    "location": "us-east",
    "resource_group": "Default",
    "tags": [
      "string"
    ]
    ```

    - Add source for importing ansible playbooks. 
    ```
    "source_type": "GitHub", 
    "source": {
         "source_type" : "git",
         "git" : {
              "git_repo_url": "https://github.com/rvsingh011/lamp-simple"
         }
    }
    ```
    - Add playbook which should run by default when `run` is triggered. Any playbook from the source can be selected. 
    ```
    "command_parameter": "site.yml"
    ```

    - Add Credentials required o run the confrigation. All credntials should be added as `name` and `value` and can be reffered in target with the same name.
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
    - Add Bastion host information. Refer the credentials provided in above step in `cred_ref`.
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
