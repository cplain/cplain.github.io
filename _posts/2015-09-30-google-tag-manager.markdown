---
layout: post
title: Google Tag Manager - Part Three
subtitle: We should totally make an app
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
---

{% assign res = "/assets/2015-09-30-google-tag-manager" %}

---

## Series Overview

&nbsp;

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then uses its own logic to construct and fire tracking events at any number of analytics tools. We will be taking a look at GTM over the next few articles, and by the end of this series we will have a working Tag Manager implementation in Android! If you are looking into GTM for iOS, the first few articles will be identical for you, it is only the client-side implementation that changes. For web, the core concepts and meanings are the same.

&nbsp;

So today, we will be looking at how to set up an Android app to talk to our container from the [previous article][Part 2].

&nbsp;

---

###Configuring the app

&nbsp;

Running GTM in an app can be extraordinarily simple, but it can also present quite a few challenges and hidden surprises depending on what you are trying to do. So best to think about all elements of your app when you start connecting in GTM. Just like any component of any codebase also make time to think about how your usages might change in the future and mitigate as many risks as you can with careful planning.

&nbsp;

####Step One: Installing the SDK

&nbsp;

Probably the easiest part of the whole process. You just need to add the following line to your **build.gradle**.

~~~
compile "com.google.android.gms:play-services-analytics:7.8.0"
~~~

If you already have the whole play services bundle compiling then don't bother with this line, you're already set up.

&nbsp;

####Step Two: Creating a TagManager

&nbsp;

Creating a TagManager is very simple, however before we rush into it, we need to think about **how** and **where** we construct our TagManager. There are two big points to consider when we create our TagManager:


- [Roboelectric][Roboelectric] and GTM **do not play nice together**

- We might only want to track application layer events now, but what if we want to track something in the network or database in future?

&nbsp;

Lets discuss the first point. What happens with a TagManager during testing? There are two big gotchas. 

1. First, if we call the TagManager from a test or execute a method that triggers the TagManager, our test will lock. Thats it, no more test. So we need to either avoid calling a real TagManager in testing or avoid those lines. Now ideally we don't want `if` statements all around our tracking code, it would be a lot nicer to simply swap out our real TagManager with a mock one. Sounds like [Dagger][Dagger] may help here.


2. The second issue is a bit more subtle. Roboelectric has issues with anything loading resources in an Application's onCreate method. This can be most commonly seen if you have [Crashlytics][Crashlytics] installed. As Crashlytics boots you will get ConcurrentModificationExceptions through your tests. GTM will give you the same problem. So not only do we want to swap our real TagManager with a mock one in tests, we actually don't want to create a real one in the first place.

&nbsp;

So now we have given it some thought our solution is very simple. We want to create our TagManager in the application class, but override that creation in the test application class and return a mock. We then want to provide our TagManager to our Dagger module which will expose it to the rest of the app.

This also solves our second point, as we can now provide that same tagging instance to any of our other modules in future. Albeit we probably want to wrap it up in an interceptor or something so as to not expose our tag manager to our networking layer directly. 

&nbsp;

So in our application's onCreate we have:

~~~
TagManager tagManager = getTagManager();
    
mComponent = DaggerGTMAppApplicationComponent.builder()
                .appModule(new AppModule(this, tagManager))
                .networkModule(new NetworkModule())
                .build();
~~~

&nbsp;

So as you can see we have bundled up all our TagManager creation in this `getTagManager()` method. Then we provide this to our module. This module can then create a `@Provides` method for our TagManager. Done! 

&nbsp;

But lets take a peek at `getTagManager()` shall we?

~~~
protected TagManager getTagManager() {
    TagManager tagManager = TagManager.getInstance(this);
    
    int id = getResources().getIdentifier(BuildConfig.GTM_BINARY_NAME, "raw", getPackageName());
    
    tagManager.loadContainerPreferFresh(BuildConfig.GTM_CONTAINER_ID, id);

    return tagManager;
}

~~~

&nbsp;

There are a few unexpected things happening here, so lets break down what is going on. First we are getting the one and only instance we will need for our TagManager using our application context. Simple enough.

