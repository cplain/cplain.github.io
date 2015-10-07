---
layout: post
title: Google Tag Manager - Bonus Round
subtitle: Some extra goodies for our bags
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
res: /assets/2015-10-06-google-tag-manager
---

---

[So we have GTM up and running!][Part 3] But now what? There must be more right? This article will take a look at some additional features we can use in GTM (again focusing on an Android client). There is no way I can cover everything, so I have picked out three interesting capabilities for us to look at.

<!--end_excerpt-->

&nbsp;

##Series Overview

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then use its own logic to construct and fire tracking events at any number of analytics tools. We will be taking a look at GTM over the next few articles, and by the end of this series we will have a working Tag Manager implementation for Android! If you are looking into GTM for iOS, the first few articles will be identical for you, it is only the client-side implementation that changes. For web, the core concepts and meanings are the same.

&nbsp;

---

##Extra Data

Typical GA events allow for a category, action, label and value, but sometimes this just isnt enough. GA has a wealth of additional dimensions that can add a lot of valuable information into the mix. Conveniently a lot of these are set for us right out of the box! However not as conveniently the majority aren't. Wouldn't it be great if we could simply set whatever ones we need when we fire our tags? Well as it happens, we can. This is probably one of the quickest enhancements you can make to your tracking and could be a lifesaver when it comes to analysing data and making decisions. You don't want to be in a position where you have a question but no data to answer it.

&nbsp;

### How to do it

First thing's first, we need to send through the data from the client. This will look very familiar to anyone who has been following the series - we are just adding additional data layer variables.

&nbsp;

Remember our click tracking method from [part 3?][Part 3]

{% highlight java %}
protected final void trackClick(String clickTarget, String clickValue) {
    mTagManager.getDataLayer().pushEvent(GTM_EVENT_CLICK, DataLayer.mapOf(
            GTM_KEY_CLICK_TARGET, clickTarget,
            GTM_KEY_CLICK_VALUE, clickValue
    ));
}
{% endhighlight %}

&nbsp;

This has been a very useful method for us so far. But there is one big flaw. What if we had two buttons with the same logical name? What if they looked and acted the same but were on different pages? How would we know which event belonged to each element? Well... We wouldn't. There isn't currently a way for us to distinguish between the two. 

So how about we solve that problem? We know GA has a `screenName` dimension because we used it for our page tracking. It would be great if we could set it in our tag, but how would we accomplish something like that?

&nbsp;

Well let's start with a little change to the client...

{% highlight java %}
protected final void trackClick(String clickTarget, String clickValue) {
    mTagManager.getDataLayer().pushEvent(GTM_EVENT_CLICK, DataLayer.mapOf(
            GTM_KEY_SCREEN_NAME, getName(),
            GTM_KEY_CLICK_TARGET, clickTarget,
            GTM_KEY_CLICK_VALUE, clickValue
    ));
}
{% endhighlight %}

&nbsp;

Nothing unexpected there at all. We are sending through our page name using the `getName()` method [we set up last time][Part 3] and that's it. The client has done all the work it needs to!

&nbsp;

All we need now is to receive our new field in GTM and find a way to pass it through to GA. That normally would mean its time for a new variable, but in this case our variable was previously set up. Very convenient for us!

&nbsp;

![Extra Data 1]({{page.res}}/extra_data_1.png)

&nbsp;

Finally we send our newly populated variable through to GA in our tag. 

&nbsp;

![Extra Data 2]({{page.res}}/extra_data_2.png)

&nbsp;

Under the four key values we can see "More Settings", and in here we can set our field! We could set any GA dimension we wanted as long as we know the name for it. 

&nbsp;


> Note: Also further down we can see a place for custom dimensions, these work in a similar fashion but you are limited to a set number. They also do not have a text-based key, you will have to use an index.

&nbsp;

So our click tag now looks like this:

&nbsp;

![Extra Data 3]({{page.res}}/extra_data_3.png)

&nbsp;

And we are done! We can now see the screen name for each click in GA by adding the `Screen Name` secondary dimension from the top dropdown in our User Action view.

