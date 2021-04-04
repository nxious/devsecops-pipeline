# Webhook setup

## Objective

This section aims to accomplish the objective listed as 6<sup>th</sup> point of [`Task 2`](../problem-statement/#task-2) under the [Problem Statement](../problem-statement).

We will be setting up a Github webhook which will trigger a build automatically on our Jenkins instance upon release.

## What is a webhook?

A webhook is a method of exchanging real-time information between applications, often triggered by an event. The exchange can occur using one of the two methods, either by polling or by a webhook. While both methods share similarities, webhooks are far more efficient as polling involves sending request at a predefined frequency for updates which introduces unnecessary server load. Since most of polls are wasted as they keep fetching old information unless an update is there, webhooks maintain 100% efficiency as they only call when there is an update instead of checking every x seconds for an update.

We will be using a webhook from Github to trigger a build on our Jenkins machine. In our setup, the pipeline will be executed on releases. This means whenever there is a new release of the application, the build will get triggered automatically. To achieve this we need to configure the webhook settings in Github as well as Jenkins. 

## Setting up the webhook

Github offers various options for webhooks. We can decide which action leads to a call on the webhook. For our application, we will be setting our pipeline trigger to release as we only need to execute the pipeline once a major update for the application is pushed. The webhook was setup by following this [guide](https://dzone.com/articles/adding-a-github-webhook-in-your-jenkins-pipeline){target="_blank"}. We can navigate to the webhook settings in Github by navigating to our `Project's repository -> Settings -> Webhook` and can see the following options:

![Webhook Settings](./images/webhook-settings.png)

The webhook URL for Jenkins is `http://JENKIN_SERVER_IP:PORT/github-webhook/` which for our instance is `http://3.XXX.XXX.XXX:8080/github-webhook/`. This URL was set as the `Payload URL` in the Github settings. Since Jenkins accepts the payload in `JSON` format, we will choose that from the dropdown as well. We can also provide a secret if we need to verify that the request is in fact from Github. Now we need to configure the event(s) upon which the build will be triggered. We will select `Let me select individual events.` and enable the `Releases` option as we need to trigger the build only on release. Finally we will set the webhook to `Active`.

Since our Jenkins EC2 instance has a security group which only allows access to my IP, we need to allow Github's Webhook IP ranges to access Jenkins which is running on port 8080. According to the [documentation](https://docs.github.com/en/github/authenticating-to-github/about-githubs-ip-addresses){target="_blank"} provided by Github, we can access the IP ranges using Github's [meta API](https://api.github.com/meta){target="_blank"}. We will whitelist the hooks IP ranges as they represent the IP blocks from which Github will be sending a trigger to Jenkins. 

Now we can test our webhook by either pushing a release on Github or redlivering a previous delivery. Furhter steps are listed in Github's [documentation](https://docs.github.com/en/developers/webhooks-and-events/testing-webhooks){target="_blank"}. 

