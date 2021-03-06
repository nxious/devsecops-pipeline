# Setting up Virtual Machines

## Objective

This section aims to accomplish the objectives listed as 1<sup>st</sup> and 2<sup>nd</sup> point of [`Task 2`](../problem-statement/#task-2) under the [Problem Statement](../problem-statement).

We will be discussing various steps taken to set up a local environment for running Jenkins and deploying the application in the following section.

## System Configuration

The project requires setting up two virtual machines using Ubuntu 18.04 LTS as the operating system. [VirtualBox](https://www.virtualbox.org/){target="_blank"} was installed and used to create the two virtual machines required for our setup.

The installation was performed using the following guide : [Ubuntu 18.04 Virtual Machine Setup](https://codebots.com/library/techies/ubuntu-18-04-virtual-machine-setup){target="_blank"}

The core count was increased to 2 and RAM was set to 8GB to provide sufficient resources to both machines.

The guide was elaborate and covered all the required steps including installation of the OpenSSH server for SSH access to the VM.

**Note: ** The network adapters were set to Bridged mode in order to allow independent IP assignment on my router itself. This allows me to access the servers via SSH and via a browser on the host machine and allows the Virtual Machines to act as real machines from a network point of view.

## Installing Jenkins

Jenkins is the leading open-source automation server. We will be using this tool to build our Continuos Integration (CI) pipeline for an application.

To install Jenkins on Ubuntu Server 18.04, I followed the official guide for Linux listed on Jenkins's website which can be found [here](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu){target="_blank"}.

![Jenkins Setup](images/Jenkins.png)

I proceeded with the `Install suggested plugins` option as guided by my mentors. 

**Note: ** Upon trying for the first time, some core plugins failed to install due to issues in the VM. I resolved all the errors by rebuilding the VM and switching to the weekly release instead of the LTS release.

After the setup was complete and the plugins finished installing, I was able to access the Jenkins dashboard:

![Jenkins Dashboard](images/Jenkins-dashboard.png)

## Choosing the application

The application that I have chosen for my pipeline is [DVNA (Damn Vulnerable Node Application)](https://github.com/appsecco/dvna){target="_blank"}. It is a NodeJS application designed to demonstrate OWASP Top 10 Vulnerabilities and comes with a guide on fixing and avoiding these vulnerabilities. 

## Configuring the application VM

To deploy the application (DVNA), I followed the guide included in the Github readme which can be found [here](https://github.com/appsecco/dvna#manual-setup){target="_blank"}. I used my personal fork of the application so that I can make changes to the application in the future.

The application requires a MySQL server for which I followed DigitalOcean's guide which can be found [here](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04){target="_blank"}.

For launching the application, MySQL configuration needs to be performed. The application's guide tells us to define the following system variables containing the database information:

```
export MYSQL_USER=dvna
export MYSQL_DATABASE=dvna
export MYSQL_PASSWORD=passw0rd
export MYSQL_HOST=127.0.0.1
export MYSQL_PORT=3306
```

If we use the export command, the variables will reset after every reboot unless defined in the .bashrc file. While automating this application, we will provide these variables through the `jenkinsfile`. To test the VM setup, I executed these commands individually.

Once all the configuration is completed, we can launch the app using `npm start` and access the application by visiting `http://VM-IP:9000` in a browser:

![DVNA Deployed!](images/DVNA.png)

**Note: **While running the application for the first time with `npm start`, I faced the error `Client does not support authentication protocol requested by server; consider upgrading MySQL client`. I found the solution for this on StackOverflow which can be located [here](https://stackoverflow.com/questions/50093144/mysql-8-0-client-does-not-support-authentication-protocol-requested-by-server){target="_blank"}. 

## Setting up SSH access

I setup SSH access to the application server from the `Jenkins` user on the Jenkins server. This was done in order to allow Jenkins to issue commands to the application server while deploying the application. This was done using the guide [here](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/){target="_blank"}.

First we need to login to our Jenkins server to generate SSH keys. We need to make sure we are generating the keys for jenkins user as it will be the one running the commands. As jenkins user doesn't have a password, we need to perform `su` using `sudo` :

```
sudo su jenkins
```

Once we are logged in as jenkins, we can proceed to generate the SSH keypair by using the command:

```
ssh-keygen -t rsa
```

This command will generate a public and private key for our user. The public key will be copied over to the `authorized_keys` folder on destination server in order to allow authentication without passing any key or password. 

We will be creating a `.ssh` directory which will contain the public key.

```
ssh common@192.168.1.5 mkdir -p .ssh
```

Here, `common` is the username of user we will be logging into and `192.168.1.5` is the IP of destination server to which we will be copying our application.

Now we will copy the public key of jenkins user to the remote server:

```
cat .ssh/id_rsa.pub | ssh common@192.168.1.5 'cat >> .ssh/authorized_keys'
```

Once the key is copied over to the destination server, we need to set permissions for the key:

```
ssh common@192.168.1.5 "chmod 700 .ssh; chmod 640 .ssh/authorized_keys"
```

In order to modify the permissions of the key, we need to enter the remote user's password once. 

Once the public key is set up on the application server, SSH is possible without providing any passwords or keys. This configuration automatically handles the key exchanges.