---
title: "Reducing Android Gradle module configuration boilerplate"
author: "Pavlos-Petros Tournaris"
date: 2019-07-20T18:04:31.577Z
lastmod: 2020-04-05T01:28:58+03:00

description: ""

subtitle: "Our codebase at Workable’s Android app has grown over the years due to the wide variety of features we offer. As most of you already know…"

image: "/posts/2019-07-20_reducing-android-gradle-module-configuration-boilerplate/images/1.png" 
images:
 - "/posts/2019-07-20_reducing-android-gradle-module-configuration-boilerplate/images/1.png" 
 - "/posts/2019-07-20_reducing-android-gradle-module-configuration-boilerplate/images/2.png" 


aliases:
    - "/reducing-android-gradle-module-configuration-boilerplate-93ee661c80c4"
---

![image](/posts/2019-07-20_reducing-android-gradle-module-configuration-boilerplate/images/1.png)

Image taken from: [https://bit.ly/32EDPVH](https://bit.ly/32EDPVH)



Our codebase at [Workable](https://medium.com/u/e5ecc5b405f2)’s Android app has grown over the years due to the wide variety of features we offer. As most of you already know, a single monolithic Android app is often one factor that contributes to slow build times.

One of the proposed ways of reducing build times, is to split the monolithic `app` module into smaller modules, that could be cached and compiled once. The suggested way of achieving that is to create more Kotlin/Java modules in comparison to Android modules and have the least possible inter-dependencies between those modules in order to avoid changes that would cause more upstream modules to re-compile.After deciding on the ideal way for us to structure each new module, we started un-tangling our codebase and creating simple and small modules.

Most of them though had to be Android ones, since we needed to use Android dependencies or they represented a feature of the app.### Creating a new module

When creating a new module, we needed to have the same `compileSdkVersion — targetSdkVersion — minSdkVersion` values like the ones existing in the `app` module. This was a trivial task at first since all those versions were referenced from a `versions.gradle` file and they would automagically generate proper accessors.

#### Entering KotlinDSL

When we decided that we should move all our `build.gradle` files to KotlinDSL, the automagic generation of accessors became a little bit of boilerplate since now we needed to reference all those properties in the following fashion, before using them:
`val compile_sdk_version: Int by rootProject  
val min_sdk_version: Int by rootProject  
val target_sdk_version: Int by rootProject`

The way this worked, is that we applied the `versions.gradle` file on the root level `build.gradle` file, so we could reference all these values by the `rootProject` property delegate.

_It might not be the best way to do it, but this is what we have figured and was working for us at the time :D_#### The boilerplate

As you can imagine, having to copy all those properties on each modules `build.gradle.kts` file and specifying the `android {}` configuration every time, created a huge boilerplate process and added several not needed lines.

Here is an example of a new module’s `build.gradle.kts` :




All this configuration is 11 lines, but there are more things that would add to those lines. For example if we specified `compileOptions {}` as well, it would reach 15 lines. Now multiply that x120 (which is the number of our modules currently) and right there you have 1800 lines of excess code and code prone to breaking all over the place if we made a single change on one of those properties for example.

### The solution

The solution was not something new, it was just that we never tried it before. Might that be because we underestimated the number of modules we would produce or the fact that we did not have the time to devote in finding a better solution, at that point :D

#### Root build.gradle.kts

The root `build.gradle.kts` file gives us access to `allprojects` &amp; `subprojects`

*   `allprojects` Are all the Gradle modules, including the root one
*   `subprojects` Are all the Gradle modules, excluding the root one

The proper place to include our single configuration, as you have guessed is the `subprojects` since we did not need to configure anything on the `rootProject` .

#### Configuring based on App/Library Android plugin

Before adding any new configurations, we needed to check for each project in `subprojects` that it was applying either the `AppPlugin` or the `LibraryPlugin` since we wanted to configure the `android` part for those only (Kotlin &amp; Java modules have separate configurations that can be abstracted and configured once as well).

The root `build.gradle.kts` ended up looking somewhat like the following example:




#### Diving in the code

If we take a closer look at the code above, we can see that the following things are happening:

*   Accessing `PluginContainer` in order to configure a `project` only when a `plugin` is added.
*   Based on the `plugin` class configuring separate things. `AppPlugin` can also specify `versionCode` &amp; `versionName`

Another good observation here is that we use `apply` on the `AppExtension/LibraryExtension` , as well as `defaultConfig` &amp; `compileOptions` . The reason we are doing this, is because we do not want to override anything that was specified on the module’s `build.gradle.kts` . We just need to append those as well, so we retain the flexibility of using things like:
`android { dataBinding.isEnabled = true }`

without having to replicate the whole configuration in the modules that would need some more configuration.

Of course, there are more abstractions that can be applied to the above example, but this is left as a practice for the reader (hint: `BaseExtension` :D)

### Conclusion

An interesting impact of that refactoring, is the additions/deletions in the PR applying the changes:




![image](/posts/2019-07-20_reducing-android-gradle-module-configuration-boilerplate/images/2.png)



Who would not be happy with that kind of change? :)

We are pretty confident that this will let us create more modules in the future, since we reduced some of the friction of creating one! I suggest you to try it even if you have a smaller number of modules :)

_Would like to thank anyone who shared their advice while figuring out how to actually do this :)_
