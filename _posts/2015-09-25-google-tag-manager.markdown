---
layout: post
title: Google Tag Manager - Part Two
subtitle: All that cloud nonsense
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
res: /assets/2015-09-25-google-tag-manager
---

---

Now that [we are familiar with GTM][Part 1], we will setting up out own container and hooking it up to Google Analytics! This is the heaviest article in the series by far, but most of this is images so you can see what is going on.

<!--end_excerpt-->

&nbsp;

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then uses its own logic to construct and fire tracking events at any number of analytics tools. We will be taking a look at GTM over the next few articles, and by the end of this series we will have a working Tag Manager implementation in Android! If you are looking into GTM for iOS, the first few articles will be identical for you, it is only the client-side implementation that changes. For web, the core concepts and meanings are the same.

&nbsp;

---

## Lets get started

Before we go charging headfirst into everything lets first stop and go over what we will be doing. 

First we will need to create a GTM container, next a GA instance to talk to, then we will need to set up the container to receive events from a client and pass them through to GA. Once that is all done we are ready to set up our client to talk to GTM in the next article!

It all sounds easy but some of those steps can be a little involved. If you're getting frustrated with where we are going, just stop and remember the plan. I'll stick to it if you will.

&nbsp;

###Let there be GTM

Step one is acquiring ourselves a GTM container to work with. Log into [https://tagmanager.google.com/][GTM] with your Google account. You should come straight to the account setup page below:

&nbsp;

![New Container Step 1]({{page.res}}/new_container_1.png)

&nbsp;

I have gone ahead and stuck "Example" as my account name. You however might want to put a bit more thought into this. Remember this is not the name of your container, your account can have several containers. For instance you may want to have one account per project with a container for iOS, Android and Web. Or you may want to have an account for your company and one or more containers per project. This is your decision.

&nbsp;

~~~
While you definitely can have one GTM container that handles multiple clients (of the same platform), it is generally considered poor practice. You most likely never want a single out of the box analytics experience. If you have two apps that are very similar consider copying the container (I will discuss how to do this later) and making modifications per specific client. 
~~~

&nbsp;

Once you hit continue you have your account. Now we can set up a container!

&nbsp;

![New Container Step 2]({{page.res}}/new_container_2.png)

&nbsp;

Again I have decided to be original and call my container "Example" you will probably want something a little more informative. I could have, for instance, called it "GTMApp - Android". Then pick your platform, for us, Android.

Once you click create you will see this screen:

&nbsp;

![New Container Step 3]({{page.res}}/new_container_3.png)

&nbsp;

That link will direct you to [the android v4 SDK][GTM v4] which is the latest sdk at time of writing. You don't need to worry about it, I'll be talking you through everything it will say as I go through this article.

Instead click 'OK' and admire your new container!

If you ever want to create another account you can in the **Accounts Tab** by clicking on the blue link **Create Account** and following the same steps.

&nbsp;

![New Container Step 4]({{page.res}}/new_container_4.png)

&nbsp;

If you want to create a new container under the same account, you can simply click on the overflow menu and select **Create Container** you will follow a very similar process as above without the account step.

&nbsp;

![New Container Step 5]({{page.res}}/new_container_5.png)

&nbsp;

And thats that! All done, we have our brand new container and we know how to make more. On to the next stage of our plan.

&nbsp;

###Bring in the Analytics

&nbsp;

Now we have a container, it's time to create something for it to talk to. Odds are you already know how to view and use GA so I will cover it only briefly. The great part about GA is it requires very little setup to get working.

First head to the [Google Analytics website][GA] and log in with your Google account.

If you have never had a GA account before you will see something like this:

&nbsp;

![New GA Step 1]({{page.res}}/new_ga_1.png)

&nbsp;

Lets click on that big friendly sign up button!

&nbsp;

![New GA Step 2]({{page.res}}/new_ga_2.png)

&nbsp;

So I have started to fill in my account details. The most important part is to select **Mobile App** up the top. Then similar to GTM you need an account and app name, most likely you will have these match your GTM account and GTM container names. Finally you need to select a **Reporting Time Zone**. Most likely this will be your current time zone. However despite not being in Perth I have decided that will do me just fine for this example.

Below this we will see a number of data sharing services (all helpfully pre-ticked for you by Google because it's nice like that) It is your decision as to whether you want Google to have this information or not.

&nbsp;

![New GA Step 3]({{page.res}}/new_ga_3.png)

&nbsp;

Once we are all done we will see our new GA instance in all its glory. I have highlighted the important bit for you. That is your tracking id and it is what GTM needs to allow it to talk to this GA account. Make sure you can find it again, **Admin > Tracking info > Tracking Code**. (also you may want to close those help windows)

&nbsp;

![New GA Step 4]({{page.res}}/new_ga_4.png)

&nbsp;

You can always create another GA instance by clicking on the **Property** dropdown and selecting **Create new property** you can do a similar action for creating a new account via the **Account** dropdown.

&nbsp;

![New GA Step 5]({{page.res}}/new_ga_5.png)

&nbsp;

This is GA done! Later we will need to come back to the **Reporting tab** and go to the **Real-Time section** to see our data come through. But until that point in time it is a temporary farewell to our GA account.

&nbsp;

###Setting up our container

Here comes the good stuff. We now have everything we need to get started. In this example I am going to set up GTM to do two things. Track all page views and all button clicks in my app. These two situations are ~90% of cases in every analytics solution we will implement, and we can do it now with just two tags.

&nbsp;

####Step One: Variables

I am going to create four variables that we will use for everything. As I will point out later we could make a few more, but I will limit it in our example to minimise complexity.

First we will need a single place to store our tracking id we created in GA. We could use it over and over without making a variable, but this is bad practice for all the reasons you wouldn't do that in code. It makes things harder to read and it becomes extremely painful to change later on.

So first lets go to the **Variables section** in GTM.

&nbsp;

![Variables Step 1]({{page.res}}/variables_1.png)

&nbsp;

GTM gives us quite a few useful variables out of the box. The client doesn't need to do anything special to pass these along. If want to use one we can go ahead and use it without worrying about getting the client to tell us what the value is. With variables such as **App Name**, **App Version Code** and **App Version Name** we don't even need to worry about GA. Because GTM will pass them through to GA without us even needing to tell it! We can enable any one of the variables in any one of the boxes with a simple check, however for our purposes the default set is fine.

Below this section we see the **User-Defined Variables** section, this is where our variables will live. So lets get on with it and make our tracking id.

First we click on **New** and we are greeted with the following:

&nbsp;

![Variables Step 2]({{page.res}}/variables_2.png)

&nbsp;

There are quite a lot of variables available, they all have different purposes and varying degrees of usefulness. However don't get overwhelmed we will only be dealing with **Constants** and **Data Layer Variables**. 

&nbsp;

- A **Constant** is exactly what you think it is, it is an unchanging value that we label and keep in the one place to make things both easier to read and easier to change in the future (this is sounding exactly like what we need for our tracking id)

- A **Data Layer Variable** is an exceptionally useful variable with an exceptionally awful name. Most simply, these are the values the client will be sending to us. When something analytics worthy happens, the client will take whatever values it needs and bundle them together in an event and then send this to what is called the **Data Layer** this then sends the event to be processed by the container.

&nbsp;

So for now, lets create a **Constant** for our tracking id:

&nbsp;

![Variables Step 3]({{page.res}}/variables_3.png)

&nbsp;

Up the top we name our variable. This will be how we refer to it later. Remember in most analytics solutions you will have a lot of variables, so you may want to define some sort of convention to make the list easier to navigate in future.

Then in the value field we put in our tracking id. That means now, whenever I use this variable via the syntax `{% raw %}{{GA - Tracking ID}}{% endraw %}` I am actually referencing my id!

When we click **Create Variable** we will see our new constant in the **User-Defined Variables** section. If we ever want to update the name, or value or even type we can by simply clicking on it.

&nbsp;

![Variables Step 4]({{page.res}}/variables_4.png)

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

![Variables Step 5]({{page.res}}/variables_5.png)

&nbsp;

You'll notice they are slightly different than our constant. Obviously we don't want to see **screenName** whenever we type `{% raw %}{{Data - Common - ScreenName}}{% endraw %}` and the same would go for all the other variables. In this case however this "value" is actually the key the client uses to assign a real value to that field. So in our client somewhere we will map **screenName** with whatever we decide to name our screen. We also have the option to set a default value for these variables, in our cases we don't really need a default screen name or click target so I have opted to leave them out (In the case of click value it could make sense but I have opted for another way for simplicity).

&nbsp;

####Step Two: Triggers

So now we have our variables, but they aren't that useful right now. They just sit there... Doing nothing... What we need is a way to determine when something has happened, not just any something, but the thing we are wanting to track. When we know it has occurred then we know to tell GA and pass along our variables. Thankfully GTM has a mechanism for this, **Triggers**. So lets navigate to the **Triggers section** in GTM.

&nbsp;

![Triggers Step 1]({{page.res}}/triggers_1.png)

&nbsp;

As we would expect it is also empty. So to begin lets click on **New** and make a trigger that fires whenever the client tells us there is a page view.

&nbsp;

![Triggers Step 2]({{page.res}}/triggers_2.png)

&nbsp;

There are a few elements we have here. Up the top is the name, similar to our variables. We can set this to whatever we would like, it is how we reference our trigger later. The new stuff is the **Choose Event** and **Fire On** sections. 

- **Choose Event:** we are stuck, it will always be set to "Custom" for an Android app. 

- **Fire On:** this is the brains of the trigger. The trigger will only fire when the conditions we outline here are true. We can have as many conditions as we want to make this as granular as we need. In this trigger we can use any variable available to us and use a number of different comparisons to determine if the event the client sent us is worth tracking. You can see them all below.

&nbsp;

![Triggers Step 3]({{page.res}}/triggers_3.png)

&nbsp;

Notice the **Event** variable. We haven't seen this guy yet. As I mentioned in the variables section, when a client sends data to GTM it does so via pushing an event to the data layer. When it does this the client will assign a name to the type of event it just pushed. This can be whatever we would like to name it, for example "pageView". The **Event** variable contains this name, so it is the perfect field to do our comparisons on.

&nbsp;

![Triggers Step 4]({{page.res}}/triggers_4.png)

&nbsp;

~~~
If you have your smart pants on today you are probably thinking to yourself that "pageView" would be perfect for a constant. And you would be right! This is just one of the many more values we will encounter that would be constants in the real world but I am leaving as their actual values here for simplicities sake.
~~~
&nbsp;

Now when we click **Create Trigger** we will see our brand new Trigger ready to go. 

&nbsp;

![Triggers Step 5]({{page.res}}/triggers_5.png)

&nbsp;

We will create one more for our click event...

&nbsp;

![Triggers Step 6]({{page.res}}/triggers_6.png)

&nbsp;

And done! We now have a way to detect when interesting things are happening!

&nbsp;

####Step Three: Tags

So we have our data and we have a way to tell us something important is happening in that data. But what we don't have is any way to do something about it. 

Enter **Tags**! As you can guess by the name, these are the core element of Google **Tag** Manager. Tags are our actions, they bind to triggers and are activated when one of those triggers fires. They then use the variables and data provided to compose and send something meaningful to Google Analytics. These are what we have been building towards, the culmination of our efforts!

So lets get started by navigating to the **Tags section**

&nbsp;

![Tags Step 1]({{page.res}}/tags_1.png)

&nbsp;

Just like before this section is empty. Lets start by clicking on **New** and creating our page views tag. This tag will fire through to GA and will be visible under **Screens** in the **Real-Time** section.

&nbsp;

![Tags Step 2]({{page.res}}/tags_2.png)

&nbsp;

When we create a new tag the first choice we are greeted with is what analytics product we would like to use. In our case this is easy, **Google Analytics**

We then need to configure our tag...

&nbsp;

![Tags Step 3]({{page.res}}/tags_3.png)

&nbsp;

Just like with variables and triggers, the name up the top is our own internal reference to the tag. More importantly we see our **Tracking ID** field. This is where we tell GTM that this tag will be sent to the GA instance we made earlier. If we wanted to also send the same event to another instance all we have to do is copy this tag and change that field. The final field is **Track Type**, this defaults to "App View" which is GTM's name for a page view, so conveniently for us the work is already done. We just need to click **Continue** and attach a trigger to our tag.

&nbsp;

![Tags Step 4]({{page.res}}/tags_4.png)

&nbsp;

Our dialog here is pretty simple. We can attach as many triggers as we want. In our case however we only want the page view trigger.

&nbsp;

~~~
Note: A tag will activate if any one of its attached triggers fires. Attaching multiple triggers does not mean your tag holds out until all triggers are true. There is no support for this, you would need to make a third trigger with all the constraints of the other two. However if more than one trigger is true in a single event, your tag will still only fire the once.
~~~

&nbsp;

So lets take a look at our final tag.

&nbsp;

![Tags Step 5]({{page.res}}/tags_5.png)

&nbsp;

Looks pretty good, all we need to do it click **Create Tag** and we are up and running with page tracking. All a client needs to do is send an event to us with the name **pageView** and we will pass that along to GA with the app name, version code and all those other built in variables. In real time!

&nbsp;

![Tags Step 6]({{page.res}}/tags_6.png)

&nbsp;

So now we have our page view tag. Lets create a tag for all the click events our client sends to us. This will look slightly different but isn't much harder. We will skip over choosing the analytics tool and just jump straight into configuring the tag.

&nbsp;

![Tags Step 7]({{page.res}}/tags_7.png)

&nbsp;

So, this time we are tracking an **Event**, to be clear, this  is not the same as what we send from the client and use in our triggers. This is a Google Analytics Event, they are visible under **Events** in the **Real-Time** section in GA. 

GA events have four key fields: **Category**, **Action**, **Label** and **Value**. These fields are viewed in GA like a hierarchy with **Category** at the top and **Value** at the bottom.

We can pass any data we like in these fields `with the exception of value, value must be a number, if it isn't a number you will not get errors but you also won't see your events show up in GA` 

&nbsp;

![Tags Step 8]({{page.res}}/tags_8.png)

&nbsp;


In my case I want a "User Action" **Category** because later I might have a "Push" category and a "Network Event" category. It would be nicer to bundle all my user interactions under the one category instead of having "Click", "Swipe", "Screenshot" etc all cluttering up my top level. Also note, this would be a good place to have a **Constant**

With my **Action** and **Label** fields you will see we can easily combine variables and text to provide context, creating more meaningful data to send through to the analytics.

I set my **Value** to empty because for our purposes there is no reason to send numbers through to GA.

Finally the **Non-Interaction Hit** field is a simple boolean. It is a way of indicating to GA that this event was caused by a deliberate user action and not something automated. It defaults to false as 90% of the time we are tracking user actions.

Lets tie this to our click trigger and hit the go button!

&nbsp;

![Tags Step 9]({{page.res}}/tags_9.png)

&nbsp;

And there we have it! Our two tags capable of tracking every page view and click event in a whole app. Very powerful stuff and we haven't even broached the advanced options. All we need to do now is publish this container and we can turn our attention to the final piece of the puzzle, the client.

&nbsp;

####Step Four: Publishing

Time to click that little red button that has been watching over us this whole time.

&nbsp;

![Publish Step 1]({{page.res}}/publish_1.png)

&nbsp;

If you click on the dropdown you may see there are actually two options available to you. The only difference between them is one makes your changes live while the other does not. Both save your current version and both give you a new working version to play with. If you do opt for the blue button, you can always go and set your saved version to live later on when you are ready.

&nbsp;

![Publish Step 2]({{page.res}}/publish_2.png)

&nbsp;

But for us, we just want to hit publish.

&nbsp;

![Publish Step 3]({{page.res}}/publish_3.png)

&nbsp;

We will see a nice little confirmation window. We can just click **Publish Now** but remember, this is your last chance to back out. Once you hit publish you are live for a minimum of 24 hours the second a client happens to pick up your change before you realise you made a mistake. The more users you have the more likely this is, so be careful.

&nbsp;

![Publish Step 4]({{page.res}}/publish_4.png)

&nbsp;

TADA! We are done! If we go to the **Versions tab** we can see version 1 of our container up and running.

&nbsp;

![Publish Step 5]({{page.res}}/publish_5.png)

&nbsp;

Our final step is to download the container file for our client. This is the file our clients will start with until they pull down the latest container. In our case there isn't a new one to pull down which makes life a lot easier! But if we then release our app and want to change the analytics a month later we can publish our new container and the clients will pick it up. But be aware, any newly downloaded clients will still start on the old container before they get to update.

&nbsp;

![Publish Step 6]({{page.res}}/publish_6.png)

&nbsp;

---

## Summary

And this is the GTM configuration done! And as we can see, its all pretty straight forward. We have glossed over a lot of advanced features but you really don't need them for the majority of analytics solutions. All that is left is setting up our Android app so we can see our data flowing through our brand new container!

&nbsp;

**[Check out the next article, where we learn how to hook a client into our container!][Part 3]**

[GTM]: https://tagmanager.google.com/
[GTM v4]: https://developers.google.com/tag-manager/android/v4/
[GA]:www.google.com.au/analytics/
[Part 1]: {% post_url 2015-09-21-google-tag-manager %}
[Part 3]: {% post_url 2015-09-30-google-tag-manager %}