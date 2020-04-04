---
title: "API Design: Designing for errors, actually matters!"
author: "Pavlos-Petros Tournaris"
date: 2017-02-25T22:52:11.571Z
lastmod: 2020-04-05T01:28:37+03:00

description: ""

subtitle: "Software engineering (SWE) is the application of engineering to the development of software in a systematic method."

image: "/posts/2017-02-25_api-design-designing-for-errors-actually-matters/images/1.jpeg" 
images:
 - "/posts/2017-02-25_api-design-designing-for-errors-actually-matters/images/1.jpeg" 


aliases:
    - "/api-design-why-designing-for-errors-actually-matters-15d850c686ad"
---

![image](/posts/2017-02-25_api-design-designing-for-errors-actually-matters/images/1.jpeg)

> **Software engineering** (**SWE**) is the application of [engineering](https://en.wikipedia.org/wiki/Engineering) to the [development](https://en.wikipedia.org/wiki/Software_development) of [software](https://en.wikipedia.org/wiki/Software) in a systematic method.

As **Software Engineers**, even more those of us who have to deal with Front-End Software Engineering, we deal every day with loads of errors. We can pretend that they do not exist… but, if we really care about the **UX** of our app, we **_have to_** deal with them.Dealing with errors though, is not just something that has an effect on the end result and what a user sees. Errors can also occur when consuming a third-party API, using a Library or even those generated by our Business Logic.All of the above categories, though, have something in common. We need and have to be able to distinct errors and their cause. In order to do this, we have to think that **_the code we are writing is going to be consumed by someone else_**. If we fail to train our brain to code with this mindset, we are prone to end up in very weird error situations._Enough with the talking though, let’s dive in a real world example._#### First occurrence

So, the other day, we decided to add **Awareness API** in our project. What we needed to use from the Awareness API, was the **DetectedActivity** feature. Pretty straightforward integration like any other Play Service, provided from Google.

Then, it was the time to request the _detected user activity_, from the **Snapshot API**. We ended up with an error, saying:

`Status Code: 7503, Status: unknown, Resolution: null`

We thought, it was ok, just another common first-time run on a newly added integration. We were pretty wrong about it, though!

Awareness API docs were referring to that status code, as:

`ACL_ACCESS_DENIED`

But what would that even mean? I mean ok, we know what `ACL` means but why would we receive such an error?

We double-checked our API keys, as we thought that an **Access Control List** related error, probably refers to a Disabled Awareness API from the **Google API Console**. That was not the case though, we kept getting that error. After some hours of debugging, we ended up finding that we missed a part of the integration guide. The part where you are told, what permissions you will need on your **AndroidManifest**, for the Awareness API features to work.

That was it, `ACL_ACCESS_DENIED` error, was referring to a missing Permission. (_Take a note on this, we will need it.)_#### Second occurrence

Time went by and I happened to need Awareness API on another project. Started off the same way like last time, but this time I knew I only needed to use Location feature. So, first things first:

1.  Enable the Awareness API
2.  Get that API key and declare it on the application tag as meta-data
3.  Add the `ACCESS_FINE_LOCATION` permission (lesson learnt)

But, hey, the app and specifically, the Awareness API**,** had different plans. Guess who was back: our dear friend `ACL_ACCESS_DENIED` .

Re-checked everything and made sure that API keys are correct, that I was not missing a Permission on the Manifest and that I was actually granting a runtime Permission for Location. What was the cause of the `ACL_ACCESS_DENIED` error then?

Started Googling around and ended up finding that the only reported issues and solutions, were the exact same I found last time. So, no apparent reason that I could think of, this time...

Spent a lot of hours debugging and trying different things, none of them seemed to be working. _Then it hit me_.
> What if I have the Location setting off?

I pulled down the Notification Center and there it was. The Location was **OFF.** What If I turned it **ON** and retry? Location was ON now. App starts and **_voila_**_:_ I got my result finally.> But why are you telling us, all of these, just to prove that you are forgetting to do the obvious thing first?Let me remind you, we are **Software Engineers** and we deal with a lot of context switching on a day, but most of all we deal with a lot of problem solving. We are trained to spot such things in advance, but we are humans above all and we are doomed to make mistakes from time-to-time.What I described above could have been resolved in just a second, if Google Android Engineers had designed the errors of Awareness API from the perspective of the consumer.

I guess, it is not just me that finds it somewhat strange, that a missing Permission from Manifest and a Setting turned OFF, map to the exact same error. **Business wise** it probably DOES mean the same, but when you are integrating such a **black-box API**, you know nothing about the business logic.**Conclusion**

As Software Engineers, it is our duty to hide our **business logic** behind **self-explanatory errors,** to the consumers of our code/API/Library/you_name_it. A simple change of mindset, could potentially save a lot of hours for a colleague out there.**TL;DR**
> A simple change of mindset can help us be better at how we code and how we design our APIs for the wild :D