&nbsp;

![Extra Data 4]({{page.res}}/extra_data_4.png)

&nbsp;

> Note: in the case of screen name, the real-time value is different from the value you will see in your actual metrics. This is because the real-time value is actually the current screen the owner of that hit is on. This is unfortunate as it means you need to wait 24 hours until you can see your new values coming through.

&nbsp;

---

##Campaign Tracking - Installs

The majority of developers will have the luxury of knowing that every user installs their production app from the Google Play Store. But there are multiple ways a user could arrive to that install page. They could have clicked on a link in a mailer you set out, they could have followed an ad or maybe they found it through your website. 

Often we will want to know where our users came from. Quite obviously this information lets us know how our different advertising streams are performing, but this is just the beginning. When combined with segments it can give us insights into how users coming from different mechanisms behave. We can look at their expectations and identify their unique pain points. Perhaps a particular ad draws in a demographic entirely different to your original user-base, what sold the app to them? What are they looking for? How can you capitalise on that? Knowing where they came from is the first step in opening up these opportunities.

Thankfully install referrer tracking is very easy to implement in GTM. So easy in fact that I'm going to take this chance to cover a common problem when working in this space as well.

&nbsp;

### How to do it

When a user opens your app for the first time, Google will send an `Intent` that contains all the referrer information. We simply need to receive this information and send it through to GTM. GTM makes this as simple as possible by providing a receiver for us! We just need to add the following in our AndroidManifest.

&nbsp;

{% highlight xml %}
<receiver
    android:name="com.google.android.gms.tagmanager.InstallReferrerReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
{% endhighlight %}

&nbsp;

