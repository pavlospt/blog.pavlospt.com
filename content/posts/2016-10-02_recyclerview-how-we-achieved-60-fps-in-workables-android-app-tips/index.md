---
title: "RecyclerView: How we achieved 60 FPS in Workable’s Android App (tips)"
author: "Pavlos-Petros Tournaris"
date: 2016-10-02T17:03:40.701Z
lastmod: 2020-04-05T01:28:27+03:00

description: ""

subtitle: "Most of us, are using RecyclerView to present data to our users, in the form of a list. It is a common thing as well that a RecyclerView…"

image: "/images/2016-10-02_recyclerview-how-we-achieved-60-fps-in-workables-android-app-tips/1.png"
images:
  - "/images/2016-10-02_recyclerview-how-we-achieved-60-fps-in-workables-android-app-tips/1.png"
  - "/images/2016-10-02_recyclerview-how-we-achieved-60-fps-in-workables-android-app-tips/2.png"

aliases:
  - "/recyclerview-how-we-achieved-60-fps-tips-in-workables-android-app-recyclerviews-c646c796473c"
---

Most of us, are using RecyclerView to present data to our users, in the form of a list. It is a common thing as well that a RecyclerView can draw multiple layouts on its rows, which pretty much means different XML layouts, different things to allocate memory for and sometimes tricky parts where things get so messy that we have frame drops.[Workable’s](https://www.workable.com/) Android application, is one of those applications that need to present data in the form of a list. At Workable, the core ingredient of our lists, are candidates. Candidates though, are not just some simple POJOs. Candidates consist of a lot of defining characteristics, that we need to consider before drawing a Candidate row on a RecyclerView.

We also use DataBinding, which has made our lives easier, BUT it has its pitfalls if you are not very careful on what you use it for and how you use it. Having said that, let’s jump on the main part.So we started with the famous, but not friendly, Allocation Tracker, included in Android Studio. Some scroll ups and scroll downs and we had a considerable sample to work on.

Analyzing the Tracker’s report it was pretty obvious that the TableLayout we were using for a part of the Candidate’s layout was consuming too many resources. We used TableLayout to overcome some technical difficulties we had, when designing the aforementioned part. But as you might already know for every problem in layouts there is always a solution. So, LinearLayout to the rescue. Using LinearLayout and its weight factors efficiently, let us overcome those design issues and free ourselves from the resource-demanding TableLayout.

Candidate part with TableLayout before

And the result when using LinearLayout.

Candidate Part after switching to LinearLayout

It might not be such a huge difference, but based on the allocations that we saved this alone might have saved us a frame or two.

Another tricky part, that was causing a lot of allocations, was that one of the things that characterise a Candidate had to be drawn in capital letters. What easier way than setting an XML flag for it on the TextView. After all though, that did not seem to be such a good idea. Our approach on this, was to just use Java’s String transformation **.toUpperCase()**.

textAllCaps attribute before

Use of .toUpperCase() after

Here is also a screenshot of the generated allocations from TextView’s “textAllCaps” attribute.

![image](/posts/2016-10-02_recyclerview-how-we-achieved-60-fps-in-workables-android-app-tips/images/1.png)

AllCapsTransformationMethod allocations

You can see here that internally the TextView is generating a new Transformation method each time:

![image](/posts/2016-10-02_recyclerview-how-we-achieved-60-fps-in-workables-android-app-tips/images/2.png)

That is probably fine in a common scenario where we just use it in a static layout, but when we have to deal with scrolling, it can create some overhead.

It might seem a little thing as well, but it really played its part on the overall optimization.So now it was the time to utilize RecyclerView’s **.onViewRecycled()** method. What this method does, is that it let us know when a row in RecyclerView has been recycled, so that we can load-off some not needed resources. As we mentioned before, we are using DataBinding. That meant, it is the appropriate time to remove **OnPropertyChangedCallbacks** from our ViewModel and then clear the ViewModel itself from the binding. We could also clear the ImageView that was holding the Candidate’s Avatar, which we have previously loaded using Glide.

In order to have a great and smooth experience in our RecyclerViews, we have also used some caching tricks directly on the RecyclerView. We then went on and measured the FPS with the current situation. The FPS Meter was constantly showing 60 FPS :) We have reached our goal, but we could not stop there. We went forward and remove the Caching tricks from the RecyclerView.

Caching tricks

Then we re-run our tests to see how the situation was holding. To our great surprise we were still at 60 FPS!! No matter what they say, that felt really great :)

The final result

To conclude, Allocation Tracker has been pretty helpful for us and our Lists :) Also no matter how big or small an optimization is, it can always contribute to the greater optimization of your application :)Enjoy and stay tuned for more :) !!
