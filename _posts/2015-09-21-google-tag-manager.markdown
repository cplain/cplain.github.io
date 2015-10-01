---
layout: post
title: Google Tag Manager - Part One
subtitle: Getting to know one another
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
res: /assets/2015-09-21-google-tag-manager
---

---

Today, I will be kicking off a series of articles to look at Google Tag Manager and getting familiar with how it works. But what is Google Tag Manager?

<!--end_excerpt-->

&nbsp;  

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then uses its own logic to construct and fire tracking events at any number of analytics tools. We will be taking a look at GTM over the next few articles, and by the end of this series we will have a working Tag Manager implementation in Android! If you are looking into GTM for iOS, the first few articles will be identical for you, it is only the client-side implementation that changes. For web, the core concepts and meanings are the same.

&nbsp;

~~~
Note: During the course of this series I will be showing real ids and values, this is to help you see what values are important and where they go. They will all be long deleted before you read them.
~~~

&nbsp;

---

## Why GTM

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

At its heart, GTM is very simple. The trick is wrapping your head around the components, what they are for and how they work together. To make it even simpler, that is exactly what I aim to do for you in this article.

GTM has only three core elements we need to understand. **The Container**, **The Client** and **The Analytics Suite(s)**. Below I will briefly outline these elements, what they look like and how they behave. Once we have all this in our heads we be ready to start setting up a GTM instance in the next article.

&nbsp;  

~~~
Just repeating myself: This section will be an overview, my aim here is to show you what everything looks like and what to expect. We will go into more depth in the next articles in the context of setting up some tracking in an example client.
~~~

&nbsp;


###The Container

This is the heart of GTM. The container has all the logic, variables and connections to additional suites. The container is what we change when we want to update what is tracked and how we track it. When we are happy with our changes we publish them as a new version of container and the changes take effect. 

Each client holds onto a version of the container which it will update periodically (note however it may take time for all clients to be on the new container version). When a client is deployed into production it must be given a version of the container use until it fetches the newer version. It is good practice to always update this to be the latest container at time of launch. 

&nbsp;

~~~
By default, the container becomes eligible to be refreshed every 12 hours. We can however
manually refresh the container if we desire.
~~~

&nbsp;

We edit our container via the [GTM web portal][GTM]. We will go over each section in enough depth for you to be able to operate the container effectively in the next article, but for now here is a first look at GTM:

&nbsp;

![Container]({{page.res}}/container_overview.png)

&nbsp;

This is a brand new container so there isn't anything of real interest happening here. I will show you how I set it up next time, so for now lets just get familiar with our new friend by exploring some of the core elements on the screen.

&nbsp;

####The Tab Bar

![Container Top]({{page.res}}/container_top.png)

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

![Container Side]({{page.res}}/container_side.png)

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

A GTM client can currently be one of three things: a website, an iOS app or an Android app. For the context this article we will deal with an Android mobile application we will call **GTMApp**. 

To follow some nice practices and ensure we can cover as many gotchas as possible, **GTMApp** will follow a clean MVP pattern, use [Dagger 2][Dagger] for dependency injection, have some unit tests that sit on top of [Roboelectric][Roboelectric] and will have several variants and flavours that point to different environments.

That is all we will need to say about our client for now. We will become a lot more familiar with **GTMApp** later.

&nbsp;

###The Analytics Suite(s)

GTM currently has three tools it talks to out of the box for mobile: **Google Analytics**, **Google AdWords** and **doubleclick by Google** (for web it is far  than three). It also has the option for custom function calls and so on. It can talk to as many or as few instances of these tools as you would like. If for some reason you wanted three hundred and ninety-four Google AdWords accounts all receiving tags from your app, nothing is stopping you.

In our example we will just deal with a single Google Analytics account. GA is where our data will wind up. We don't look at GTM to see our reports and results, we look at GA. It will have no idea GTM is sitting in-between the client and itself. GA is ideal for this as it has a real-time feature we can use to confirm our values are all correct. It is often believed that this isn't properly supported by GTM, but most often this is due to misconfiguration. 

&nbsp;

~~~
A quick note on talking to GA in real-time: A real-time event is visible in GA for 30 minutes before it leaves. After that 30 minutes the event is not viewable until 24 hours later when it appears as part of the daily reports and figures. This is just how GA works as it needs to run processing etc on all the daily figures to generate meaningful reports and statistics.
~~~

&nbsp;

---

## Summary

So we now have a basic understanding of GTM, and we know how it all looks. This is all great but we haven't really done anything yet. But soon enough we will start to see how these components come together in a super simple way to allow some pretty powerful analytics.

&nbsp;

**[Check out the next article, where we learn how to make our very own container!][Part 2]**

[GTM]: https://tagmanager.google.com/
[Dagger]: http://google.github.io/dagger/
[Roboelectric]: http://robolectric.org/
[Part 2]: {% post_url 2015-09-25-google-tag-manager %}