Once we do this we then create an resource id variable using the not very well known `getIdentifier()` method. This method allows us to get a resource id from the name and type of the resource we are looking for. But this leads to two questions:

1. What raw file are we fetching?

2. Why are we doing it this way and not just using its id?

&nbsp;

Question one is very simple, we are fetching our container binary we downloaded earlier on the **Versions tab**. This is where we set it as our default container file.

Question two is also pretty simple. Earlier when I described what our **GTMApp** would look like, I said we would have a number of different flavours pointing to different environments. So it would make a lot of sense that I would like a different GTM container for my different environments, perhaps a testing one a development one and a production one. Either way I want this as configurable as possible. The `getIdentifier()` method is a handy way to tie a resource to a `buildConfigField` in Gradle.

Once we do this we load our default container using another BuildConfig field. This time it is the GTM container id. The very first highlighted field in our top bar. Do you remember?

&nbsp;

![Memory Lane]({{res}}/container_top.png)

&nbsp;

So these two values will be sitting in our build.gradle like this:

~~~
prod {
   ...
   buildConfigField "String", "GTM_CONTAINER_ID", "GTM-MZRKNH"
   buildConfigField "String", "GTM_BINARY_NAME", "gtm_mzrknh_v1"
   ...
}
~~~

&nbsp;

Note also the format of our binary file name. GTM provides the file as "GTM-MZRKNH_v1" this doesn't play very nicely with Android so we have to rename it to something more palatable.

&nbsp;

Now what would this method look like in our TestGTMApp?

~~~
@Override
protected TagManager getTagManager() {
    return Mockito.mock(TagManager.class, Mockito.RETURNS_MOCKS);
}
~~~
&nbsp;

Quite a lot simpler. We are just returning a fake TagManager that will not give us errors when used. Note also we use the `RETURNS_MOCKS` option. This is because when we go to use our TagManager later we will need to access some objects within the TagManager that we also want mocked.

&nbsp;

####Step Three: Using the TagManager

&nbsp;

The last step where we have to do something! Once this is set up we can sit back and watch the analytics data roll in. There will be two tracking methods we want to create. One for page views and one for clicks. Using an MVP pattern we can put these in our base presenter class. Our presenters should be aware of every single action that happens in the app so it makes a lot of sense for our methods to live there.

&nbsp;

Lets check out our page tracking method.

~~~
@Inject TagManager mTagManager;

protected abstract String getName();

protected final void trackPage() {
    String name = getName();
    mTagManager.getDataLayer().pushEvent("pageView", DataLayer.mapOf("screenName", name));
}
~~~
&nbsp;

So what is going on here? Not much at all really. We are fetching the name for our page via `getName()` which will be implemented by all subclasses of our base presenter (Ensuring we have a unique name). We are then fetching the data layer of our TagManager and pushing through a "pageView" event. 

We construct our key-value pairs via the DataLayer's `mapOf` utility. This utility takes Strings and formats them into a HashMap. With pos 0 being a key for pos 1, 2 for 3 and so on. It will throw an error if the number of Strings provided is uneven or otherwise error worthy. For a lot of developers this might make you a little uncomfortable, especially if you have a look at the internals. In that case feel free to make your own solution (perhaps a builder style pattern) as long as you return a HashMap<String, String> in the end. For our examples we will stay with the default.

Note also this method is final. This is so our subclasses cant accidentally break our tracking.

That is it! Now whenever our presenter is created we can call `trackPage()` and we are done. However we may want two more things. 

1. We will want to move those hardcoded values to a constants file

2. What if we have a presenter subclass we don't want to track? Maybe a routing page that is invisible to a user and the business never needs to know exists? We can't have the call in our base create method if that is the case, we need to call it everywhere it is needed! Over and over again! Or do we?

&nbsp;

~~~
@Inject TagManager mTagManager;

protected abstract String getName();

protected final void trackPage() {
    if (shouldTrackPage()) {
        mTagManager.getDataLayer().pushEvent(GTM_EVENT_PAGE_VIEW, DataLayer.mapOf(GTM_KEY_SCREEN_NAME, getName()));
    }
}

