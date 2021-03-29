# Pipeline limitations and assumptions

## Objective

In this section, we will test our pipeline for a different application as per [`Task 4`](../problem-statement/#task-4) listed under the [Problem Statement](../problem-statement).

We will discuss the limitations and assumptions of the pipeline that we have built for DVNA by testing a new application with a different tech stack as compared to the previous one.

## Setting up the application

The application which I have chosen to test the limitations and assumptions made for the existing pipeline is [SuiteCRM](https://suitecrm.com){target="_blank"}. SuiteCRM is a PHP based, free and open source Customer Relationship Management application. This application will allow us to test our pipeline's limitations as it is an actual production application instead of a sample app like DVNA, hence it has a bigger codebase and also uses a completely different tech stack as compared to DVNA.

I proceeded by making a personal fork of the master branch of SuiteCRM on Github. This will allow me to modify and test the application according to my requirements.

### Installation

Installation of the application was carried out by following the [official guide](https://docs.suitecrm.com/admin/installation-guide/downloading-installing/){target="_blank"}. The requirements are an appropriate version of PHP, a web server, and a database. 

I installed the latest version of php by using the following command:

```
sudo apt install php
```

**Note: **The above installation of PHP does not install the modules required by SuiteCRM. The required modules were installed later by following [this guide](https://websiteforstudents.com/install-suitecrm-on-ubuntu-16-04-18-04-with-apache2-mariadb-and-php-7-2/){target="_blank"}.

For the webserver, I chose to use Apache 2 as I have worked with it previously and it is one of the most used webserver. An alternative to Apache is Nginx which is also a very popular webserver.

To install Apache 2 on the instance, I used the following command:

```
sudo apt install apache2
```

**Note: **Apache runs on port 80 so please make sure the port is opened in the security group.

For the database, I already had a running RDS instance on AWS. I reused the same instance and created a new database named `suitecrm` for the application. The endpoint was passed as the server URL along with the same credentials used previously.

SuiteCRM uses [Composer](https://getcomposer.org){target="_blank"} as a package manager, which is similar to npm for DVNA. We will install composer to get the dependencies as well as some tools.

Composer can be installed by following the official guide [here](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos){target="_blank"}. It comes as a `.phar` file which is a PHP executable. 

The file is made executable by using the command `chmod +x composer.phar` and is placed inside `/usr/local/bin/` which allows direct access to the executable from anywhere in the system.

**Note: **Composer v2 is not compatible with a specific plugin for SuiteCRM named Wikimedia Composer Merge, this is a known issue hence I downgraded Composer to 1.x channel to continue with the setup.

The files of SuiteCRM need to be placed inside `/var/www/html` which is the default root directory for the Apache webserver. Once the files are copied in the directory we need to run `composer install` in order to install the dependencies. Once composer has finished running, we can access the application installer by visiting `http://server_ip`. If everything is setup correctly we should see the SuiteCRM installer.

The installer will allow us to install the application using the details provided. It primarily requires the database information, admin user details and checks for correct permissions along with required modules.

**Note: **The installer was unable to access the modules and uploads directory due to insufficient permissions. This was fixed using [this method](https://docs.suitecrm.com/admin/installation-guide/downloading-installing/#_copying_suitecrm_files_to_web_server){target="_blank"} listed in the [official documentation](https://docs.suitecrm.com/admin/installation-guide/downloading-installing/){target="_blank"}.

**Note: ** The installer suggested a change in the max_upload_size parameter in php.ini which was increased to 10MB from 6MB.

Once the installer has finished, we can see the login page:

![SuiteCRM Login Page](../images/suitecrm-login.png)

Since SuiteCRM is written in PHP, most of the tools from the previous pipeline were rendered useless. The guides located [here](https://samate.nist.gov/index.php/Source_Code_Security_Analyzers.html){target="_blank"} and [here](https://owasp.org/www-community/Source_Code_Analysis_Tools){target="_blank"} helped in finding new tools for our application.

### Software Composition Analysis (SCA)

The tools used for SCA phase were different as compared to DVNA since SuiteCRM is written in PHP. One tool which was retained is the OWASP Dependency Check as it supports multiple languages.

#### [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/){target="_blank"}

##### Installation

The installation steps for ODC were same as the ones followed for DVNA mentioned [here](../software-composition-analysis/#installation_2). Since the tool installation is not application specific, we can use the same Dependency Check installation to test SuiteCRM.

##### Usage

Dependency Check was used in the same manner as used for DVNA, the tool definition was mentioned in the jenkinsfile and added as a stage:

```
stage ('Performing OWASP Dependency Check') {
    steps {
        dependencyCheck additionalArguments: '--format JSON dependency-check-report.json', odcInstallation: 'DVNA'
        sh 'mv dependency-check-report.json ~/reports/suitecrm/'
    }
}
```

#### [Local PHP Security Checker](https://github.com/fabpot/local-php-security-checker){target="_blank"}

Local PHP Security Checker is an open-source command line tool that checks if your PHP application depends on PHP packages with known security vulnerabilities. It uses the Security Advisories Database behind the scenes. 

##### Installation

Local PHP Security Checker is available as an executable for Linux which can be downloaded from the releases section of the tool's github page.

##### Usage

Since Local PHP Security Checker comes as an executable, a script was created to run the tool and placed in the `scipts/suitecrm/` directory located in the home directory of jenkins user. 

The script ran the tool placed in the tools directory with the flags required for getting output according to our requirements:

```
#!/bin/bash
cd ~/scripts/suitecrm/
./local-php-security-checker --path=/var/lib/jenkins/workspace/SuiteCRM/ --format=json > ~/reports/suitecrm/local-php-security-checker-report.json
exit 0
```

Execution of the script was added as a stage in jenkinsfile:

```
stage ('Running Local PHP Security Checker') {
    steps {
        sh 'bash ~/scripts/suitecrm/local-php-security-checker.sh'
    }
}
```

#### [Enlightn Security Checker](https://github.com/enlightn/security-checker){target="_blank"}



### SAST

#### [PHPStan](https://github.com/phpstan/phpstan){target="_blank"}

##### Installation

##### Usage

#### [Snyk](https://snyk.io/){target="_blank"}

##### Installation

##### Usage

#### [SonarQube](https://www.sonarqube.org/){target="_blank"}

##### Installation

##### Usage

### Code Quality Analysis

#### [PHP Mess Detector](https://github.com/phpmd/phpmd){target="_blank"}

##### Installation

##### Usage

### [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer){target="_blank"}

##### Installation

##### Usage

### DAST

#### OWASP Zed Attack Proxy (ZAP)

##### Installation

##### Usage

### Generating SBoM

### Deploying SuiteCRM

### Conclusion

Following the steps used to create a pipeline for DVNA

https://docs.suitecrm.com/admin/installation-guide/downloading-installing/#_copying_suitecrm_files_to_web_server

Other miscellaneous differences observed between pipelines for both applications are as follows:

- **Difference in code size: **There was a huge difference in the code size of both applications. 
- **Difference in report size: **
- **Difference in pipeline execution time: **