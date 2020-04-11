---
title: "Android: Improving sign-in experience with Google Sign-In and SmartLock"
author: "Pavlos-Petros Tournaris"
date: 2016-12-14T15:30:24.265Z
lastmod: 2020-04-05T01:28:33+03:00

description: ""

subtitle: "How many times have you struggled logging in on a newly downloaded app? How many times have you registered on a social network, a streaming…"

image: "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/1.jpeg"
images:
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/1.jpeg"
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/2.jpeg"
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/3.jpeg"
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/4.png"
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/5.jpeg"
  - "/images/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/6.jpeg"

aliases:
  - "/android-improving-sign-in-experience-with-google-sign-in-and-smartlock-f0bfd789602a"
---

> How many times have you struggled logging in on a newly downloaded app? How many times have you registered on a social network, a streaming service or a productivity tool and you found out, that they also provide a mobile solution?

Well personally, I can recall a lot of times, that I found myself in that position. A lot of apps provide great UI, even on their log-in screens, but UX is something more painful and as a result something more valuable, when you are trying to attract new users!Although one might think, this would be quite a hard to implement task, it is actually a lot simpler. Google already provides us with two ways of improving the sign-in experience in our apps.

> Google Sign-In (previously known as Google+ Sign-In) and SmartLock

The majority of users download an app and they want to quickly interact with it, especially if it is a productivity tool or some kind of a social network.

> What a better way of helping them by letting them sign-in in just one tap?

Almost every Android user adds his/hers Google account during the setup process and most of the time it is their primary email address, which means the email that they most probably use when they sign-up on various services. But usually, people tend to add more emails and Google accounts on their phones, for example: their work email.

This is something we can leverage in order to ease the sign-in process. To achieve this, we will use the [Google Account Login](https://developers.google.com/android/guides/setup) package,
`compile &#39;com.google.android.gms:play-services-auth:x.x.x&#39;`

from Play Services, which includes Google Sign-In API, as well as, the Credential API for SmartLock.For the needs of the article we have created a demo app which is available on [Github](https://github.com/charbgr/AuthManager) as well.

So without further ado, let’s dive in the actual implementation :)

### Google Services Configuration File

To start using Google Services we first need to create a configuration file. This process has also been quite streamlined and it is just one click in order to download it. You can find detailed instructions [here](https://developers.google.com/identity/sign-in/android/start-integrating). After downloading it, place it inside your “app” folder and you are good to go :)

### Google Sign-In

Google Sign-In, was previously known as Google+ Sign-In back in time, when Google required every new user to also create a Google+ social profile.

After Google dropped that requirement, all of their services were rebranded to plain Google, like Google Sign-In, for example.

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/1.jpeg)

As you can see at the bottom of the screenshot, this is the rebranded Google Sign-In button offered as a standalone view from Google.

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/2.jpeg)

Google Sign-In button (XML)

Now that we have added the Sign-In Button we need to configure it on our activity as well.

Google Sign-In button configuration

We also need to configure the GoogleApiClient, which will handle the Google Sign-In API and Credentials API requests:

Google API Client initial configuration

Let’s explain what these lines do:

1.  **addConnectionCallbacks →** Makes the current Activity, aware of GoogleApiClient connection lifecycle.
2.  **enableAutoManage** → Let’s GoogleApiClient “hook” on the current Activity in order to manage the connect-disconnect operations based on the Activity’s lifecycle.
3.  **addApi(Auth.GOOGLE_SIGN_IN_API, googleSignInOptions) →** Here we are declaring that we will use the Google Sign-In API, with the GoogleSignOptions we have already created.
4.  **addApi(Auth.CREDENTIALS_API)** → We will also use the Credentials API for SmartLock, so we are declaring this one as well.

We are finally ready to proceed with the normal Google Sign-In flow. First step is to **startActivityForResult** with the Sign-In Intent when tapping on the Sign-In Button:

Initiate Google Sign-In

After that we are ready to handle the result in **onActivityResult**:

Google Sign-In Resolution handling

We can now process the result of Google Sign-In in order to update our UI. Depending on the result, we either sign-in the user or present him with a sign-up screen. All of these cases are implemented in detail on the demo project accompanying this article :)

### SmartLock

SmartLock is a powerful password manager that Google provides through the same Google Account Login package in Play Services.

> But what does SmartLock offer for us as developers and our end users?

**SmartLock** allows us to:

