# Jenkins Pipeline Setup

## Objective

This section aims to accomplish the objectives listed as 3rd point of [`Task 1`](../problem-statement/#task-1) under the [Problem Statement](../problem-statement).

## Pipeline

In Jenkins, a pipeline is a group of events or jobs which are interlinked with one another in a sequence. These sequences helps us automate the building, testing and deployment of the application.

## Setting up the Jenkins pipeline

We have already setup Jenkins on a VM in the previous section. Now we will be using Jenkins to build a CI pipeline which will be responsible for fetching the latest application code from Github, copying it over to the application VM and run the application.

Jenkins provides a friendly web-interface to achieve all this. Equipped with the official pipeling building [guide](https://www.jenkins.io/doc/pipeline/tour/hello-world/){target="_blank"}, I took the following steps in order to setup the pipeline:

### General

1. Make a new pipeline project by clicking on `New Item` on the left side menu.
2. Enter the name of the project, which in my case is `DVNA`.
3. Check the `Discard old builds` option, as we do not have a use for previous builds.
4. Check the `Github Project` option and provide the URL for the project as we will be fetching code from Github.

### Build triggers

5. Check the `GitHub hook trigger for GITScm Polling` option as it allows us to trigger builds using GitHub hooks. We can use specific events to trigger a build such as release etc.

### Pipeline

5. From the Definition dropdown, select the `Pipeline script from SCM` option as we will be using a `jenkinsfile`, a script which is used to orchestrate the build process. This file will be included in the root directory of our application repository on GitHub.

## Configuring jenkinsfile

The `jenkinsfile` is a script that contains the definition of a Jenkins Pipeline. It allows us to define the various steps and actions that will be taken by Jenkins in order to work with our application in various stages. I went with the a 2-stage approach, covering the building and deploymwent of the application. The application will be built on the Jenkins machine and then will be copied over to the application VM to be deployed.