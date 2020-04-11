---
title: "Android MVI architecture with Jetpack & Coroutines/Flow — Part 1 "
author: "Pavlos-Petros Tournaris"
date: 2020-03-29T21:07:42.403Z
lastmod: 2020-04-05T01:29:05+03:00

description: ""

subtitle: "RecyclerView Adapter w/ ViewBinding"

image: "/posts/2020-03-29_android-mvi-architecture-with-jetpack-coroutinesflowpart-1recyclerview-adapter-w/images/1.jpeg" 
images:
 - "/posts/2020-03-29_android-mvi-architecture-with-jetpack-coroutinesflowpart-1recyclerview-adapter-w/images/1.jpeg" 

aliases:
    - "/android-mvi-architecture-with-jetpack-coroutines-flow-part-1-recyclerview-adapter-w-83a10134207f"
---

RecyclerView Adapter with ViewBinding
==================

![image](/posts/2020-03-29_android-mvi-architecture-with-jetpack-coroutinesflowpart-1recyclerview-adapter-w/images/1.jpeg)

Photo by [Mike Lewis HeadSmart Media](https://unsplash.com/@mikeanywhere?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/flow?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

*I have been developing on Android since late 2011 and always remember that we wanted, since the early days, Google to be opinionated about the suggested architecture on Android :)*

This indeed happened a few years later, with Jetpack’s Android Architecture Components, which introduced **ViewModels**. **ViewModels** is the **VM** part of a **MVVM** architecture and can also survive configuration changes on screens. ViewModels communicate with the UI through an observable stream implemented by **LiveData**.

The thing though, is that the community also needed a new way to make async API calls and Database operations (or generally off-main thread work) easier, without the learning curve that the RxJava has for most of beginners on Reactive frameworks. The alternatives to RxJava were also quite boilerplate-y and more prone to be used incorrectly from people that are not good on primitive threading &amp; concurrency, like me 😝!

Enter Kotlin Coroutines &amp; Flow
==================

The community though had started recently been switching to Kotlin Coroutines, which offer structured concurrency and Kotlin Flow that can essentially replace the RxJava way we used to work, in order to transform the data between our architecture’s layers, to a stream. That could be either a bi-directional or a uni-directional fashion, depending on the needs of each project.

What we are going to cover
==================

In this series of articles we are going to see how we created a new app that fetches Github repositories of a Github user, persists them in the local storage and observes the local storage only, in order to display any UI changes. We will also see how we can create a RecyclerView ListAdapter with the help of newly released ViewBinding, in order to also leverage the diffing functionality for free.

In order to achieve all of the above we will make use of Jetpack’s Android Architecture Components and also Kotlin Coroutines + Flow for the parts that we will need to do asynchronous work on any of our layers, either that is a remote API source, or our local storage which will be backed by Room.

Last but not least, we will see how we modularized the application and configured our Kotlin style &amp; static code analysis tools for a multi-module project (Detekt + Spotless). At the end we will see how to leverage all of this configuration in order to create a seamless CI pipeline for our newly created app.### Creating a base RecyclerView adapter with ViewBinding

In order to create a base adapter for all of our RecyclerViews in the app, we decided to make use of the newly released ViewBinding as well and use ListAdapter’s functionality to implement diffing quite easily.

Creating a base adapter item
==================

By doing this we will be able to have a single interface to implement and use as common configuration for our adapter’s items.

{{< gist pavlospt 222da13e4afd52ee26fa42a50b1f05ba >}}

This item only includes an `itemViewType` property so that we can have polymorphic RecyclerView adapters. We only need this property for now, but we can surely add more in order to enrich our base functionality in the future.

Creating a base ViewHolder
==================

Our base ViewHolder will be extended from each ViewHolder in each of our adapters.

{{< gist pavlospt 45c9957f9c95023e7b4f6d1c4329a1cf >}}

What we do here is create an abstract class and use generics to utilize the base adapter item we have created above and the ViewBinding for the item’s layout.

The `bind` method for our ViewHolder is abstract since we require every new ViewHolder extending that one to implement this method.

The other available method is `bind` with a List of payloads that can help us partially update our rows in the RecyclerView. Since we are overriding both of them on the base adapter (see below) we need to make that method `open` and provide a default implementation that calls `abstract bind(item: Item)` in order to ensure that we fallback correctly when the diffing operation does not output any results (see below).

Creating a base adapter
==================

Our base adapter will extend from `ListAdapter` since we want to use `DiffUtil` for diffing in our RecyclerViews.

{{< gist pavlospt 0954329d5885b899594a3182c55998d6 >}}

The implementation we have here just delegates the `onBindViewHolder` methods to our own ViewHolders. We also provide an extension function for easier access to the layout inflater when creating a new adapter.#### Creating a base DiffUtil callback

Our base DiffUtil callback will extend from `DiffUtil.ItemCallback` and use `ViewBindingAdapterItem` as a generic input.

{{< gist pavlospt 775c0b0c8e7795a968d17222b86e6274 >}}

Conclusion
==========

We can now create as many adapters as we want with minimal effort by just extending the already created base components :)

You can find all of the code above and examples of their usage to the following repo, which will be the one we will cover in this series of articles!

[pavlospt/refactored-umbrella](https://github.com/pavlospt/refactored-umbrella)