That is about as painless as it gets. With this receiver in place we have our install tracking. If you are also using GA (you shouldn't be but if you are) this will automatically invoke the GA `CampaignTrackingReceiver`.

However there is one drawback, only one Receiver can register itself for this intent. If we want multiple receivers we are in trouble and it is surprising how frequently this can come up.

Fortunately there is a simple fix for this as well. We need to create our own receiver and from there fire everything we want.

&nbsp;

In AndroidManifest:

{% highlight xml %}
<receiver
    android:name=".receiver.PassthroughReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="com.android.vending.INSTALL_REFERRER" />
    </intent-filter>
</receiver>
{% endhighlight %}

&nbsp;

Nothing new there, we have taken out GTM's receiver in place of our own.

&nbsp;

But lets take a look at `PassthroughReceiver`

{% highlight xml %}
public class PassthroughReceiver extends BroadcastReceiver {
    private final BroadcastReceiver[] mReceivers;

    public PassthroughReceiver() {
        mReceivers = new BroadcastReceiver[] {
                new InstallReferrerReceiver(),
                new AnotherCustomReceiver(),
                new ForGoodMeasureOneMoreReceiver()
        };
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        for (BroadcastReceiver receiver : mReceivers) {
            receiver.onReceive(context, intent);
        }
    }
}
{% endhighlight %}

&nbsp;

Pretty easy! When our receiver is created by the OS we construct an array of everything we want to trigger. Then when the broadcast occurs we can loop through each one and trigger it. That's all there is to it so now you have no excuse to not be tracking your install referrer.

&nbsp;

---

##Exception Tracking

If we look in our analytics interface we can see a section which will allow us to view crash data. This is an interesting feature, but what exactly is it for? And how do we use it?

&nbsp;

![Exceptions 1]({{page.res}}/exceptions_1.png)

&nbsp;

This feature allows us to capture two different types of exceptions, caught and uncaught. We can feed this data into our reporting and analysis to see what kinds of impacts our app performance is having on goals and targets. To make things better for us, this feature is also supported (to some degree) in GTM!

&nbsp;

> Note: this is not a good tool for debugging crashes. Something like Crashlytics adds much more value than whatever GA can provide. So while it might be tempting to consolidate your tools, do not punish yourself by going through with whatever plan you may be dreaming up. The purpose of this feature is to consolidate metrics not functionality. Do not fear however, we will also ensure that our analytics does not conflict with whatever crash reporting tool we may be using.

&nbsp;

### How to do it

Let's split this up into each area and tackle them one at a time:

&nbsp;

#### Uncaught Exceptions

To track our uncaught exceptions we are going to need the same two sides we always need, the client and the container. 

&nbsp;

Let's start with the container. We are going to need a new variable and also a new trigger. So lets go ahead and set them up:

&nbsp;

![Exceptions 2]({{page.res}}/exceptions_2.png)

&nbsp;

![Exceptions 3]({{page.res}}/exceptions_3.png)

&nbsp;

So we have a description and we also have a trigger set to fire on our new event `fatalException`. Note here that we do not have a variable for a stack-trace. This is not sent up for us automatically either, we simply do not get one. This is important and part of the reason I said above this is **not** a debugging tool. GA is not designed to view stack-traces and this feature is not intended to capture them. Our description can give us some information and our secondary dimensions can give us a little more (possibly enough to debug the simplest of issues), but that is the extent.

&nbsp;

> Note: I have also set a default value for the description variable. This is a useful feature but pretty self explanatory. You may be able to think of a few cases in the previous articles were we could have used this option instead of doing it in code. There are pros and cons for either method but really you should be doing things this way as GTM should encapsulate all business rules to promote flexibility.

&nbsp;

**Now for the new bit:** once we have our variables and triggers, we need our exception tag.

&nbsp;

![Exceptions 4]({{page.res}}/exceptions_4.png)

&nbsp;

As you can see, this is actually not that scary. GA has an "Exception" type that takes a description. It also has a boolean indicating if the exception is fatal or non-fatal. That is all there is to it. Now we know that if we fire an event at GTM in our client with the name `fatalException` and a description we will get our crash!

&nbsp;

But how exactly do we do that? ...

&nbsp;

Well if you are familiar with how tools such as Crashlytics work, you probably know the answer. We need to assign a custom [UncaughtExceptionHandler][Handler] to our app's main `Thread`. This handler will have its `uncaughtException()` method invoked in the event of a crash. Typically handlers like this should intercept the event, conduct their action, then trigger the pre-existing UncaughtExceptionHandler. This allows for a chain of handlers, all operating in harmony. Say for instance the existing default was the Crashlytics' handler, if we don't pass it along our crash will get reported in GA but not make it to Crashlytics, even worse we have no idea what Crashlytics was going to pass along too. We could break a whole suite of functionality!

&nbsp;

That all being said, let's take a look at a `GTMExceptionHandler` (this is a custom class, we need to make it ourselves)

&nbsp;

{% highlight java %}
public class GTMExceptionHandler implements UncaughtExceptionHandler {
    private TagManager mTagManager;
    private ExceptionParser mExceptionParser;
    private UncaughtExceptionHandler mOriginalHandler;

    public GTMExceptionHandler(@NonNull Context context, @NonNull TagManager tagManager,  @Nullable UncaughtExceptionHandler originalHandler) {
        mTagManager = tagManager;
        mOriginalHandler = originalHandler;
        mExceptionParser = new StandardExceptionParser(context, new ArrayList<String>());
    }

    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        Map<String, Object> dataLayerVariables = new HashMap<>();
        if(mExceptionParser != null) {
            String threadName = thread != null ? thread.getName() : null;
            String description = mExceptionParser.getDescription(threadName, ex);
            dataLayerVariables = DataLayer.mapOf("description", description);
        }

        mTagManager.getDataLayer().pushEvent("fatalException", dataLayerVariables);

        if(mOriginalHandler != null) {
            mOriginalHandler.uncaughtException(thread, ex);
        }
    }
}
{% endhighlight %}

&nbsp;

So there are a few things happening here. Lets break it apart. When we construct our handler we pass in a context, our tag manager and the original handler that was set. The context is used to create a `StandardExceptionParser` which is a class from the analytics support library. We don't explicitly need to use this, but it does a great job of formatting a description for us.

