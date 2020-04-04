---
title: "Building a helpful Android CI with Danger & Jenkins"
author: "Pavlos-Petros Tournaris"
date: 2018-08-29T08:17:55.141Z
lastmod: 2020-04-05T01:28:51+03:00

description: ""

subtitle: "In our days, almost every project, is backed by a Continuous Integration system (aka CI). Either that is an Open Source project or a…"

image: "/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/1.png" 
images:
 - "/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/1.png" 
 - "/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/2.png" 
 - "/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/3.png" 
 - "/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/4.png" 


aliases:
    - "/building-a-helpful-android-ci-with-danger-jenkins-bf751be7a74c"
---

![image](/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/1.png)

An example of our testing pipeline

In our days, almost every project, is backed by a **Continuous Integration** system (aka **CI**). Either that is an Open Source project or a private repository in the organization of your company.
> Why do I need a CI?

Well, CI is what will have your back, at any given time. We are engineers and we might make mistakes from time to time, but 99% of those times our mistakes could have been easily detected by the proper tools, running in our CI, for each PR or commit.

To give you a real life example, we were using `.forEach` on `Map` which is only available on API 24+ but because we were targeting Java 1.8 we did not get any warnings, even though this was causing a fatal exception due to `NoClassDefFoundError` on APIs below 24. Other than searching for such usages and fixing them we also decided to write a Lint rule that will protect us in the future, in case someone forgets about this limitation (will probably explain in another article, how to write such a rule).
> What tools are you talking about?

I will mention some of the tools currently being used in modern Android stacks.

1. Detekt

2. KtLint

3. FindBugs

4. ErrorProne

5. Android Lint

While some of those can be used only for code style and formatting (e.g. KtLint) others can really detect bugs using static analysis (e.g. ErrorProne, FindBugs or even Android Lint with custom rules).

A way to use the tools above is by adding a `pre-commit` hook that will prevent you from committing un-styled or non-formatted code. Such a setup, you can find in [Squanchy](https://github.com/squanchy-dev/squanchy-android/blob/develop/team-props/git-hooks.gradle) repository. But most of them, will take some time to execute and having to run them, before being able to commit, is not viable. That is where a CI comes to offload that task from you.

### Our beloved CI

There are many solutions in the market that need minimal setup or are quite easy to integrate, such as CircleCI, TravisCI or Google Cloud Build. In Workable’s Mobile Team, we have developed our own in-house CI, based on already existing infrastructure, powered by Jenkins. Of course, one might say, why would you need to “invent the wheel” all over again? Well, most of those services, have paid plans for private repos and are quite costly for our needs. Having 2–3 Jenkins slaves in-house, felt a lot more cost effective!

In addition, it’s always better to build something on your own, rather than using an already established service :P It makes you feel a bit geeky!

### Danger Systems
> Danger runs during your CI process, and gives teams the chance to automate common code review chores.

Such chores, are potential nullability bugs, formatting issues or even prohibited usages of methods, like the one we mentioned before.

We used Jenkins’ **Github Pull Request Builder** plugin to invoke a Jenkins job for PR commits. After that, all you need to do, is add a file on your repo named`Dangerfile`. In this file, you let Danger know, what plugins to invoke and/or whatever you want it to post on your PR.

For example, you can let reviewers know, that the current PR is a work in progress:
`if (github.pr_body + github.pr_title).include?(&#34;WIP&#34;)  
   warn(&#34;Pull Request is Work in Progress&#34;)  
end`

What we found useful, was the following JUnit Danger plugin, which outputs unit test results, on your PR:
`junit_tests_dir = &#34;path/to/your/results/**/*.xml&#34;  
Dir[junit_tests_dir].each do |file_name|  
  junit.parse file_name  
  junit.show_skipped_tests = true  
  junit.report  
end`



![image](/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/2.png)



Lastly, we also used Danger’s Android Lint plugin to output any Lint issues as well, on the PR:
`android_lint.report_file = &#34;path/to/your/lint-results.xml&#34;  
android_lint.filtering = true  
android_lint.skip_gradle_task = true  
android_lint.lint(inline_mode: true)`



![image](/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/3.png)

An example of a custom Lint rule result reported inline in the offending file.



As you might have noticed, JUnit &amp; Lint plugins, expect to find the result files and just parse them.

In order to succeed on this task and prevent it from failing the execution, with, we had to use the following Gradle command:
`./gradlew   
--continue yourTestTask \  
--tests=your.test.suite \   
yourLintTask \  
-PabortOnLintError=false`

*   We use**— continue** in order for **testDebug** task to keep executing when a test has failed.
*   Use **abortOnLintError** parameter in order to let Lint also finish its execution when an error is produced.

We also need to add that extra snippet in our root **build.gradle** file, to allow **yourTestTask** ignore any failures and let Danger report them:


Root build.gradle file

### But…

Danger is a PR only tool and unfortunately you can not bypass its hardcoded checks that detect whether you are using a CI system. We could supposedly take advantage of its reporting tools on Github’s Commit Status API and turn off any commenting functionality.### What about non PR commits?

As I mentioned before, we already had a custom CI in place, that was running our tests on every commit, and report back to Slack, should the build or tests have broken.

It was time to enrich it with more features like static analysis and also integrate it with Github’s Commit Status API.

We are also using [fastlane](https://fastlane.tools/) to automate some tasks like test running, APK building and uploading either to Fabric Beta or PlayStore. All of the above tasks, are executed in what fastlane calls “lanes”. Each lane is assigned a specific task and you essentially “drive” that lane when you want to execute it.

All we had to do, was create a lane that will run Android Lint and [Detekt](https://github.com/arturbosch/detekt) and use that one along with the test running lane. Apart from that, it would be great to have Jenkins also post on Github’s Commit Status API, for the ongoing state of the lane run.




The above snippet executes an HTTP request using Jenkins’ **httpRequest** plugin, in order to update a given commit’s state.

The `requestBody` in the example above is something similar to the following:
`{  
  &#34;state&#34;: commit-status,// values: failure, success, pending, error  
  &#34;target_url&#34;: your-current-build-url,  
  &#34;description&#34;: &#34;A description&#34;,  
  &#34;context&#34;: context // A differentiation label  
}`

All that is left now, is to call `postCommitStatus `method, when our build initiates, to mark the commit as **pending** and when the lane finishes, update the commit state to **successful** or **failed**.




![image](/posts/2018-08-29_building-a-helpful-android-ci-with-danger-jenkins/images/4.png)

Showcasing all possible states of a commit

### Conclusion

After having written a bit of CircleCI and TravisCI configurations, I have to say that configuring Jenkins was 100x more difficult. When your company though, has an in-house Jenkins infrastructure, you can avoid spending money on third-party services.

As always, you should choose what is best for you and your needs. In case you happen to be using Jenkins, like we do, I hope this article helped you make CI a friend of yours.

After all, this is what will help you keep your crash-free rate above 99.5% 🙌
