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

**Note: **Composer v2 is not compatible with a specific plugin for SuiteCRM named Wikimedia Composer Merge, this is a [known issue](https://github.com/wikimedia/composer-merge-plugin/issues/184){target="_blank"} hence I downgraded Composer to 1.x channel to continue with the setup.

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

The complete report generated by OWASP Dependency Check can be accessed [here](/reports/suitecrm/){target="_blank"}.

#### [Local PHP Security Checker](https://github.com/fabpot/local-php-security-checker){target="_blank"}

Local PHP Security Checker is a CLI based tool which scans application dependencies for known vulnerabilities by running them against the [Security Advisories Database](https://github.com/FriendsOfPHP/security-advisories){target="_blank"}.

##### Installation

Local PHP Security Checker is available as an executable for Linux which can be downloaded from the [releases section](https://github.com/fabpot/local-php-security-checker){target="_blank"} of the tool's Github page.

##### Usage

Since Local PHP Security Checker comes as an executable, a script was created to run the tool and placed in the `~/scripts/suitecrm/` directory located in the home directory of jenkins user. 

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

The complete report generated by Local PHP Security Checker can be accessed [here](/reports/suitecrm/){target="_blank"}.

### SAST

For the SAST phase, some tools from the DVNA pipeline were retained as they supported multiple languages. Some new tools for PHP were introduced to get better scans of the application.

#### [PHPStan](https://github.com/phpstan/phpstan){target="_blank"}

PHPStan (PHP Static Analysis Tool) is aimed to substitute the role of the compiler from other languages. It is focused on finding bugs and errors in PHP code without executing the code.

##### Installation

PHPStan is available as an executable for Linux which can be downloaded from the [releases section](https://github.com/phpstan/phpstan/releases){target="_blank"} of the tool's Github page.

##### Usage

Since PHPStan is an executable, a script was created to run the tool and placed in the `~/scripts/suitecrm/` directory located in the home directory of jenkins user. The required arguments for the tool were located in the [documentation](https://phpstan.org/user-guide/command-line-usage){target="_blank"}.

The script executed the tool placed in the tools directory with the flags required for getting output according to our requirements:

```
#!/bin/bash
cd ~/scripts/suitecrm
php phpstan.phar --no-interaction --memory-limit=2G --error-format=json analyse ~/workspace/SuiteCRM/ > ~/reports/suitecrm/phpstan-report.json
exit 0
```

Execution of the bash script was added as a stage in jenkinsfile:

```
stage ('Performing PHPStan Analysis') {
            steps {
                sh 'bash ~/scripts/suitecrm/phpstan.sh'
            }
        }
```

The complete report generated by PHPStan can be accessed [here](/reports/suitecrm/){target="_blank"}.

#### [Snyk](https://snyk.io/){target="_blank"}

snyk is an open-source security platform for finding out vulnerabilities in the source code of an application. 

##### Installation

snyk can be installed using Composer, which acts as the package manager for SuiteCRM. The installation was done by following the official [snyk guide for PHP](https://support.snyk.io/hc/en-us/articles/360003817397-Snyk-for-PHP){target="_blank"}.

##### Usage

To use the snyk CLI tool, a bash script was created with the required arguments and placed in the `~/scripts/suitecrm` folder along with other scripts.

```
#!/bin/bash

cd ~/workspace/SuiteCRM
snyk auth $SNYK_API_KEY
snyk test --json > ~/reports/suitecrm/snyk-report.json
exit 0
```

Execution of the bash script was added as a stage in jenkinsfile:

```
stage ('Performing snyk.io analysis') {
    steps {
        withCredentials([string(credentialsId: 'SNYK_API_KEY', variable: 'SNYK_API_KEY')]) {
            sh 'bash ~/scripts/suitecrm/snyk.sh'
        }
    }
}
```

The complete report generated by PHPStan can be accessed [here](/reports/suitecrm/){target="_blank"}.

#### [SonarQube](https://www.sonarqube.org/){target="_blank"}

SonarQube was retained from the DVNA pipeline as it supports PHP. It provides a detailed report about the application and scans various important parts of the code. It also provides a score in the dashboard along with all the bugs and errors which allows for easier access of the issues.

##### Installation

Since SonarQube server was [already installed](/static-analysis/#installation_3) for DVNA, a new project was added in the SonarQube dashboard and the scan was executed in the same manner as DVNA.

##### Usage

The SonarQube Scan execution was added directly to the jenkinsfile as the SonarQube plugin for Jenkins allows for seamless integration and direct scan execution.

```
stage ('Performing SonarQube analysis') {
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }

    steps {
        withSonarQubeEnv ('SonarQube') {
            sh '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey="SuiteCRM" -Dsonar.projectBaseDir="/var/lib/jenkins/workspace/SuiteCRM"'
        }
    }
}
```

The complete report generated by SonarQube can be accessed [here](/reports/suitecrm/){target="_blank"}.

### Code Quality Analysis

#### [PHP Mess Detector](https://github.com/phpmd/phpmd){target="_blank"}

##### Installation

##### Usage

The complete report generated by PHP Mess Detector can be accessed [here](/reports/suitecrm/){target="_blank"}.

**Note:** PHPMD

https://stackoverflow.com/questions/14456592/how-to-stop-an-unstoppable-zombie-job-on-jenkins-without-restarting-the-server/38481808#38481808

```
Jenkins .instance.getItemByFullName("SuiteCRM")
        .getBuildByNumber(36)
        .finish(hudson.model.Result.ABORTED, new java.io.IOException("Aborting build")); 
```

### [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer){target="_blank"}

##### Installation

##### Usage

The complete report generated by PHP Code Sniffer can be accessed [here](/reports/suitecrm/){target="_blank"}.

### DAST

#### OWASP Zed Attack Proxy (ZAP)

DAST for DVNA was done using the Official OWASP ZAP Jenkins plugin. Initially the pipeline was configured to use the same setup as DVNA, however after updating Jenkins, the OWASP ZAP Plugin was broken. Because the plugin is not being maintained anymore I decided to implement ZAP using docker for SuiteCRM.

##### Installation

Since we are using the Docker image of OWASP ZAP, we need to install docker on our machine. Installation of docker was simple and easy by following the official documentation. 

```
sudo apt install docker
```

##### Usage

```
docker run -v $(pwd):/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -t http://$(ip -f inet -o addr show docker0 | awk '{print $4}' | cut -d '/' -f 1):80 -g gen.conf -r testreport.html
```

The complete report generated by OWAP ZAP can be accessed [here](/reports/suitecrm/){target="_blank"}.

### Generating SBoM

```
stage ('Generating SBoM') {
    steps {
        sh 'composer require --dev cyclonedx/cyclonedx-php-composer'
        sh 'composer make-bom --json --output-file=/var/lib/jenkins/reports/suitecrm/sbom.json'
    }
}
```

The software bill of materials generated by CycloneDX can be accessed [here](/reports/suitecrm/){target="_blank"}.

### Deploying SuiteCRM

SuiteCRM was deployed on the application server using SSH commands. The application server is an Ubuntu 18.04 based EC2 instance with Apache and PHP pre-installed. The steps involved logging into the application server, copying the locally built dependencies and the application files and restarting Apache to get the application back up and running again.

Since the application configuration of SuiteCRM is stored in config.php, overwriting the file will result in running the SuiteCRM setup again. To prevent this, the existing config.php file on the application server was retained and rest of the files were removed and replaced with new ones from our Jenkins machine.

The followig steps were added as shell commands in the jenkinsfile and all the commands grouped together in the form of a pipeline stage. The application server IP and username were stored as secrets in Jenkins and passed as environment variables in the stage.

```
stage ('Deploying the application') {
    environment {
        APPLICATION_SERVER_IP = credentials('APPLICATION_SERVER_IP')
        APPLICATION_SERVER_USERNAME = credentials('APPLICATION_SERVER_USERNAME')
    }

    steps {
        sh '''
            ssh ${APPLICATION_SERVER_USERNAME}@${APPLICATION_SERVER_IP} "sudo rm -rf /var/www/html/*"
            scp -r /var/www/html/* ${APPLICATION_SERVER_USERNAME}@${APPLICATION_SERVER_IP}:/var/www/html
            ssh -T ${APPLICATION_SERVER_USERNAME}@${APPLICATION_SERVER_IP} "sudo cp ~/config.php /var/www/html/"
        '''
    }
}
```

### Conclusion

Following the steps used to create a pipeline for DVNA, I was able to create a similar if not identical pipeline for SuiteCRM. However, there was a major difference in both pipelines in multiple aspects which rendered some assumptions for the pipeline incorrect. The main differences observed between pipelines for both applications are as follows:

- **Difference in code size: **There was a major difference in the code size of both applications. SuiteCRM is a real world application with thousands of active contributors and around 10,000 files while DVNA is a demonstration app with around 150 files.

- **Difference in report size: **The reports for SuiteCRM contain more details as there are more files which are scanned. Going through each report individually is quite difficult since there are thousands of lines.

- **Difference in pipeline execution time: ** DVNA took 7-10 minutes whereas SuiteCRM took 30 minutes to an hour. The major difference in code size results in increased scan times resulting in long pipeline execution times.

The system requirements were also modified to meet the requirements for running the pipeline for SuiteCRM. Since DVNA was a small and lightweight application, the resources required were minimal. That was not the case for SuiteCRM as the Jenkins EC2 was constantly running out of memory and CPU usage was at 100% while running the pipeline, sometimes causing the EC2 instance to crash or freeze which required a manual reboot from the AWS console. The instance was upgraded from t2.medium to t2.large for extra 4GB memory. There was also a shortage of storage since the logs and reports generated by SuiteCRM took more space as compared to DVNA. The storage was expanded to 25GB by the time pipeline was complete.

There was also a difference of package managers for both applications. DVNA being node.js based used npm which allowed direct installation of some tools. SuiteCRM uses Composer as the package manager however most of the tools used were available as PHP executable binaries.

- **SCA: **For SCA, OWASP Dependency Checker was retained as it supported PHP. A new tool named Local PHP Security Checker was added to scan PHP files specifically.

- **SAST: **For SAST, two tools namely snyk and SonarQube were retained and a new PHP specific tool named PHPStan was added for complete static analysis of the application.

- **DAST: **For DAST, the tool remained the same (OWASP ZAP), however the implementation was done using Docker instead of the Jenkins plugin. The report produced by ZAP was similar however no major vulnerabilities were found as the application in maintained properly.

- **Deployment: **Since SuiteCRM is PHP based, the deployment was done on Apache2 instead of a process manager. The deployment was done on an AWS EC2 server similar to first phase of DVNA.

Implementing a real-world application has made me realise that each application has its own requirements and each pipeline is unique in some way depedning upon the usage and the requirements.