1.  **Ask** users to **save** their credentials.
2.  **Request** those **credentials** when opening the app.
3.  **Use** credentials saved on **Chrome**, if we declare that our website and app can **share** credentials.
4.  **Display** email **hints** in case we want to help the user in the sign-in/sign-up process.
5.  Finally and most importantly, all of the above are **stored** on Google’s servers and users have complete control over what is **saved**/**deleted**.

We will cover all these cases in detail below, but if you think there might be something missing, please make sure to check the demo project on Github.#### 1) Ask users to save their credentials

First of all we check to make sure that the email address and the password, are valid for our business logic (this is a quick implementation for the purpose of the demo) and after that, we create the **Credential** object. Finally, we invoke the **Credentials API** in order to save the previously created **Credential**. Below, you can see how that is presented to a user.

A quick implementation of a Credentials save proccedure

> **Credential** is a key element of the **SmartLock** domain. It holds all the credential information (either account type or password, a name and a profile picture URI) related to an E-mail address. **Credentials** can either have an Account Type or a Password.

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/3.jpeg)

We can also see that the credentials we just saved, are available on [passwords.google.com](https://passwords.google.com) for the E-mail address we previously selected:

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/4.png)

Credential saved on [passwords.google.com](https://passwords.google.com)

#### **2) Request credentials when opening the app**

After saving the aforementioned credentials, we can now request them when opening the app in order to automatically sign a user in, or give them the ability to use them for instant sign-in.

In order to request **Credentials**, we need to create a **CredentialRequest**, that specifies what kind of **Credentials** we want to request. You can declare that the **Credentials** you want, should contain a password or their type is one of: **Google**, **Facebook**, **Twitter** etc.

SmartLock Credential Request configuration

After creating your **CredentialRequest** object, you pass it to **Credentials API** and you handle the result:

Request Credentials

Credentials Request Result handling

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/5.jpeg)

Resolve Credential Request

Start resolution for Credential Request

One thing, you should consider here, is that the **Credential** object retrieved does not have an “**email**” field. Actually the email on the **Credential** is named “**id**”. Another strange thing is also that if you have requested **Credentials** of specific Account types, you need to have in mind that they will not contain a password, due to the fact that Account type and password fields can not co-exist.

#### 3) Use Credentials saved on Chrome, if we declare that our website and app can share Credentials

User **Credentials** saved on **Chrome** can be very valuable for our case as well. Well, **SmartLock** offers **Credential** sharing between **Chrome** and **Android applications**. All we need to do is:
`- Create a Digital Asset Links JSON file (assetlinks.json)

- Upload it on our server, under &#34;/.well-known/&#34; directory`

On this [link](https://developers.google.com/identity/smartlock-passwords/android/associate-apps-and-sites) you can find detailed steps on how you can create the **Digital Asset Links** JSON file and add it on your app.

The last step, in order to enable this integration, is to [fill out](https://docs.google.com/forms/d/1UFsC7aMT5IT02PH9Hpbq0vvGJycShlUAKVp8s0QGb9w/viewform) an **Affiliation Form**, which usually takes 2 or 3 days to be accepted.

At this point, we have to thank **Steven Soneff** ([Steven Soneff](https://medium.com/u/5c13480627a2)) from **Google’s Identity** team, for his valuable help on this process.

#### 4) Display email hints in case we want to help the user in the sign-in/sign-up process

As a nice fallback when a user does not have any **Credentials** stored for our app, we can display some E-mail **hints**, in order to help the user choose an E-mail to **sign-in** or **sign-up**.

> So how can we do this?

The steps are pretty much the same for it as well :)

> Google Account Login APIs are quite identical which helps us easily bootstrap the requests.

Email Hint Request configuration

Let’s explain this code a little bit: We set the **HintRequest** to support email addresses and we also add a **HintPicker** configuration, which allows us to show a cancel button and also have a prompt as the dialog’s title. In our case we chose to show a sign-in prompt. **Google** also provides a sign-up **prompt**.

Afterwards, we need to invoke **startIntentSenderForResult**:

Request Email Hints

![image](/posts/2016-12-14_android-improving-signin-experience-with-google-signin-and-smartlock/images/6.jpeg)

And following we handle the result:

Result of Email Hint Request

The result on this case contains a **Credential** object including the email address that the user has selected:

Handling of Email Hint Request Resolution

Email Hint Request success

#### Conclusion

**Google Sign-In** and **SmartLock** possible outcomes can produce a lot of boilerplate code, as well.

To help you with this task and allow you to focus on the engineering process of your business logic, we have created a module, named **AuthManager**. **AuthManager** handles all of the cases and their outcomes, described above, while providing a fluent API. **AuthManager** is also written 100% in **Kotlin** :)

You can find AuthManager on [Github](https://github.com/charbgr/AuthManager).

Feedback and PullRequests are always welcome :)**Authors:** [Pavlos-Petros Tournaris](https://medium.com/u/d2295c5f4208), [Vasilis Charalampakis](https://medium.com/u/f985d4ea8dce)

_If you liked it, hit the “heart” button and share it with the world :) Stay tuned for more :)_
