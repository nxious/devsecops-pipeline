# Migrating to AWS

## Objective

In this section, we will be migrating our local VM setup to the cloud on AWS.

## About AWS

Amazon Web Services (AWS) is one of the major cloud providers. They have worldwide servers with a comprehensive catalog of services. We will be using these services to run our Jenkins machine and the application server in the cloud.

## Instance setup

### Jenkins server

We will be using Elastic Compute 2 (EC2) service on AWS to deploy our Jenkins server. EC2 provides virtual servers in the cloud that can be configured according to our hardware and software requirements. 

To launch an instance, we will navigate to the EC2 Dashboard using the search bar in our AWS portal.

![EC2 Dashboard](images/ec2-dashboard.png)

We can proceed to launch a new instance by clicking the orange `Launch Instance` button in the EC2 dashboard. 

Since our VMs were running on Ubuntu Server 18.04, we will proceed with the same in AMI (Amazon Machine Images) selection.

![AMI Selection](images/ami-selection.png)

We will be launching a `t2.medium` instance, which is equipped with 2 CPU cores and 4 GB RAM, sufficient for running Jenkins and various tools.

![Instance Size Selection](images/size-selection.png)

We will continue with the default settings for Instance Details as there are no changes required here.

![Instance Details](images/instance-details.png)

The instance storage will be increased to 10GB as it is the minimum recommended amount and will allow us to store various applications, scripts and maintain logs and artifacts.

![Instance Storage](images/storage.png)

We will add the application type tag to our instance which will allow us to locate the specific instance easily.

![Instance Tags](images/instance-tags.png)

In security group configuration, we will be limiting access of the Jenkins dashboard, SonarQube dashboard and SSH to our IP address. The current IP address will be automatically picked up by using `My IP` option.

**Note:** Make sure you have a static IP address before setting specific IP access. Dynamic IP address would require you to change your IP address in the security group everytime your IP address changes.

![Security Group Configuration](images/security-group.png)

Now we can review the final configuration of our instance. Once we have an overview of the settings, we can deploy the instance using the `Launch` button.

![Instance Review](images/instance-review.png)

Upon launching the instance, we are asked to either create a new key pair or use and exisitng one. This key will be used to access the instance using SSH.

**Note:** While using an existing key, make sure you have access to the file as AWS allows downloading the file only once at the time of generation.

![Key](images/key.png)

Once the instance is up and running, the details such as IP address can be accessed from the EC2 dashboard by naigating to the `Running Instances` page. We can also manage and monitor our instance from here. 

Once we SSH to our instance, we can proceed to set up Jenkins by following the steps in [Installing Jenkins](../setting-up-vms/#installing-jenkins) and creating our pipeline using the same steps as [Pipeline Setup](../pipeline-setup).

**Note:** The default user for SSH is `ubuntu`.

We will be re-installing our complete toolset for various stages (SCA, SAST, etc.). All of the scripts created for the tools will be copied over from our local VM and uploaded to our instance via SFTP.

#### SCA Tools

##### npm-audit

npm audit was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation).

##### retire.js

retire.js was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_1).

##### OWASP Dependency Check

OWASP Dependency Check was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_2).

**Note:** While running the pipeline, Depenency Check was unable to locate the `yarn audit` command, which had to be installed manually using `npm install yarn -g`.

##### audit.js

audit.js was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_3).

#### SAST Tools

##### njsscan

retire.js was installed by following the same steps as mentioned in the [VM installation](../static-analysis/#installation).

##### insider

retire.js was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_1).

##### snyk.io

snyk.io was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_2). The API key was set as a secret in Jenkins.

##### Sonarqube Scanner

Sonarqube Scanner was installed by following the same steps as mentioned in the [VM installation](../software-composition-analysis/#installation_3). The token was set as a Jenkins secret.

The SonarQube server was set to launch at boot by adding the following command to `crontab -e`:

```
@reboot bash /var/lib/jenkins/scripts/sonarqube/bin/linux_x86/sonar.sh start
```

#### DAST Tools

The DAST phase consists of a single tool, OWASP ZAP. The tool was installed and configured using the same steps as mentioned in [installation steps](../dynamic-analysis/#installation). A seperate `freestyle project` was setup to run the tool.

#### ESLint (Code Linting)

ESLint was installed by following the same steps as mentioned in the [VM installation](../code-linting/#installtion). The initial config command was run in the root directory of DVNA.

#### CycloneDX (SBoM)

CycloneDX was installed by following the same steps as mentioned in the [VM installation](../software-bill-of-materials/#installation).

### Application server

For the application server, we will follow the same steps as we followed for the Jenkins instance except for a few minor changes :

- We will launch a `t2.micro` instance with 1 CPU core and 1 GB of RAM, which should be sufficient for running a lightweight node application.

- In the network VPC, we will allow SSH access only from the IP of our Jenkins instance and open port 9090 to everyone as that is the application access point.

The rest of the configuration, such as the MySQL database and SSH access, was done according to steps followed in [Configuring the application VM](../setting-up-vms/#configuring-the-application-vm).

