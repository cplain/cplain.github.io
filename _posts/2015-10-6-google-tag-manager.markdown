---
layout: post
title: Google Tag Manager - Bonus Round
subtitle: Some extra goodies for our bags
tags: Android GTM Analytics Google_Analytics Google_Tag_Manager
author: Coby Plain
res: /assets/2015-10-06-google-tag-manager
hide: true
---

---

[So we have GTM up and running!][Part 3] But now what? There must be more right? This article will take a look at some additional features we can use in GTM. there is no way I can cover everything, so I have picked out three interesting ones for us to look at.

<!--end_excerpt-->

&nbsp;

###Series Overview

**Google Tag Manager** (GTM) is a layer of separation over other analytics suites. It serves as a distribution gateway that will receive notifications from clients and then uses its own logic to construct and fire tracking events at any number of analytics tools. We will be taking a look at GTM over the next few articles, and by the end of this series we will have a working Tag Manager implementation in Android! If you are looking into GTM for iOS, the first few articles will be identical for you, it is only the client-side implementation that changes. For web, the core concepts and meanings are the same.

&nbsp;

---

###Extra Data

Sometimes having simply a category, action, label and value just isn't enough. GA has a wealth of additional dimensions, a lot are set for us but a lot arent. Wouldn't it be great if we could set whatever ones we desired in our tags too? Well, very easily, we can. This is probably one of the simplest enhancements you can make to your tracking and could be very useful when it comes to analysing data and making decisions. You don't want to be in a position where you have a question but no data to answer it.

&nbsp;

#### How to do it

First things first, we need to send through the data from the client. This will look exactly the same as what we have done before, we are just adding new data layer variables.

&nbsp;

Remember our click tracking method from before?

{% highlight java %}
protected final void trackClick(String clickTarget, String clickValue) {
    mTagManager.getDataLayer().pushEvent(GTM_EVENT_CLICK, DataLayer.mapOf(
            GTM_KEY_CLICK_TARGET, clickTarget,
            GTM_KEY_CLICK_VALUE, clickValue
    ));
}
{% endhighlight %}

&nbsp;

This has been a very useful method for us so far. But there is one big flaw. What if we had two buttons with the same logical name? What if they looked and acted the same but were on different pages? How would we know which event belonged to each element? Well... We wouldn't. There isn't currenlty a way for us to distingush between the two. So how about we solve that problem? It would be great if we could set GA's `screenName` dimension in our tag. So to help out GTM lets send through our page name using the `getName()` method [we set up last time][Part 3].

&nbsp;

So what will the new method look like?

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

---

###Campain Tracking 

#### Installs

#### App Opens


&nbsp;

---

###Crash Tracking

#### Caught

#### Uncaught



&nbsp;

---

## Summary

So with very little code and very little effort we have a powerful, flexible analytics solution. What more could you want? Well surprisingly GTM can provide us with a lot more. There is a trove of value to be discovered in its advanced options and power usages.

But hopefully for now this series has you equipped to handle the basics and use this new insight to better your products and brands. And as you push ahead with this new platform you can grow your skills and step bravely into deeper waters!

Good luck

[Dagger]: http://google.github.io/dagger/
[Crashlytics]: https://fabric.io/kits/android/crashlytics/summary
[Roboelectric]: http://robolectric.org/
[Part 1]: {% post_url 2015-09-21-google-tag-manager %}
[Part 2]: {% post_url 2015-09-25-google-tag-manager %}
[Part 3]: {% post_url 2015-09-30-google-tag-manager %}