protected boolean shouldTrackPage() {
    return true;
}
~~~

&nbsp;

As you can see here I have made some constants and also condensed our lines a bit. But what I have also done is introduce a `shouldTrackPage()` method which returns true by default. This means for our 90% of trackable presenters they will work by virtue of existing. In the other 10% we can override this method and return false, preventing the tracking from occurring. Far neater and also far simpler. 

Note they sill need a name, I have found this is useful for debugging, and also ensures any new presenters will not use a default value by accident.

&nbsp;

Now lets look at click tracking:

~~~
// Everything we had above

protected final void trackClick(String clickTarget, String clickValue) {
    mTagManager.getDataLayer().pushEvent("click", DataLayer.mapOf(
            "screenName", getName(),
            "clickTarget", clickTarget,
            "clickValue", clickValue
    ));
}
~~~

&nbsp;

What we see is very similar to our page view above. Note again I have left out the constants so we can see what is happening. This time we would call the `trackClick()` method when a click event has occurred. We pass in the name of the button or clickable element (similar to a page name we define this ourselves) and a value such as a web address for a link, a state for a checkbox and so on. However there is one thing we have overlooked. 90% of cases will just be a button that has been clicked. We don't want to be passing the same value in over and over do we? 

&nbsp;

Lets add in another method.

~~~
// Everything we had above

protected final void trackClick(String clickTarget) {
    trackClick(clickTarget, GTM_KEY_CLICK_VALUE_DEFAULT);
}

protected final void trackClick(String clickTarget, String clickValue) {
    mTagManager.getDataLayer().pushEvent(GTM_EVENT_CLICK, DataLayer.mapOf(
            GTM_KEY_SCREEN_NAME, getName(),
            GTM_KEY_CLICK_TARGET, clickTarget,
            GTM_KEY_CLICK_VALUE, clickValue
    ));
}
~~~
&nbsp;

This time we have two methods, one doing the tracking and one providing a default `clickValue` in the case we don't have one. Much more clean, now we can call either method depending on if we have a value to pass or not.

&nbsp;

####Step Four: Reap the rewards

&nbsp;

Lets check in on our GA account shall we?

&nbsp;

![Results 1]({{res}}/results_1.png)

&nbsp;

Look at that! We have all our page views coming in real-time! Up the top we can see how many users are active right now (funnily enough just me). There are some graphs to show relative usages, the leftmost is over the last 30 minutes and the right is over the last minute. As items enter and exit the table underneath will update, if a row flashes red it is decreasing, if its green it is increasing. 

Also note we are looking at the **Screen Views (Last 30 mins)** option here. This is just so I can show you a few pages. Otherwise we would just see whatever my **GTMApp** is currently showing, which isn't as nice. 

Alright lets check out **Events**.

&nbsp;

![Results 2]({{res}}/results_2.png)

&nbsp;

Everything we want is here too! The view is very similar. We can't see our label data however. This is actually just how GA lays out its real-time section. To see the labels we need to filter on our User Action category by clicking on it.

&nbsp;

![Results 3]({{res}}/results_3.png)

&nbsp;

And there you have it! Everything hooked up. And with remarkably little effort. What is even better you can clone a container by going to **Admin > Export Container** in GTM and then import it in another account via **Admin > Import Container**. So you only need to do this once for the most part!

&nbsp;

![Import 1]({{res}}/import_1.png)

&nbsp;

---

## Summary

&nbsp;

So with very little code and very little effort we have a powerful, flexible analytics solution. What more could you want? Well surprisingly GTM can provide us with a lot more. There is a trove of value to be discovered in its advanced options and power usages.

But hopefully for now this series has you equipped to handle the basics and use this new insight to better your products and brands. And as you push ahead with this new platform you can grow your skills and step bravely into deeper waters!

Good luck

[Dagger]: http://google.github.io/dagger/
[Crashlytics]: https://fabric.io/kits/android/crashlytics/summary
[Part 2]: {% post_url 2015-09-25-google-tag-manager %}