Then when an exception occurs we create ourselves a new `HashMap<String, Object>` which will contain our description. We construct it in this way so that if our exception parser isn't created correctly for whatever reason, the event will not fail and we will still have the default description we set in GA.

Once we have a formatted description we fire our event to GTM and pass the exception up the chain. Simple!

&nbsp;

{% highlight java %}
Note: I encourage the use of constants for "description" and "fatalException", they were removed for readability
{% endhighlight %}


&nbsp;

We can set this handler in our application's `onCreate()`

{% highlight java %}
Thread.setDefaultUncaughtExceptionHandler(new GTMExceptionReporter(
        this,
        mTagManager,
        Thread.getDefaultUncaughtExceptionHandler())
);
{% endhighlight %}

&nbsp;

If we do this after setting up a crash reporting tool, we will be passing our values through to them, if we set it up before, they *should* be passing through to us. If the service you use has set things up right, this shouldn't cause an issue either way. Personally I like to set mine after however, just in case.

&nbsp;

Now if we head over to GA we can see our crashes coming through, here is a forced one to demonstrate what we can see.

&nbsp;

![Exceptions 5]({{page.res}}/exceptions_5.png)

&nbsp;

Fantastic! And if we click into our application version we can get a bit more information, including our description and also access to secondary dimensions (by default we will have options such as time, OS version and screen resolution but we can set others as outlined above).

&nbsp;

![Exceptions 6]({{page.res}}/exceptions_6.png)

&nbsp;

> Note: Crashes and exceptions are not under the real-time banner. We have to wait 24 hours before we see crashes show in GA

&nbsp;

### Caught Exceptions

Now we have implemented Uncaught exceptions, tracking caught ones seems trivial. We need to simply create another tag and trigger (remembering this time to change our non-fatal flag to false) then fire the event whenever we catch our exception. The trick here is deciding where to put the event firing code.

Ideally this is in once nice location so that you don't have to go around sticking tracking events all through your beautiful codebase. The ideal place will likely change depending on your architecture, but I will offer one option. 

If you are using [Timber][Timber] (which I highly recommend) you can plant a new logger and intercept all your tracked and logged exceptions that way.

&nbsp;

In your application's `onCreate()` you would simply call:

{% highlight java %}
Timber.plant(new GTMExceptionLogger(mTagManager));
{% endhighlight %}

&nbsp;

And inside `GTMExceptionLogger` we have:

{% highlight java %}
public class GTMExceptionLogger extends Timber.Tree {

    private TagManager mTagManager;

    public GTMExceptionLogger(@NonNull TagManager tagManager) {
        mTagManager = tagManager;
    }

    @Override
    protected void log(int priority, String tag, String message, Throwable t) {
        if (t != null) {
            // Construct description here
            mTagManager.getDataLayer().pushEvent("nonFatalException", dataLayerVariables);
        }
    }
}
{% endhighlight %}

&nbsp;

Which will be triggered whenever you invoke `Timber.e()`.

&nbsp;

And there we have it! Tracking our crashes through GA via GTM, an interesting metric that can provide a lot of insight for a small amount of work.

&nbsp;

---

## Summary

These are just a handful of features available to us using GTM with new ones being added constantly. I'd encourage you to have a play and see just what is available to you. Most developers have the impression that analytics isn't the most interesting of topics (I was one of them) and as such it is often very poorly done. However in my time investigating GTM and analytics in general, I have found that if implemented correctly, analytics tools can provide amazing insights into how people interact with your app. It informs and powers decisions and often challenges your assumptions. 

In this series I have covered the basics, but it is in no way a complete analytics solution. To truly gain value from your analytics you need to think about what you want to achieve, how you will know if you have achieved it and what questions you will be asking if you don't. If what you are tracking does not provide answers to these areas then you need to stop and rethink what you are doing.


[Crashlytics]: https://fabric.io/kits/android/crashlytics/summary
[Handler]: http://developer.android.com/reference/java/lang/Thread.UncaughtExceptionHandler.html
[Timber]: https://github.com/JakeWharton/timber
[Part 3]: {% post_url 2015-09-30-google-tag-manager %}
