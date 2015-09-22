---
layout: post
title: Google Tag Manager
subtitle: The platform where the tagline is included
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
---

{% assign res = "/assets/2015-09-21-google-tag-manager" %}

---

## Overview

&nbsp;

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then uses its own logic to construct and fire tracking events at any number of analytics tools.  

&nbsp;  

~~~
Note: During the course of this article I will be showing real ids and values, this is to help you see what values are important and where they go. They will all be long deleted before you read them.
~~~

&nbsp;

---

## Why GTM

&nbsp;

There are three core benefits of GTM:

1. The client need only talk to the single tool to communicate with multiple different suites. This makes analytics easier to implement, maintain and structure

2. Additional tools can be hooked in after the client has been deployed, if another team wants their own instance of a tool or a new tool altogether it can be put in without a redeploy of the client

3. What is tracked and how it is sent through to the analytics suites can also be changed without redeploying the client. If the business no longer wants to see a metric, it can be removed

&nbsp;  

~~~
Tip: If you only want a limited set of tracking to start, build the clients to track everything within reason. Then you can limit the tracking at the GTM layer. Which means GTM can be changed to send it through when people inevitably change their minds. No code changes or client launches required!
~~~

&nbsp;

---

## How it works

&nbsp;

At its heart, GTM is very simple. The trick is wrapping your head around the components, what they are for and how they work together. To make it even simpler, that is exactly what I aim to do for you in this article.

GTM has only three core elements we need to understand. **The Container**, **The Client** and **The Analytics Suite(s)**. Below I will briefly outline these elements, what they look like and how they behave. Once we have all this in our heads we will run through setting up GTM in our client.

&nbsp;  

~~~
Just repeating myself: This section will be an overview, my aim here is to show you what everything looks like and what to expect. We will go into more depth later in the context of setting up some tracking in an example client.
~~~

&nbsp;


###The Container

&nbsp;

This is the heart of GTM. The container has all the logic, variables and connections to additional suites. The container is what we change when we want to update what is tracked and how we track it. When we are happy with our changes we publish them as a new version of container and the changes take effect. 

Each client holds onto a version of the container which it will update periodically (note however it may take time for all clients to be on the new container version). When a client is deployed into production it must be given a version of the container use until it fetches the newer version. It is good practice to always update this to be the latest container at time of launch. 

&nbsp;

~~~
The exact update logic is: 
WHEN a client is launched
IF it has been more than 24 hours since the list time it checked
THEN check and update the container
~~~

&nbsp;

We edit our container via the [GTM web portal][GTM]. We will go over each section in enough depth for you to be able to operate the container effectively, but for now here is a first look at GTM:

&nbsp;

![Container]({{res}}/container_overview.png)

&nbsp;

This is a brand new container so there isn't anything of real interest happening here. I will show you how I set it up in a bit but for now lets just get familiar with our new friend by exploring some of the core elements on the screen.

&nbsp;

####The Tab Bar

&nbsp;

![Container Top]({{res}}/container_top.png)

&nbsp;

