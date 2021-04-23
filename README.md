# Deploying the LAMP stack on IBM Cloud Virtual Servers for VPC with IBM Cloud Schematics

Deliver high-performance web apps and websites with the LAMP stack. LAMP is a bundle of open source software that you can use to create a solid and reliable foundation for your web app development. The following components are included in the LAMP stack:

* **Linux** as the operating system
* **Apache HTTP Server** as the webserver
* **MySQL** as the database
* **PHP** as the programming language

This playbook is designed to configure the LAMP stack on an [IBM Cloud Virtual Servers for VPC instance](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers) by using the built-in Ansible capabilities in Schematics.

​IBM Cloud Schematics provides powerful tools to automate your cloud infrastructure provisioning and management process, the configuration and operation of your cloud resources, and the deployment of your app workloads. To do so, Schematics uses open source projects, such as Terraform, Ansible, OpenShift, Operators, and Helm, and delivers these capabilities to you as a managed service. Rather than installing each open source project on your workstation, and learning the API or CLI. You declare the tasks that you want to run in IBM Cloud and watch Schematics run these tasks for you. For more information about Schematics, see [About IBM Cloud Schematics](https://cloud.ibm.com/docs/schematics?topic=schematics-about-schematics).
​
## About this playbook
​
To run this playbook, you must have a Virtual Private Cloud (VPC) and a Virtual Server instance (VSI) where you want to install the LAMP stack. When you run this playbook, Schematics securely connects to the target VSI by using SSH. To ensure that access to your target VSI is secured always, this playbook requires a bastion host to be configured within your VPC in addition to the target VSI. You can automate the setup of your VPC, your target VSI, and the bastion host by using a [Schematics Terraform template](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp).

The playbook in this repository was tested on IBM Cloud VPC Generation 2 VSIs that run CentOS 7.x. You can use this playbook on virtual servers that run either CentOS or RHEL. The playbook might not work with other `Linux` distributions.

When you run the playbook, the LAMP stack is configured on the target VSI. In addition, a web app is deployed to the stack that greets the user with a "Hello" message.

## Prerequisites
​
To run this playbook, complete the following tasks:
* Make sure that you have the required permissions to [create an IBM Cloud Schematics action](https://cloud.ibm.com/docs/schematics?topic=schematics-access).
* Make sure that you have the required permissions to [create and work with IBM Cloud VPC infrastructure components](https://cloud.ibm.com/docs/vpc?topic=vpc-iam-getting-started).
* [Create an upload an SSH key to the VPC dashboard](https://cloud.ibm.com/docs/vpc?topic=vpc-ssh-keys). This SSH key is used to access your bastion host and the VSIs in your VPC. Make sure that you upload the SSH key to the same region where you want to create your VSIs.
* [Provision a multitier VPC with a bastion host](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp) and a VSI that you can use to install the LAMP stack. To create this environment, you use the built-in Terraform capabilities in Schematics workspaces. For more information about Schematics workspaces, see [Creating workspaces](https://cloud.ibm.com/docs/schematics?topic=schematics-workspace-setup#create-workspace).
​
## Input variables
​
You must retrieve the following values to run the playbook in IBM Cloud Schematics.
​
|Input variable|Required / optional|Data type|Description|
|--|--|--|--|
|`upassword`|Required|String|Enter a password for your MySQL user, according to MySQL password policy. For example, `Abc@123abc`.|
|`dbname`|Optional|String|Enter the name that you want to use for your MySQL database. The default database name is `mysqldb`. |
|`dbuser`|Ootional|String|Enter a username that you want to set up for your MySQL database. The default user is `root`.|
|`mysql_port`|Optional|String|Enter the port number that your MySQL database listens on. The default port is 3306.|
|`httpd_port`|Optional|String|Enter the port number that your Apache HTTP Webserver listens on. The default port is 80.|
​
## Running the playbook in Schematics by using the UI
​
​1. Open the [Schematics action configuration page](https://cloud.ibm.com/schematics/actions/create?name=lamp-simple&url=https://github.com/Cloud-Schematics/ansible-lamp-simple).
2. Review the name for your action, and the resource group and region where you want to create the action. Then, click **Create**.
3. Select the `site.yml` playbook.
4. Select the **Verbosity** level to control the depth of information are shown when you run the playbook in Schematics.
5. Expand the **Advanced options**.
6. Enter all required input variables as key-value pairs. Then, click **Next**.
7. In the **IBM Cloud resource inventory** section, click the **Edit** button.
8. Enter the following details:
   - **Bastion host IP** Enter the public IP address of the Bastion host that you created.
   - **IBM Cloud inventory host groups** Enter the private IP dresses of all virtual servers where you want to configure the LAMP stack. For more information about how to define your host inventory, see the [Ansible documentation](https://docs.ansible.com/ansible/2.9_ja/plugins/inventory/ini.html).

     ```
     [webserver]
     172.16.0.4
     ```
     **Note**
     You can either create dynamic or Static inventory host group. Dynamic and static inventory creations are shown in the screen capture.
     ![Dynamic host group creation](/images/dyn_invgrp.png)

     ![Static host group with resource query creation](/images/static_inv.png)

   - **IBM Cloud resource inventory SSH key** Enter the private SSH key that you want to use to connect to your virtual servers. The private SSH key must match the public key that you added to the virtual server when you created it. If you stored the private key on your local workstation, you can run `cat ~/.ssh/id_rsa` to see the private key. **Note** You can run  `pbcopy < ~/.ssh/id_rsa` to copy entire private SSH key and paste.

9. Click **Check action** to verify your action details. The **Jobs** page opens automatically. You can view the results of this check by looking at the logs.
10. Click **Run action** to install the LAMP stack on your virtual server. You can monitor the progress of this action by reviewing the logs on the **Jobs** page.
​
## Running the playbook in Schematics by using the command prompt

1. Create a `hosts.ini` file on your local machine and add the private IP addresses of the virtual servers where you want to install the LAMP stack.

   ```
   [webserver]
   172.16.0.4
   ```

2. Retrieve the public IP address of the bastion host that you created.
3. Create the Schematics action. Enter all the input variable values that you retrieved earlier. When you run this command and are prompted to enter a GitHub token, enter the return key to skip this prompt.
   ```
   ibmcloud schematics action create --name lamp --location us-south --resource-group default --template https://github.com/Cloud-Schematics/lamp-simple --playbook-name site.yml --bastion <bastion_floating_IP> --target-file hosts.ini --credential ~/.ssh/id_rsa --input upassword=Abc@123abc --input dbuser=root --input dbname=mysqldb --input mysqlport=3306 --input httpd_port=<webserver port>
   ```

4. Verify that your Schematics action is created and note the ID that was assigned to your action.
   ```
   ibmcloud schematics action list
   ```

5. Create a job to run a check for your action. Replace `<action_ID>` with the action ID that you retrieved. In your CLI output, note the **ID** that was assigned to your job.
   ```
   ibmcloud schematics job run --command-object action --command-object-id <action_ID> --command-name ansible_playbook_check
   ```

   Example output
   ```
   ID                  us-south.JOB.lamp.fedd2fab
   Command Object      action
   Command Object ID   us-south.ACTION.lamp.1aa11a1a
   Command Name        ansible_playbook_check
   Name                JOB.lamp.ansible_playbook_check.2
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
​
Check the job logs for list of task such as `TASK Display Index page content` as shown in the screen capture.

![Job logs with the database details](/images/lamp_output.png)

## Delete an action
​
1. From the [Schematics actions dashboard](https://cloud.ibm.com/schematics/actions){: external}, find the action that you want to delete.
2. From the actions menu, click **Delete**.
​

## Reference
​
Review the following links to find more information about Schematics.
​
- [IBM Cloud Schematics documentation](https://cloud.ibm.com/docs/schematics)

​
## Getting help
​
For help and support with using this template in IBM Cloud Schematics, see [Getting help and support](https://cloud.ibm.com/docs/schematics?topic=schematics-schematics-help).
