---
title: "Restructuring Litho’s CircleCI config for Workflows"
author: "Pavlos-Petros Tournaris"
date: 2018-05-19T20:47:30.181Z
lastmod: 2020-04-05T01:28:46+03:00

description: ""

subtitle: "Nearly a year and a half ago, Facebook released Litho as an Open Source project. Litho is an Android framework, which lets you define your…"

image: "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/1.jpeg" 
images:
 - "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/1.jpeg" 
 - "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/2.png" 
 - "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/3.png" 
 - "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/4.png" 
 - "/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/5.png" 


aliases:
    - "/restructuring-lithos-circeci-config-for-workflows-38bb9e28f56a"
---

![image](/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/1.jpeg)

Nearly a year and a half ago, Facebook released [Litho](https://fblitho.com/) as an **Open Source** project. Litho is an **Android** framework, which lets you define your UI in a declarative way. It immediately got my interest and I started getting my hands dirty with some examples and pet projects.It was a nice experience getting in touch with Litho and its React-like nature, that was a first for me. The amount of interest in that new area, made me realise that I could dive a bit more into it, by also contributing to the project. So just like any other **Open Source** project, that I like I started checking the “Issues” tab to see if I could help resolving any bugs or contributing new features.

### Automated testing

Litho, just like the majority of **Open Source** projects, includes a lot of tests (without tests it usually is hard to gain a developer’s trust). Testing infrastructure on Litho is based on Unit tests that are run either through [BUCK](https://buckbuild.com/) or using **Gradle**.

But tests alone, do not mean anything, if you are not able to have proper feedback on your PRs and also make sure each commit that goes into `master` branch is “green”.

Litho was using **CircleCI** to leverage the run of its Unit test suite, gather test results and publish a **SNAPSHOT** for each commit pushed on `master` .
> CircleCI and its configuration can certainly prove a bit tricky or hard to get around. But let’s hope that at the end of this, you will be able to understand more of how it works and set it up on your own projects as well.

### Previous situation

Previously, Litho was using a custom Docker image made by [Pascal Hartig](https://medium.com/u/19026c4fef9d), one of Litho’s Android engineers.

This image was downloading **BUCK**, building it and saving it on the Docker environment. The same was done for **Android NDK** and **Android SDK** respectively. When **CircleCI** started a build for a commit, it would then configure some needed keys for archives uploading and also exporting BUCK into **PATH**. Finally, it started executing all **BUCK** &amp; **Gradle** builds for sample projects, as well as executing BUCK &amp; Gradle tests, which was finalized by publishing a **SNAPSHOT** and storing test results.

While this worked great, there was the need to configure parallel jobs for **BUCK** &amp; **Gradle** tests &amp; sample builds.

### Problems with the previous situation

While starting to investigating what could be improved, there were certain things I noticed, that could be more performant and others that came up after discussing them with Pascal.

*   **CircleCI** configuration was using a custom Docker image that was effectively pulled each time the Job was running, due to how CircleCI caches docker images on its containers, costing around 5–6 mins of spinning an environment.



![image](/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/2.png)

*   Builds &amp; tests were not running in parallel, which meant that even if a **BUCK** build required 2mins to execute tests, it would have to wait for **Gradle** to also finish executing tests before proceeding.
*   The regex used to collect test results was not collecting **BUCK** test results.#### Here come Workflows

**CircleCI** provides a feature, called **Workflows**. Workflows allow us to define a list of **Jobs** that will run in parallel, but you can also declare dependencies between jobs, which would effectively turn their execution to be sequential.
`- checkout_code        
- build:            
    requires:              
      - checkout_code        
- buck_sample_build:            
    requires:              
      - build        
- buck_sample_barebones_build:            
    requires:              
      - build        
- buck_sample_codelab_build:            
    requires:              
      - build        
- gradle_sample_kotlin_build:            
    requires:              
      - build        
- buck_litho_it_tests_run:            
    requires:              
      - build        
- buck_litho_it_powermock_tests_run:            
    requires:              
      - build        
- gradle_tests_run:            
    requires:              
      - build`### Transitioning Litho to use Workflows

First step was to create a Job that would checkout the repo code and start setting up required dependencies, including: **BUCK**, dependencies that will help us build **BUCK**, such as **ant**, **** and **Android NDK**. This Job was defined as a first step on Litho’s `build_and_test` Workflow.

Depending on that Job, is `build` Job, which is responsible for building Litho and saving any **Gradle** produced caches.

Afterwards, `build` is faning-out to 7 other Jobs responsible to build samples and execute tests. Each one of these Jobs is also configured to store test results and upload any produced artifacts.




![image](/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/3.png)

Fan-out example (from circleci.com)



This **Workflow** is finalized by `publish_snapshot` Job which is depending on every Job that executes tests. After their successful execution, a Gradle task is responsible to upload an archive on Bintray containing the latest changes, as a **SNAPSHOT**.
`- publish_snapshot:            
    requires:              
      - buck_litho_it_tests_run              
      - buck_litho_it_powermock_tests_run              
      - gradle_tests_run`

### Caches &amp; Workspace

CircleCI’s config DSL, includes amongst others, `cache` &amp; `workspace`. In order to better explain those terms let me define them like following:

*   `cache` is saved content that will be retained between Workflow executions



![image](/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/4.png)

Caching (from circleci.com)

`- &amp;save-repo-cache      
  paths:        
    - ~/.gradle/caches        
    - ~/.gradle/wrapper`

*   `workspace` is saved content that will be retained among Jobs executions of the same Workflow



![image](/posts/2018-05-19_restructuring-lithos-circleci-config-for-workflows/images/5.png)

Workspace (from circleci.com)

`- persist_to_workspace:            
    root: workspace            
    paths:              
      - repo`

So generally, something that would worth caching is for example any Gradle cached dependencies, provided that there is a proper cache busting mechanism in place. Whereas our repo’s code is something that all Jobs need so it’s worth persisting it to our `workspace` in order for every Job to be able to “attach” and get access to that content.
`attach_workspace: &amp;attach_workspace    
  attach_workspace:      
    at: ~/litho-working-dir/workspace`

### Job configuration

Jobs that are part of a **Workflow** can also define `filters` filters for branches. This is especially helpful, since we would not like to execute `publish_snapshot` Job in case the commit we were running on is not on `master` branch, but rather a **PR**. Might seem like a small change but it can save 7–8 mins of waiting for the Workflow’s execution to finish.
`filters:              
  branches:                
    only: master`

### Post restructure improvements

After a looot of commits, Workflow executions and discussion with Pascal on improvable points, we finally managed to have a Workflow running in around 15–20mins. Points that we improved:

*   **Removed** custom Docker image, by using CircleCI’s Android image and pulling any needed dependencies on start-up. Gave us a 1sec container configuration for 95% (or even more) of Job executions.
*   **Using** CircleCI’s Android Docker image Android SDK, instead of downloading from scratch.
*   **Make** test &amp; build Jobs actually **run** **in parallel**, allowing for **early** **feedback** in case something is broken only on a “sample” module for example. Combined with Github’s integration this is really **informative**, because you can instantly be notified if a Job responsible to execute tests for your changes, has failed or not, without having to wait for the whole **Workflow** to finish.
*   **Collect** previously not collected test results from **BUCK** tests executions.### Conclusion

It has been a fun experience and eye-opening experience. I have never dealt with CI tools in that extent before and I can totally say that DevOps or CI provisioning is a certainly a difficult job (respect to our colleagues out there who have to deal with that every day).

Also, thanks to [Pascal Hartig](https://medium.com/u/19026c4fef9d) for helping on the process and discussing points of improvement.

You can find the commit that restructures `config.yml` here: [https://github.com/facebook/litho/commit/2c411830d11d08fc2af194d8ea244d0cbdf33f76](https://github.com/facebook/litho/commit/2c411830d11d08fc2af194d8ea244d0cbdf33f76)

And Litho’s final `config.yml` here: [https://github.com/facebook/litho/blob/master/.circleci/config.yml](https://github.com/facebook/litho/blob/master/.circleci/config.yml)

I would be totally happy to help and answer questions if you have trouble restructuring towards Workflows on your project :)
