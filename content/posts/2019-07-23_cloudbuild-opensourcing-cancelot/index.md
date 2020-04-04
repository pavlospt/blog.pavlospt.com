---
title: "CloudBuild: Open-sourcing Cancelot"
author: "Pavlos-Petros Tournaris"
date: 2019-07-23T19:54:06.057Z
lastmod: 2020-04-05T01:29:00+03:00

description: ""

subtitle: "CloudBuild is a CI/CD offered by Google. At Workable’s Mobile department we made a proof of concept to check if CloudBuild fit’s our needs…"

image: "/posts/2019-07-23_cloudbuild-opensourcing-cancelot/images/1.png" 
images:
 - "/posts/2019-07-23_cloudbuild-opensourcing-cancelot/images/1.png" 


aliases:
    - "/cloudbuild-open-sourcing-cancelot-de58f3ceaa83"
---

> Automatically cancel running builds on the same branch, when a new commit is pushed



![image](/posts/2019-07-23_cloudbuild-opensourcing-cancelot/images/1.png)

Image taken from: [https://cloud.google.com/images/products/cloud-build/cloud-build.png](https://cloud.google.com/images/products/cloud-build/cloud-build.png)



CloudBuild is a CI/CD offered by Google. At [Workable](https://medium.com/u/e5ecc5b405f2)’s Mobile department we made a proof of concept to check if CloudBuild fits our needs in order to be used for artifact (APK) building.A complex project requires a lot of custom configuration in order to properly set it up and we were able to find most of them in the great CloudBuilders community repo.

### Cancelling a running build

CloudBuild builds though come at a cost and depending on your budget and the machine type you use. So having builds run for multiple commits that are pushed on a branch, means a bigger cost on your monthly bill.

As you might have guessed this is something that would not work in case multiple members of our team were working on the same branch and pushing commits with a high frequency.

We need to find a solution that would let us cancel any previously running builds in favor of the new ones.

### Entering Cancelot

Cancelot is the name of the Builder we built for the purpose described above. Initially our team had named it “Ralph” from the movie “Wreck-it Ralph”, but Twitter is so resourceful and came up with the name “Cancelot”.

Cancelot is added as a buildstep on your CloudBuild YAML file and is responsible to check for any builds that are currently running for the same branch.

It makes use of the CloudBuild Go package in order to fetch the running builds, by filtering with the following criteria:
``build_id != &#34;[CURRENT_BUILD_ID]&#34; AND   
source.repo_source.branch_name = &#34;[BRANCH_NAME]&#34; AND   
status = &#34;WORKING&#34; AND   
start_time&lt;&#34;[CURRENT_BUILD_START_TIME]&#34;``

After successfully fetching any previously started (&amp; still running builds) it loops through them and cancels them.

#### Usage

Cancelot’s usage and setup is inspired by the CloudBuilders “slackbot” so it is very easy to use it.

*   Deploy the GCR image of Cancelot to your project’s GCR
*   Add Cancelot’s buildstep in your CloudBuild YAML file`steps:  
- name: &#39;gcr.io/$PROJECT_ID/cancelot&#39;  
  args: [   
    &#39;--current_build_id&#39;, &#39;$BUILD_ID&#39;,  
    &#39;--branch_name&#39;, &#39;$BRANCH_NAME&#39;  
  ]`

### Open Source

We wanted to give back to the community, just like we were able to use the great stuff that other members have built. Cancelot currently lives on an open PR in CloudBuilders community repository. You are free to use it at your own will until the PR is either accepted or closed 😄

Edit: **Cancelot** has now been merged into CloudBuilders repo :D You can find the official builder there!

[[New builder] Introduce Cancelot by pavlospt · Pull Request #271 ·…](https://github.com/GoogleCloudPlatform/cloud-builders-community/pull/271)
