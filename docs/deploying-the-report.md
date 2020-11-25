# Deploying the report

## Objective

This section aims at completing the objectives listed as [`Task 2`](../problem-statement#task-2) under the [Problem Statement](../problem-statement).

## Setting up MkDocs

MkDocs is a simple static site generation framework aimed at building project documentation written in Python. It is highly customizable and offers flexibility. It is easy to host and uses Markdown based source files.

The installation was quick by following their official guide [here](https://www.mkdocs.org/#installation){target="_blank"}.

I chose to customize the report by opting for the Material theme and changing the font to Inter. The final `mkdocs.yml` file looks like this:


## Setting up Netlify

Netlify is a cloud provider that offers hosting and serverless backend services for web applications and static websites. We will be using their free services to integrate our MkDocs based documentation and 

## Setting up a custom domain

I am using Cloudflare as a DNS provider for my personal domain `https://nxio.us`. To access the report at my own domain I followed the guide provided by Netlify, which involved adding a CNAME record with the desired subdomain as the `Name` and the custom domain on Netlify as the `Content`.

Proxying the record through Cloudflare provides additional features such as a CDN, HTTPS and optimized loading times.

![Cloudflare Settings](images/Cloudflare.png)