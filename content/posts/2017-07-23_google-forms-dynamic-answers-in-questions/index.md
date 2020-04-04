---
title: "Google Forms: Dynamic answers in Questions"
author: "Pavlos-Petros Tournaris"
date: 2017-07-23T10:57:29.573Z
lastmod: 2020-04-05T01:28:44+03:00

description: ""

subtitle: "Android Community in Greece has its own Slack team :) New members that want to join our Slack team, fill out an invitation form and then…"

image: "/posts/2017-07-23_google-forms-dynamic-answers-in-questions/images/1.png" 
images:
 - "/posts/2017-07-23_google-forms-dynamic-answers-in-questions/images/1.png" 


aliases:
    - "/google-forms-dynamic-answers-in-questions-9e66ba9b3ead"
---

![image](/posts/2017-07-23_google-forms-dynamic-answers-in-questions/images/1.png)

Android Community in Greece has its own Slack team :) New members that want to join our Slack team, fill out an invitation form and then administrators invite them over to the team!

But the form currently was asking for quite a little of info that would let us understand and analyze the needs of the people joining our team.

We ended up discussing it with the Admin team and decided to actually add a multi-select question with all the channels of the Slack team, in order to better understand why new people want to join the team.### The Form

So well, we already had a Google Form that we currently use. Someone would think that we could add all of the channels on the form editor. But, if you think of it in the future it might probably not scale at all, due to maintenance issues caused by a large number of Slack channels.
> We needed a more elegant, more dynamic and more scalable solution

To our great luck, Google provides us with 2 key tools for this job.

1.  Google App Scripts
2.  Google Form Triggers

Let’s go into a little detail on the above:

#### Google App Scripts

Google App Scripts are scripts written in GS that act on a Google App, like a Google Spreadsheet or in our case a Google Form. They have their own Domain model and they provide access, in order to programmatically edit data from a Spreadsheet or make complex operations upon the already filled in data.

#### Google Form Triggers

Google Form Triggers are essentially “hooks” on the main actions on a Google App. For example, on a Google Form there are 2 main hooks. These are: **“onOpen”** and **“onSubmit”**. Adding a function on a **.gs** file and calling this function in **“onOpen”** trigger, will cause the function to be executed every time the form opens.For the second part, we needed to **dynamically** load Slack channels as **CheckBoxItem** answers. In order to do that we leveraged Slack’s API for channel listing. We also needed to load these answers every time someone opens the Form, so as to ensure the validity of data.

This is a classic use case for the **“onOpen”** trigger. What is needed now is to create the function to be triggered from **“onOpen”**:
`function onOpen () {  
  var form = FormApp.openById(FORM_ID)  
  var requestResponse = UrlFetchApp.fetch(SLACK_API_CHANNELS_URL)  
}`

In order to load data from the Slack API, we used the “UrlFetchApp” inside the Google App Script, which essentially is a Network client for remote API calls. We also created a var pointing to our already created Google Form, by its **“id”**.

Next step, is to parse the channels listing response and prepare the **CheckBoxItem** dynamic answers:
`function onOpen () {  
  var form = FormApp.openById(FORM_ID)  
  var requestResponse = UrlFetchApp.fetch(SLACK_API_CHANNELS_URL)  
  var parsedJsonResponse = JSON.parse(requestResponse.getContentText())  
  var isSlackResponseOk = parsedJsonResponse.ok``  if (!isSlackResponseOk) return``  var eligibleChannels = prepareEligibleChannels(parsedJsonResponse)``  var checkBoxItem = sanitizedCheckBoxItem(form)``  eligibleChannels = eligibleChannels.map(function (channel) {  
    return checkBoxItem.createChoice(channel.name)  
  })``  checkBoxItem  
    .setTitle(&#39;Select the channels you will probably join:&#39;)  
    .setChoices(eligibleChannels)  
}`

What the code above does, is to check if the response from Slack’s API is **“ok”** and then use a method to **reduce** and keep only the channels we are interested in:
`function prepareEligibleChannels (parsedJsonResponse) {  
  return parsedJsonResponse.channels.reduce(function  (filteredChannels, channel) {  
    if (isChannelEligible(channel)) {  
      filteredChannels.push({  
        name: channel.name,  
        topic: channel.topic.value  
      })  
    }  
    return filteredChannels  
  }, [])  
}``function isChannelEligible (channel) {  
  return !channel.is_general  
    &amp;&amp; !channel.is_archived  
    &amp;&amp; !channel.is_private  
    &amp;&amp; channel.is_channel  
    &amp;&amp; !isIgnoredChannel(channel)  
}``function isIgnoredChannel (channel) {  
  return IGNORED_CHANNELS.indexOf(channel.name) &gt; -1  
}`

Now that we have the appropriate data needed to create our answers for the **CheckBoxItem**, we need to get a reference on the single Item in our Google Form. This could already be a **CheckBoxItem** (or something else) or might not even exist at all. That leads us to cater for 2 specific cases:

1.  The Item already exists and/or it is of a different type, so we need to delete it and create a new one in its place.
2.  The Item does not exist at all and we need to create a new one.

Let’s see how we achieved that using the Domain model of the Google Forms:
`function sanitizedCheckBoxItem (form) {  
  var checkBoxItem = findFirstItem(form)``  if (checkBoxItem != null) {  
    form.deleteItem(checkBoxItem)  
  }``  checkBoxItem = createCheckBoxItem(form)``  return checkBoxItem  
}``function findFirstItem (form) {  
  return form.getItems()[0]  
}``function createCheckBoxItem (form) {  
  return form.addCheckboxItem()  
}`> The code above makes the assumption that we will only have one Item in our Google Form. If this is not the case we have to alter the code to find the appropriate Item and manipulate it accordingly.

### Conclusion

Apparently, for every crazy idea or need out there, seems that there is always a solution. All it needs is some spare time and some engineering curiousness and you can certainly build everything :)

You can also find all of the above code in a PR on our Slack’s team Github repository: [https://github.com/androiddevs-gr/slack-tools/pull/4](https://github.com/androiddevs-gr/slack-tools/pull/4)