I am calling this the tab bar even though it is more of a composite navigational bar (doesn't have the same ring does it). There are a few items of interest I have highlighted. For the sake of brevity we will not go over everything on the bar, just these six items.

&nbsp;

1. **Container ID:** This is the unique identifier for your container. It is the reference that your clients will point to when communicating to it.

2. **Accounts Tab:** This is the default tab you will open on, it simply contains a list of accounts and containers you have access too, when working on your container you will rarely visit this tab.

3. **Container Tab:** This is the core tab we will deal in, the default view is shown above.

4. **Versions Tab:** A history of all the versions for your container is found under this tab. When publishing a new version you can go here to download the container file, which is used as the default container for the client. You can also push an older version back to live and perform a few other management operations here.

5. **Admin Tab:** This tab contains settings, some of which will be useful later on. This is also the tab you are taken to when setting up a new account or container.

6. **Publish Button:** This button will save any unpublished changes in your current working version mark that version as 'live'. It will then create a new working version for you. You can mark an older version of the container as 'live' later on if you have made a mistake, but nevertheless be careful around this button.

&nbsp;

####The Side Bar

&nbsp;

![Container Side]({{res}}/container_side.png)

&nbsp;

The side bar will only be visible in the **Container Tab**, it is however something you will become very familiar with. The highlighted three: **Tags**, **Triggers** and **Variables**  are core to GTM so we will discuss each one in depth a bit later. Here I will just provide a brief summary similar to how we approached the top bar.

&nbsp;

~~~
Search, Overview and Folders are not relevant to the content of this article, they are a discussion for another time.
~~~

&nbsp;

1. **Tags:** these are your events, each tag represents an interaction or occurrence you want to send to some analytics tool. Here you can view your current tags, add new ones and edit old ones.

2. **Triggers:** are what power your tags, each tag can have one or more triggers. When the trigger is fired that will cause any attached tag to activate. Here you can view your triggers, add new ones and edit old ones. Similar to the tags section.

3. **Variables:** come in many shapes and sizes, most simply they are representative of the data flowing through GTM. Here, like before, you can view your current variables, add new ones and edit old ones.

&nbsp;

###The Client

&nbsp;

A GTM client can currently be one of three things: a website, an iOS app or an Android app. For the context this article we will deal with an Android mobile application we will call **GTMApp**. 

To follow some nice practices and ensure we can cover as many gotchas as possible, **GTMApp** will follow a clean MVP pattern, use Dagger 2 for dependency injection, have some unit tests that sit on top of Roboelectric and will have several variants and flavours that point to different environments.

That is all we will need to say about our client for now. We will become a lot more familiar with **GTMApp** later.

&nbsp;

###The Analytics Suite(s)

&nbsp;

GTM currently has three tools it talks to out of the box: **Google Analytics**, **Google AdWords** and **doubleclick by Google**. It also has the option for custom function calls and so on. It can talk to as many or as few instances of these tools as you would like. If for some reason you wanted three hundred and ninety-four Google AdWords accounts all receiving tags from your app, nothing is stopping you.

In our example we will just deal with a single Google Analytics account. GA is where our data will wind up. We don't look at GTM to see our reports and results, we look at GA. It will have no idea GTM is sitting in-between the client and itself. GA is ideal for this as it has a real-time feature we can use to confirm our values are all correct. It is often believed that this isn't properly supported by GTM, but most often this is due to misconfiguration. 

&nbsp;

~~~
A quick note on talking to GA in real-time: A real-time event is visible in GA for 30 minutes before it leaves. After that 30 minutes the event is not viewable until 24 hours later when it appears as part of the daily reports and figures. This is just how GA works as it needs to run processing etc on all the daily figures to generate meaningful reports and statistics.
~~~

&nbsp;

---


## Lets get started

&nbsp;

###The Plan

&nbsp;

Before we go charging headfirst into everything lets first stop and go over what we will be doing. 

First we will need to create a GTM container, next a GA instance to talk to, then we will need to set up the container to receive events from a client and pass them through to GA. Once that is all done we can then set up our client to talk to GTM and just like that we have analytics!

It all sounds easy but some of those steps can be a little involved. If you're getting frustrated with where we are going, just stop and remember the plan. I'll stick to it if you will.

&nbsp;

###Let there be GTM

&nbsp;

Step one is acquiring ourselves a GTM container to work with. Log into [https://tagmanager.google.com/][GTM] with your Google account. You should come straight to the account setup page below:

&nbsp;

![New Container Step 1]({{res}}/new_container_1.png)

&nbsp;

I have gone ahead and stuck "Example" as my account name. You however might want to put a bit more thought into this. Remember this is not the name of your container, your account can have several containers. For instance you may want to have one account per project with a container for iOS, Android and Web. Or you may want to have an account for your company and one or more containers per project. This is your decision.

&nbsp;

~~~
While you definitely can have one GTM container that handles multiple clients (of the same platform), it is generally considered poor practice. You most likely never want a single out of the box analytics experience. If you have two apps that are very similar consider copying the container (I will discuss how to do this later) and making modifications per specific client. 
~~~

&nbsp;

Once you hit continue you have your account. Now we can set up a container!

&nbsp;

![New Container Step 2]({{res}}/new_container_2.png)

&nbsp;

Again I have decided to be original and call my container "Example" you will probably want something a little more informative. I could have, for instance, called it "GTMApp - Android". Then pick your platform, for us, Android.

Once you click create you will see this screen:

&nbsp;

![New Container Step 3]({{res}}/new_container_3.png)

&nbsp;

That link will direct you to [the android v4 SDK][GTM v4] which is the latest sdk at time of writing. You don't need to worry about it, I'll be talking you through everything it will say as I go through this article.

Instead click 'OK' and admire your new container!

If you ever want to create another account you can in the **Accounts Tab** by clicking on the blue link **Create Account** and following the same steps.

&nbsp;

![New Container Step 4]({{res}}/new_container_4.png)

&nbsp;

If you want to create a new container under the same account, you can simply click on the overflow menu and select **Create Container** you will follow a very similar process as above without the account step.

&nbsp;

![New Container Step 5]({{res}}/new_container_5.png)

&nbsp;

And thats that! All done, we have our brand new container and we know how to make more. On to the next stage of our plan.

&nbsp;

###Bring in the Analytics

&nbsp;

Now we have a container, it's time to create something for it to talk to. Odds are you already know how to view and use GA so I will cover it only briefly. The great part about GA is it requires very little setup to get working.

First head to the [Google Analytics website][GA] and log in with your Google account.

If you have never had a GA account before you will see something like this:

&nbsp;

![New GA Step 1]({{res}}/new_ga_1.png)

&nbsp;

Lets click on that big friendly sign up button!

&nbsp;

![New GA Step 2]({{res}}/new_ga_2.png)

&nbsp;

So I have started to fill in my account details. The most important part is to select **Mobile App** up the top. Then similar to GTM you need an account and app name, most likely you will have these match your GTM account and GTM container names. Finally you need to select a **Reporting Time Zone**. Most likely this will be your current time zone. However despite not being in Perth I have decided that will do me just fine for this example.

Below this we will see a number of data sharing services (all helpfully pre-ticked for you by Google because it's nice like that) It is your decision as to whether you want Google to have this information or not.

&nbsp;

![New GA Step 3]({{res}}/new_ga_3.png)

&nbsp;

Once we are all done we will see our new GA instance in all its glory. I have highlighted the important bit for you. That is your tracking id and it is what GTM needs to allow it to talk to this GA account. Make sure you can find it again, **Admin > Tracking info > Tracking Code**. (also you may want to close those help windows)

&nbsp;

![New GA Step 4]({{res}}/new_ga_4.png)

&nbsp;

You can always create another GA instance by clicking on the **Property** dropdown and selecting **Create new property** you can do a similar action for creating a new account via the **Account** dropdown.

&nbsp;

![New GA Step 5]({{res}}/new_ga_5.png)

&nbsp;

This is GA done! Later we will need to come back to the **Reporting tab** and go to the **Real-Time section** to see our data come through. But until that point in time it is a temporary farewell to our GA account.

&nbsp;

###Setting up our container

&nbsp;

Here comes the good stuff. We now have everything we need to get started. In this example I am going to set up GTM to do two things. Track all page views and all button clicks in my app. These two situations are ~90% of cases in every analytics solution we will implement, and we can do it now with just two tags.

&nbsp;

####Step One: Variables

&nbsp;

I am going to create four variables that we will use for everything. As I will point out later we could make a few more, but I will limit it in our example to minimise complexity.

First we will need a single place to store our tracking id we created in GA. We could use it over and over without making a variable, but this is bad practice for all the reasons you wouldn't do that in code. It makes things harder to read and it becomes extremely painful to change later on.

So first lets go to the **Variables section** in GTM.

&nbsp;

![Variables Step 1]({{res}}/variables_1.png)

&nbsp;

GTM gives us quite a few useful variables out of the box. The client doesn't need to do anything special to pass these along. If want to use one we can go ahead and use it without worrying about getting the client to tell us what the value is. With variables such as **App Name**, **App Version Code** and **App Version Name** we don't even need to worry about GA. Because GTM will pass them through to GA without us even needing to tell it! We can enable any one of the variables in any one of the boxes with a simple check, however for our purposes the default set is fine.

Below this section we see the **User-Defined Variables** section, this is where our variables will live. So lets get on with it and make our tracking id.

First we click on **New** and we are greeted with the following:

&nbsp;

![Variables Step 2]({{res}}/variables_2.png)

&nbsp;

There are quite a lot of variables available, they all have different purposes and varying degrees of usefulness. However don't get overwhelmed we will only be dealing with **Constants** and **Data Layer Variables**. 

&nbsp;

- A **Constant** is exactly what you think it is, it is an unchanging value that we label and keep in the one place to make things both easier to read and easier to change in the future (this is sounding exactly like what we need for our tracking id)

- A **Data Layer Variable** is an exceptionally useful variable with an exceptionally awful name. Most simply, these are the values the client will be sending to us. When something analytics worthy happens, the client will take whatever values it needs and bundle them together in an event and then send this to what is called the **Data Layer** this then sends the event to be processed by the container.

&nbsp;

So for now, lets create a **Constant** for our tracking id:

&nbsp;

![Variables Step 3]({{res}}/variables_3.png)

&nbsp;

Up the top we name our variable. This will be how we refer to it later. Remember in most analytics solutions you will have a lot of variables, so you may want to define some sort of convention to make the list easier to navigate in future.

Then in the value field we put in our tracking id. That means now, whenever I use this variable via the syntax `{% raw %}{{GA - Tracking ID}}{% endraw %}` I am actually referencing my id!

When we click **Create Variable** we will see our new constant in the **User-Defined Variables** section. If we ever want to update the name, or value or even type we can by simply clicking on it.

&nbsp;

![Variables Step 4]({{res}}/variables_4.png)

&nbsp;

Well now we know how to create variables and what the two key types are, lets make the three other  values we will need to track page view and button presses. They are: 

- The name of the screen 

- The name of the button being pressed and 

- Any special value the button may have. This could be and on or off state for a checkbox, the address of a url for a clickable link, the phone number that is about to be called, whatever you'd like.

&nbsp;

~~~
You probably noticed a lot of these examples are for screen elements that aren't necessarily buttons. I never said I was going to do ONLY buttons with a single tag did I? 
~~~

&nbsp;

So here are our new variables:

&nbsp;

![Variables Step 5]({{res}}/variables_5.png)

&nbsp;

You'll notice they are slightly different than our constant. Obviously we don't want to see **screenName** whenever we type `{% raw %}{{Data - Common - ScreenName}}{% endraw %}` and the same would go for all the other variables. In this case however this "value" is actually the key the client uses to assign a real value to that field. So in our client somewhere we will map **screenName** with whatever we decide to name our screen. We also have the option to set a default value for these variables, in our cases we don't really need a default screen name or click target so I have opted to leave them out (In the case of click value it could make sense but I have opted for another way for simplicity).

&nbsp;

####Step Two: Triggers

&nbsp;

So now we have our variables, but they aren't that useful right now. They just sit there... Doing nothing... What we need is a way to determine when something has happened, not just any something, but the thing we are wanting to track. When we know it has occurred then we know to tell GA and pass along our variables. Thankfully GTM has a mechanism for this, **Triggers**. So lets navigate to the **Triggers section** in GTM.

&nbsp;

![Triggers Step 1]({{res}}/triggers_1.png)

&nbsp;

As we would expect it is also empty. So to begin lets click on **New** and make a trigger that fires whenever the client tells us there is a page view.

&nbsp;

![Triggers Step 2]({{res}}/triggers_2.png)

&nbsp;

There are a few elements we have here. Up the top is the name, similar to our variables. We can set this to whatever we would like, it is how we reference our trigger later. The new stuff is the **Choose Event** and **Fire On** sections. 

- **Choose Event:** we are stuck, it will always be set to "Custom" for an Android app. 

- **Fire On:** this is the brains of the trigger. The trigger will only fire when the conditions we outline here are true. We can have as many conditions as we want to make this as granular as we need. In this trigger we can use any variable available to us and use a number of different comparisons to determine if the event the client sent us is worth tracking. You can see them all below.

&nbsp;

![Triggers Step 3]({{res}}/triggers_3.png)

&nbsp;

Notice the **Event** variable. We haven't seen this guy yet. As I mentioned in the variables section, when a client sends data to GTM it does so via pushing an event to the data layer. When it does this the client will assign a name to the type of event it just pushed. This can be whatever we would like to name it, for example "pageView". The **Event** variable contains this name, so it is the perfect field to do our comparisons on.

&nbsp;

![Triggers Step 4]({{res}}/triggers_4.png)

&nbsp;

~~~
If you have your smart pants on today you are probably thinking to yourself that "pageView" would be perfect for a constant. And you would be right! This is just one of the many more values we will encounter that would be constants in the real world but I am leaving as their actual values here for simplicities sake.
~~~
&nbsp;

Now when we click **Create Trigger** we will see our brand new Trigger ready to go. 

&nbsp;

![Triggers Step 5]({{res}}/triggers_5.png)

&nbsp;

We will create one more for our click event...

&nbsp;

![Triggers Step 6]({{res}}/triggers_6.png)

&nbsp;

And done! We now have a way to detect when interesting things are happening!

&nbsp;

####Step Three: Tags

&nbsp;

So we have our data and we have a way to tell us something important is happening in that data. But what we don't have is any way to do something about it. 

Enter **Tags**! As you can guess by the name, these are the core element of Google **Tag** Manager. Tags are our actions, they bind to triggers and are activated when one of those triggers fires. They then use the variables and data provided to compose and send something meaningful to Google Analytics. These are what we have been building towards, the culmination of our efforts!

So lets get started by navigating to the **Tags section**

&nbsp;

![Tags Step 1]({{res}}/tags_1.png)

&nbsp;

Just like before this section is empty. Lets start by clicking on **New** and creating our page views tag. This tag will fire through to GA and will be visible under **Screens** in the **Real-Time** section.


&nbsp;

![Tags Step 2]({{res}}/tags_2.png)

&nbsp;

When we create a new tag the first choice we are greeted with is what analytics product we would like to use. In our case this is easy, **Google Analytics**

We then need to configure our tag...

&nbsp;

![Tags Step 3]({{res}}/tags_3.png)

&nbsp;

Just like with variables and triggers, the name up the top is our own internal reference to the tag. More importantly we see our **Tracking ID** field. This is where we tell GTM that this tag will be sent to the GA instance we made earlier. If we wanted to also send the same event to another instance all we have to do is copy this tag and change that field. The final field is **Track Type**, this defaults to "App View" which is GTM's name for a page view, so conveniently for us the work is already done. We just need to click **Continue** and attach a trigger to our tag.

&nbsp;

![Tags Step 4]({{res}}/tags_4.png)

&nbsp;

Our dialog here is pretty simple. We can attach as many triggers as we want. In our case however we only want the page view trigger.

&nbsp;

~~~
Note: A tag will activate if any one of its attached triggers fires. Attaching multiple triggers does not mean your tag holds out until all triggers are true. There is no support for this, you would need to make a third trigger with all the constraints of the other two. However if more than one trigger is true in a single event, your tag will still only fire the once.
~~~

&nbsp;

So lets take a look at our final tag.

&nbsp;

![Tags Step 5]({{res}}/tags_5.png)

&nbsp;

Looks pretty good, all we need to do it click **Create Tag** and we are up and running with page tracking. All a client needs to do is send an event to us with the name **pageView** and we will pass that along to GA with the app name, version code and all those other built in variables. In real time!

&nbsp;

![Tags Step 6]({{res}}/tags_6.png)

&nbsp;

So now we have our page view tag. Lets create a tag for all the click events our client sends to us. This will look slightly different but isn't much harder. We will skip over choosing the analytics tool and just jump straight into configuring the tag.

&nbsp;

![Tags Step 7]({{res}}/tags_7.png)

&nbsp;

So, this time we are tracking an **Event**, to be clear, this  is not the same as what we send from the client and use in our triggers. This is a Google Analytics Event, they are visible under **Events** in the **Real-Time** section in GA. 

GA events have four key fields: **Category**, **Action**, **Label** and **Value**. These fields are viewed in GA like a hierarchy with **Category** at the top and **Value** at the bottom.

We can pass any data we like in these fields `with the exception of value, value must be a number, if it isn't a number you will not get errors but you also won't see your events show up in GA` 

&nbsp;

![Tags Step 8]({{res}}/tags_8.png)

&nbsp;


In my case I want a "User Action" **Category** because later I might have a "Push" category and a "Network Event" category. It would be nicer to bundle all my user interactions under the one category instead of having "Click", "Swipe", "Screenshot" etc all cluttering up my top level. Also note, this would be a good place to have a **Constant**

With my **Action** and **Label** fields you will see we can easily combine variables and text to provide context, creating more meaningful data to send through to the analytics.

I set my **Value** to empty because for our purposes there is no reason to send numbers through to GA.

Finally the **Non-Interaction Hit** field is a simple boolean. It is a way of indicating to GA that this event was caused by a deliberate user action and not something automated. It defaults to false as 90% of the time we are tracking user actions.

Lets tie this to our click trigger and hit the go button!

&nbsp;

![Tags Step 9]({{res}}/tags_9.png)

&nbsp;

And there we have it! Our two tags capable of tracking every page view and click event in a whole app. Very powerful stuff and we haven't even broached the advanced options. All we need to do now is publish this container and we can turn our attention to the final piece of the puzzle, the client.

&nbsp;

####Step Four: Publishing

&nbsp;

Time to click that little red button that has been watching over us this whole time.

&nbsp;

![Publish Step 1]({{res}}/publish_1.png)

&nbsp;

If you click on the dropdown you may see there are actually two options available to you. The only difference between them is one makes your changes live while the other does not. Both save your current version and both give you a new working version to play with. If you do opt for the blue button, you can always go and set your saved version to live later on when you are ready.

&nbsp;

![Publish Step 2]({{res}}/publish_2.png)

&nbsp;

But for us, we just want to hit publish.

&nbsp;

![Publish Step 3]({{res}}/publish_3.png)

&nbsp;

We will see a nice little confirmation window. We can just click **Publish Now** but remember, this is your last chance to back out. Once you hit publish you are live for a minimum of 24 hours the second a client happens to pick up your change before you realise you made a mistake. The more users you have the more likely this is, so be careful.


&nbsp;

![Publish Step 4]({{res}}/publish_4.png)

&nbsp;

TADA! We are done! If we go to the **Versions tab** we can see version 1 of our container up and running.

&nbsp;

![Publish Step 5]({{res}}/publish_5.png)

&nbsp;

Our final step is to download the container file for our client. This is the file our clients will start with until they pull down the latest container. In our case there isn't a new one to pull down which makes life a lot easier! But if we then release our app and want to change the analytics a month later we can publish our new container and the clients will pick it up. But be aware, any newly downloaded clients will still start on the old container before they get to update.

&nbsp;

![Publish Step 6]({{res}}/publish_6.png)

&nbsp;

And this is the GTM configuration done! All that is left is setting up our Android app.

&nbsp;

###Configuring the app

&nbsp;

Running GTM in an app can be extraordinarily simple, it can also present quite a few challenges and hidden surprises depending on what you are trying to do.

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


- Roboelectric and GTM **do not play nice together**

- We might only want to track application layer events now, but what if we want to track something in the network or database in future?

&nbsp;

Lets discuss the first point. What happens with a TagManager during testing? There are two big gotchas. 

1. First, if we call the TagManager from a test or execute a method that triggers the TagManager, our test will lock. Thats it, no more test. So we need to either avoid calling a real TagManager in testing or avoid those lines. Now ideally we don't want if statements all around our tracking code, it would be a lot nicer to simply swap out our real TagManager with a mock one. Sounds like Dagger may help here.


2. The second issue is a bit more subtle. Roboelectric has issues with anything loading resources in an Application's onCreate method. This can be most commonly seen if you have Crashlytics installed. As Crashlytics boots you will get ConcurrentModificationExceptions through your tests. GTM will give you the same problem. So not only do we want to swap our real TagManager with a mock one in tests, we actually don't want to create a real one in the first place.

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

But hopefully for now this article has you equipped to handle the basics and use this new insight to better your products and brands. And as you push ahead with this new platform you can grow your skills and step bravely into deeper waters!

Good luck

[GTM]: https://tagmanager.google.com/
[GTM v4]: https://developers.google.com/tag-manager/android/v4/
[GA]:www.google.com.au/analytics/