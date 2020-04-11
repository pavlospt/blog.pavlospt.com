---
title: "Effectively disabling Crashlytics buildId re-generation in KotlinDSL"
author: "Pavlos-Petros Tournaris"
date: 2019-05-25T12:17:26.030Z
lastmod: 2020-04-05T01:28:57+03:00

description: ""

subtitle: "Android Studio 3.5 has reached beta and it was time to try it out as a daily driver in work. Especially with Project Marble having…"

image: "/images/2019-05-25_effectively-disabling-crashlytics-buildid-regeneration-in-kotlindsl/1.png"
images:
  [
    "/images/2019-05-25_effectively-disabling-crashlytics-buildid-regeneration-in-kotlindsl/1.png",
  ]

aliases:
  - "/effectively-disabling-crashlytics-buildid-re-generation-in-kotlindsl-a3025d7c38c8"
---

![image](/posts/2019-05-25_effectively-disabling-crashlytics-buildid-regeneration-in-kotlindsl/images/1.png)

Image taken from ProAndroidDev

Android Studio 3.5 has reached beta and it was time to try it out as a daily driver in work. Especially with Project Marble having introduced so many bug fixes and performance improvements, that meant the benefits would have an immediate effect.

Android Studio 3.5 also includes a new version of “instant run” written from scratch. Since we had it disabled in order to avoid possible build issues with the previous implementation, it was time to give the new “instant run” aka “Apply changes”, a try.

### The problem

After enabling “Apply Changes” we tried changing something and just pressing the “Apply Code changes” button to check how fast it would work. After it finished building, the changes failed to take effect because and there was an issue reported in red color on Android Studio.

That issue was outlining that Resources had changed and that was the reason it failed to apply any changes made like it was supposed to. But it also included the offending resource file, which was: `asssets/crashlytics-build.properties` located in `merged_assets/your_build_type/` of the main app module usually.

Immediately we thought that this was not supposed to happen, because we had disabled Crashlytics **buildId** re-generation in order to speed up build times and have better task caching for kapt &amp; Kotlin compilation.

### A bit of background

Our Gradle files were all written in Groovy, but one time we decided to give KotlinDSL a try and migrated all of them.

In Groovy all we needed to do in order to disable Crashlytics **buildId** re-generation was to add the following:

`ext { alwaysUpdateBuildId = false }`

which translated to KotlinDSL was similar to:

`ext.set(&#34;alwaysUpdateBuildId&#34;, false)`

The above line was placed in a custom build type which was the one we were using to init some of the rest of our build types, that are being used for different staging servers. The setup was similar to something like the following:

Apparently this solution was not correct. Since “Apply Code changes” yield such an error, we started investigating and actually trying out different solutions.

### The actual solution

Fabric advises to use the syntax displayed above in order to disable **buildId** re-generation but KotlinDSL was not very fond of that. It seems that there is a compatibility issue between Groovy’s dynamic type generation and the way KotlinDSL works, so the flag was not properly propagated in order to disable re-generation.

After searching a bit around we found a few different ways to write the same thing, either on Github or on StackOverflow. Some of them, are the following:

The correct one can be found on line 5. This solution though had to be manually applied on each **buildType** since it seems that it was not propagated when using the `initWith` function on custom **buildTypes**.

#### Explanation

As user Ivan Tokic mentions on their [answer](https://stackoverflow.com/a/55745719/1470614):

> You can only call `extra` on `ExtensionAware` objects, with `BuildType` not being declared as one.> However, during runtime, build types actually are `ExtensionAware`, which is why this works in Groovy due to its dynamicity, but not in Kotlin where `extra` in this scope will reference the `Project`&#39;s extensions.

This was actually the reason that explained the mishap we had between our transition from Groovy to KotlinDSL.

### More on this

While we were searching around for a fix and the reasoning, we also found a repo reproducing the error from a Gradle engineer: [https://github.com/autonomousapps/Crashlytics-KDSL-Issue](https://github.com/autonomousapps/Crashlytics-KDSL-Issue). Another solution that we found being suggested and was also suggested to us, was the creation of a `*.gradle` that would contain the disabling flag in **extras** written in Groovy, which would then be applied to the `build.gradle` file of the module that was utilizing the Fabric Gradle plugin:

`apply(from = &#34;*.gradle&#34;)` in `app/build.gradle` (most probably it is the main app module utilizing Fabric Gradle plugin)

### **Conclusion**

We are very excited to try out the new “Apply Changes” feature and also very happy to see such detailed error reporting, which actually saved quite a few seconds in build time for us.

One thing to also keep in mind, is that when **transitioning** between Groovy and KotlinDSL we should be more careful from now on, since it is entirely possible to miss some details quite crucial to our daily workflow.

It is very re-assuring though to know that the Android community is great and there will always be out there someone that could help you debug the issue and provide a solution to your problem!

Thank you Ivan Tokic for that SO answer :D !!!
