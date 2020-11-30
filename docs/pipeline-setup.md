# Jenkins Pipeline Setup

## Objective

This section aims to accomplish the objectives listed as 3rd point of [`Task 1`](../problem-statement/#task-1) under the [Problem Statement](../problem-statement).

## What is a Pipeline?

In Jenkins, a pipeline is a group of events or jobs which are interlinked with one another in a sequence. These sequences helps us automate the building, testing and deployment of the application.

## Setting up the Jenkins pipeline

We have already setup Jenkins on a VM in the previous section. Now we will be using Jenkins to build a pipeline which will be responsible for fetching the latest application code from Github, building it and then copying it over to the application VM and deploying it.

Jenkins provides a friendly web-interface to achieve all this. Equipped with the official pipeling building [guide](https://www.jenkins.io/doc/pipeline/tour/hello-world/){target="_blank"}, I took the following steps in order to setup the pipeline:

### General

- Make a new pipeline project by clicking on `New Item` on the left side menu.
- Enter the name of the project, which in my case is `DVNA`.
- Check the `Discard old builds` option, as we do not have a use for previous builds.
- Check the `Github Project` option and provide the URL for the project as we will be fetching code from Github.

### Build triggers

- Check the `GitHub hook trigger for GITScm Polling` option as it allows us to trigger builds automatically using GitHub hooks. We can set the condition upon which a new build will be triggered.

### Pipeline

- From the Definition dropdown, select the `Pipeline script from SCM` option as we will be using a `jenkinsfile`, a script which is used to orchestrate the build process. This file will be included in the root directory of our application repository on GitHub.

## Configuring jenkinsfile

The `jenkinsfile` is a script that contains the definition of how the Jenkins Pipeline will function. It allows us to define the various steps and actions that will be taken by Jenkins in order to work with our application spread over various stages. I went with a 2-stage approach, covering the building followed by deployment of the application. The application will be built on the Jenkins machine and then will be copied over to the application VM to be deployed.

The structure of my `jenkinsfile` is as follows:

```
pipeline {
    agent any
    stages {
        stage ('Building the application') {
            steps {
                sh 'npm install'
            }
        }

        stage ('Deploying the application') {
            environment{
                MYSQL_USER = 'root'
                MYSQL_DATABASE = 'dvna'
                MYSQL_PASSWORD = 'passw0rd'
                MYSQL_HOST = '127.0.0.1'
                MYSQL_PORT = '3306'
            }

            steps {
                sh '''
                    ssh common@192.168.1.7 "cd dvna && pm2 stop DVNA"
                    ssh common@192.168.1.7 "rm -rf dvna && mkdir dvna"
                    rsync -r * common@192.168.1.7:~/dvna
                    ssh -T common@192.168.1.7 "cd dvna && MYSQL_USER=${MYSQL_USER} MYSQL_DATABASE=${MYSQL_DATABASE} MYSQL_PASSWORD=${MYSQL_PASSWORD} MYSQL_HOST=${MYSQL_HOST} MYSQL_PORT=${MYSQL_PORT} pm2 start --name=DVNA npm -- start"
                '''
            }
        }
    }
}
```

**Note: ** While using individual shell commands, make sure that you keep a track of the directories. Each SSH command will set the `PWD` to the root of the home directory of the user. My earlier approach consisted of single line sh commands to remove old build, copy new build and then execute it. I face the error where my app was not found by node as I was not in the correct directory.

The application required environment variables setup for it to function properly. These were passed directly to the node application in the last step.

**Note: ** There are many other ways to pass environment variables to the application server, however the method above worked best for my situation. You can use Jenkins plugins, shell scripts etc. depending upon you use case and purpose.