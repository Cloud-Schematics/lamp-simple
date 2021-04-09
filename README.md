# LAMP Stack

LAMP stands for Linux, Apache, MySQL, and PHP. Together, they provide a proven set of software for delivering high-performance web applications.

IBM Cloud Schematics provides powerful tools to automate your cloud infrastructure provisioning and management process, the configuration and operation of your cloud resources, and the deployment of your app workloads.  To do so, Schematics leverages open source projects, such as Terraform, Ansible, OpenShift, Operators, and Helm, and delivers these capabilities to you as a managed service. Rather than installing each open source project on your machine, and learning the API or CLI, you declare the tasks that you want to run in IBM Cloud and watch Schematics run these tasks for you. For more information about Schematics, see [About IBM Cloud Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-about-schematics).

These playbooks are intended to be a reference and starter's guide to building Ansible Playbooks for use with IBM Cloud and Schematics. These playbooks were tested on CentOS 7.x so we recommend that you use CentOS or RHEL to test these modules. 


## About this playbook

This playbook is designed to to deploy the LAMP stack components on an IBM cloud VSI by running RedHat Ansible locally or by using IBM Cloud Schematics Actions.

    Linux
    Apache
    MySQL (mariadb)
    PHP
In this deployment we deploy a [simple web application]() which greets the user with a "Hello" msg.
This playbook has been run and tested using VSIs in a VPC Gen2 environment, deployed using the IBM Cloud Multitier VPC Bastion LAMP example. 

## Prerequisites
    
To run this playbook, complete the following tasks:
- Make sure that you have the required permissions to [create an IBM Cloud Schematics action](https://cloud.ibm.com/docs/schematics?topic=schematics-access).
- Make sure that you have the required permissions to provision Multitier VPC Bastion LAMP on IBM cloud
For example click [here](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)
- SSH Key on IBM Cloud

## Input variables

|Input variable|Required/ optional|Data type|Description|
|--|--|--|--|
|`mysql_port`|Required|Number|MySQL port number|
|`httpd_port`|Required|Number|Httpd port number|
|`dbuser`|Required|String|User name for the mysql database|
|`upassword`|Required|String|Password for the mysql database|

## Running the playbook in Schematics by using UI

1. Open the [Schematics action configuration page](https://cloud.ibm.com/schematics/actions/create?name=lampsimple&url=https://github.com/Cloud-Schematics/lamp-simple).
2. Review the name for your action, and the resource group and region where you want to create the action. Then, click **Create**.
3. Select the `site.yml` playbook.
4. Select the **Verbosity** level to control the depth of information that will be shown when you run the playbook in Schematics.
5. Expand the **Advanced options**.
6. Enter all required input variables as key-value pairs. Then, click **Save**.
7. Enter the bastion host ip, inventory and SSH key. Then, click **Save**. 
8. Click **Check action** to verify your action details. The **Jobs** page opens automatically. You can view the results of this check by looking at the logs.
9. Click **Run action** to deploy the LampStack. You can monitor the progress of this action by reviewing the logs on the **Jobs** page.

## Running the playbook in Schematics by using the command line

1. Create the Schematics action. Enter all the input variable values that you retrieved earlier. When you run this command and are prompted to enter a GitHub token, enter the return key to skip this prompt.
   ```
   ibmcloud schematics action create --name lampstack --location us-south --resource-group default --template https://github.com/Cloud-Schematics/lamp-simple --playbook-name site.yml --input "mysql_port": "<mysql_port>" --input "httpd_port": "<httpd_port>" --input "dbuser": "<dbuser>" --input "upassword": "<db_password>"
   ```

   Example output:
   ```
   Enter github-token>
   The given --inputs option region: is not correctly specified. Must be a variable name and value separated by an equals sign, like --inputs key=value.

   ID               us-south.ACTION.lampstack.1aa11a1a
   Name             lampstack
   Description
   Resource Group   default
   user State       live

   OK
   ```

2. Verify that your Schematics action is created and note the ID that was assigned to your action.
   ```
   ibmcloud schematics action list
   ```

3. Create a job to run a check for your action. Replace `<action_ID>` with the action ID that you retrieved. In your CLI output, note the **ID** that was assigned to your job.
   ```
   ibmcloud schematics job run --command-object action --command-object-id <action_ID> --command-name ansible_playbook_check
   ```

   Example output:
   ```
   ID                  us-south.JOB.lampstack.fedd2fab
   Command Object      action
   Command Object ID   us-south.ACTION.lampstack.1aa11a1a
   Command Name        ansible_playbook_check
   Name                JOB.lampstack.ansible_playbook_check.2
   Resource Group      a1a12aaad12b123bbd1d12ab1a123ca1
   ```

4. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>
   ```

5. Create another job to run the action. Replace `<action_ID>` with your action ID.
   ```
   ibmcloud schematics job run --command-object action --command-object-id <action_ID> --command-name ansible_playbook_run
   ```

6. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>
   ```


## Verification

Check the job logs for of TASK: `Display Index page content`. The content should give List of Databases and the host name if everything completed successfully.

```
ASK [Display Index page content] *****************************************************
 2021/01/12 10:01:18 ansible-playbook run | ok: [dbhost0] => {
 2021/01/12 10:01:18 ansible-playbook run |     "output.content": "<html>\n <head>\n  <title>Ansible Application</title>\n </head>\n <body>\n </br>\n  <a href=http://172.22.192.6/index.html>Homepage</a>\n </br>\n </br>\n  <a href=http://172.22.192.6/about.html>About Us</a>\n </br>\nHello, World! I am a web server configured using Ansible and I am : vsi2</BR>List of Databases: </BR>foodb\nhelp\ninformation_schema\nmysql\nperformance_schema\nsys\n</body>\n</html>\n\n"
 2021/01/12 10:01:18 ansible-playbook run | }
```

## Reference

Review the following links to find more information about Schematics and Lampstack

- [IBM Cloud Schematics documentation](https://cloud.ibm.com/docs/schematics)
- [LampStack](https://www.ibm.com/cloud/learn/lamp-stack-explained)

## Getting help

For help and support with using this template in IBM Cloud Schematics, see [Getting help and support](https://cloud.ibm.com/docs/schematics?topic=schematics-schematics-help).
