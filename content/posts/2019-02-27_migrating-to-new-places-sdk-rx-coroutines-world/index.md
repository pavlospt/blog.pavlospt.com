---
title: "Migrating to new Places SDK: Rx & Coroutines world"
author: "Pavlos-Petros Tournaris"
date: 2019-02-27T13:10:51.930Z
lastmod: 2020-04-05T01:28:55+03:00

description: ""

subtitle: "Context"

image: "/posts/2019-02-27_migrating-to-new-places-sdk-rx-coroutines-world/images/1.png" 
images:
 - "/posts/2019-02-27_migrating-to-new-places-sdk-rx-coroutines-world/images/1.png" 
 - "/posts/2019-02-27_migrating-to-new-places-sdk-rx-coroutines-world/images/2.png" 


aliases:
    - "/migrating-to-new-places-sdk-rx-coroutines-world-9adcedef20c"
---

![image](/posts/2019-02-27_migrating-to-new-places-sdk-rx-coroutines-world/images/1.png)

Taken from [https://cloud.google.com/blog/products/maps-platform/introducing-new-improved-places-sdks](https://cloud.google.com/blog/products/maps-platform/introducing-new-improved-places-sdks) Courtesy of Google.

### Context

Up until **January 29, 2019** all Places functionality was part of Play Services. Since that day Google announced new **Places SDKs** for both **Android** &amp; **iOS** platforms. ([https://cloud.google.com/blog/products/maps-platform/introducing-new-improved-places-sdks](https://cloud.google.com/blog/products/maps-platform/introducing-new-improved-places-sdks)). The deadline for the migration to new Places SDK, is **July 29, 2019**.

The new Places SDK brings in new methods replacing the previous ones, so this should be considered as a breaking change if you want to follow that path. However, there is a compat library released as well, which delegates all the calls on Location library from Play Services, to Places client.

In this article we will explore the migration to the new Places SDK on Android and provide helper examples for **RxJava** and **Coroutines**.

### Changed methods



![image](/posts/2019-02-27_migrating-to-new-places-sdk-rx-coroutines-world/images/2.png)

As you can see, the above methods follow some common patterns:

1.  `get` verb has been removed from the functions’ names.
2.  All methods have a common pattern for their type of argument and type of response. `nameOfMethod.capitalize() + Request/Response` This makes it quite easy to be able to construct your request data, just by thinking what method you need to invoke.
3.  All the above methods are being invoked on an instance of `PlacesClient` which you can easily create by just calling: `Places.createClient(context)`### Entering the Rx world

#### Issues

Previously we were using a wrapper library above the old Play Services Location library, which exposed all the functionality to a reactive fashion by converting a **PendingResult** to an **Observable**.

Although, this library has not received any updates since 2017 and it had also not been migrated to the newest style of Location library, which uses **Task** instead of **PendingResult.**

#### Initial approach

Our initial approach was to fork the library and migrate it to use the newest Places SDK, but after the first proof-of-concept this turned to be a huge task and also quite bloated, since the parts we needed could be abstracted in something simpler (we will see below).

#### Solution

Since the burden of migrating the whole library was too much, we decided to convert the callback-style **Task** API, to a reactive way that would fit in our codebase and would produce the minimum changes needed.

The solution was pretty simple. All we needed to do is create a new class that would extend **ObservableOnSubscribe.**


TaskObservable.kt



This allowed us to create a simple extension function (we are using Kotlin, but this would be pretty trivial task in a Java codebase as well), to convert any **Task** to an **Observable**.
`fun &lt;T&gt; Task&lt;T&gt;.toObservable(): Observable&lt;T&gt; = Observable.create(TaskObservable(this))`

The last step was to remove the library we were using and replace the usage sites with our solution.

We should also notice here, that previously we were using the library as-is. After this change and since this is always a subject to change, the legacy part needed to be hidden behind an interface. This will save us time in the future, in case there are more changes to come and will make our testing easier.### It’s 2019, who cares about Rx 😝

You are probably right! A lot of people are converting parts of their Rx codebase or writing new things, that require asynchronous programming, with **Coroutines**.

Also that was a chance for us to get our hands dirty and try-out Coroutines and how we could achieve something similar to the Rx result.

The solution here was pretty simple as well. We only needed to convert the **Task** callback-style API to a **suspend fun.**


SuspendedTask.kt



Once again, we added an extension function doing the work described previously. For the sake of simplicity we are only handling the **success** and **failure** cases here, but you might want to cover more cases, like cancelled tasks etc.### **Conclusion**

The migration to the new Places SDK seemed to be a lot harder initially, but with a clear planning and finding what can be abstracted away, you can save a lot of time.

The article’s purpose is to help you make that migration easier and effortless. I did not want to dive into specific changes of the new SDK. You can find those detailed here: [https://developers.google.com/places/android-sdk/client-migration](https://developers.google.com/places/android-sdk/client-migration)

Hope you enjoyed it!
