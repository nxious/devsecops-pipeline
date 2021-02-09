# Migrating to AWS

## Objective

In this section, we will be migrating our local VM setup to the cloud on AWS.

## About AWS

Amazon Web Services (AWS) is one of the major cloud providers. They have worldwide servers with a comprehensive catalog of services. We will be using these services to run our Jenkins machine and the application server in the cloud.

## Instance setup

### Jenkins server

We will be using Elastic Compute 2 (EC2) service on AWS to deploy our Jenkins server. EC2 provides virtual servers in the cloud that can be configured according to our hardware and software requirements. 

We will be launching a `t2.medium` instance, which is equipped with 2 CPU cores and 4 GB RAM, which is sufficient for running Jenkins along with various tools.

We will be expanding our instance storage to 10GB as it is the minimum recommended amount and will allow us to store various applications, scripts and maintain logs and artifacts.

Once the instance is up and running we can proceed to setup Jenkins by following the steps in [Installing Jenkins](../setting-up-vms/#installing-jenkins) and creating our pipeline using the same steps as [Pipeline Setup](../pipeline-setup).

We will be re-installing our complete toolset for various stages (SCA,SAST etc.). All of the scripts created for the tools will be copied over from our local VM and uploaded to our instance via SFTP.

#### SCA Tools

##### npm-audit

npm-audit was installed using the same steps as the local installation.

##### retire.js

##### OWASP Dependency Check

##### audit.js

#### SAST Tools

##### njsscan

##### insider

##### snyk.io

##### Sonarqube Scanner

#### DAST Tools

#### Miscellaneous tools

##### ESLint

##### CycloneDX

### Application server

For the application server, we will follow the same steps as we followed for the Jenkins instance except for a few minor changes :

- We will launch a `t2.micro` instance with 1 CPU core and 1 GB of RAM, which should be sufficient for running a lightweight node application.

- In the network VPC, we will allow SSH access only from the IP of our Jenkins instance and open port 9090 to everyone as that is the application access point.

The rest of the configuration, such as the MySQL database and SSH access, will be done according to steps initially followed in [Configuring the application VM](../setting-up-vms/#configuring-the-application-vm).

###