---
layout: post
title: Improving Gradle Flavours / Variants
subtitle: Still doesn't come in chocolate
tags: Android Gradle Flavours Variants Versioning
author: Coby Plain
res: /assets/2015-10-21-improving-gradle-flavours
hide: true
---

---

Gradle is one of the most useful, powerful and misused tools at an Android developer's disposal. Thankfully there are plenty of smart people out there that have written fantastic material on how to improve our scripts. Amongst these posts I have searched for one tip in particular, how to better manage flavours and variants, alas the article has never come. So recently I decided just couldn't take it anymore and I went on a quest to solve it for myself. 

<!--end_excerpt-->

&nbsp;

> I'm going to start from the very beginnings, but if you are finding things a bit too simple free to jump down to where things get more complex.

&nbsp;

---

## What are we trying to solve?

Interestingly enough our problem starts with a pretty cool solution. It is more than common for us to need different configurations of our apps for various reason. Often these will have different environments, features, timings and so on. 

&nbsp;

More often than not you will want this in two dimensions. Say you want to test your app pointing to three different environments, you will need to see how they handle in a full release configuration (obfuscation, cert pinning, tighter timeouts, etc) but having those options always on will hamper your actual development. Meaning you want to be able to switch between the different environments, but also between release and debugging configurations.

&nbsp;

Thankfully Gradle has us covered with **variants** and **flavours**. 

&nbsp;

---

## A simple approach

When producing builds we can create our app in any variant and flavour combination we would like. Depending on what configuration we have set, Gradle will populate our `BuildConfig` class with a set of default values. Below I have my variant set to `debug` and my flavour set to `staging`:

&nbsp;

{% highlight java %}
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.seaplain.android.example.staging.debug";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "staging";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0.0-staging";
}
{% endhighlight %}

&nbsp;

This is your default `BuildConfig` class, at first glance we can see it is pretty useful. We have access to our version names and codes as well as what flavour/variant combo we have active. Now if we desired, we could control sections of our code through `if` or `switch` statements:

&nbsp;

{% highlight java %}
final String endpoint;
if (BuildConfig.FLAVOUR.equals("staging")) {
    endpoint = "https://api.staging.seaplain.com";
} else {
    endpoint = "https://api.seaplain.com";
}
{% endhighlight %}

&nbsp;

**But thats awful you should be saying, don't do that!!!** And you would be right, now our code is heavily tied up with our build configuration. Using this method we have no idea what values are linked to what configuration without extensive knowledge of the code. When we go to add or remove a new configuration we have no idea what pain we could unleash. There is a better way! 

&nbsp;

---

## A step in the right direction

Gradle provides us with the exceptionally useful `buildConfigField` and `resValue` methods. These allow us to set our own values in `BuildConfig`! 

&nbsp;

In our module's `build.gradle` file:

{% highlight java %}
productFlavors {
    staging { 
        buildConfigField "String", "BASE_ENDPOINT", "https://api.staging.seaplain.com"
        resValue "string", "app_name", "Example App Stage"
    }
    prod { 
        buildConfigField "String", "BASE_ENDPOINT", "https://api.seaplain.com"
        resValue "string", "app_name", "Example App"
    }
}
{% endhighlight %}

&nbsp;

Which will generate for `staging`:

{% highlight java %}
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.seaplain.android.example.staging.debug";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "staging";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0.0-staging";
  // Fields from product flavour: staging
  public static final String BASE_ENDPOINT = "https://api.staging.seaplain.com";
}
{% endhighlight %}

&nbsp;

Much nicer! Now our code can now always reference `BuildConfig.BASE_ENDPOINT` without worrying what configuration we have set! 

&nbsp;

You may notice we also created an app name field which we cant see in `BuildConfig`. As we used `resValue` is it assigned a resource id and now lives with the rest of our resources. As we set the type to `string` we access it via `@string/app_name`. 

&nbsp;

Notice here that we can define the type of variable we want to set with both methods. We are limited to the different resource types for `resValue`. But when using `buildConfigField` we can use something a bit more advanced if required, simply use the fully qualified name of whatever object or constant you desire.

&nbsp;

E.g

{% highlight java %}
buildConfigField "java.util.concurrent.TimeUnit", "NETWORK_TIMEOUT_UNIT", "java.util.concurrent.TimeUnit.MINUTES"
{% endhighlight %}

Produces the perfectly usable:

{% highlight java %}
public static final java.util.concurrent.TimeUnit NETWORK_TIMEOUT_UNIT = java.util.concurrent.TimeUnit.MINUTES;
{% endhighlight %}

&nbsp;

We are definitely making progress! We have one nice clean place where we can see all our values and settings for our different builds. 

&nbsp;

But there is a problem to this solution as well. In MVP architecture it is very common to make use of Gradle modules to segment code. What if we have multiple gradle modules with different settings? What if we need to reuse one of our values in two different fields or across two different modules? 

&nbsp;

Different modules have their own variants and flavours, hence their own `BuildConfig` class. So we would need to set up the same code in each module we added. In doing this, some fields may be the same across modules and others new for each module we add. Now we not only have duplication, we start to see a spread in our configurations again. We need to look at the `build.config` of every module just to see what each configuration has in store. We can't change one value without having to check everywhere. We are almost back where we started!

&nbsp;

---

## The current state of affairs

To solve this problem it is time to take a step up to our top-level `build.gradle` file. In our top level we can define variables that each module can reference when setting up the flavours and variants.

&nbsp;

For our current simple example (adding one more field for illustration), our top level would have:

{% highlight java %}
import java.util.concurrent.TimeUnit // Only needed to dodge using the fully qualified name in our gradle file, you may or may not want to do this

allprojects {
    ext {
        config = [
            // Where you should set up things like: applicationId, versionCode, versionName, compileSdkVersion, buildToolsVersion, etc
        
            // Staging
            stagingBaseEndpoint :  "https://api.staging.seaplain.com",
            stagingAppName :  "Example App Stage",
            stagingTimeout :  "${TimeUnit.MINUTES.toMillis(3)}",

            // Production
            prodBaseEndpoint :  "https://api.seaplain.com",
            prodAppName :  "Example App"
            prodTimeout :  "${TimeUnit.SECONDS.toMillis(30)}",
        ]
    }
}
{% endhighlight %}

&nbsp;

And in our app module:

{% highlight java %}
productFlavors {
    staging { 
        buildConfigField "String", "BASE_ENDPOINT", config.stagingBaseEndpoint
        resValue "string", "app_name", config.stagingAppName
    }
    prod { 
        buildConfigField "String", "BASE_ENDPOINT", config.prodBaseEndpoint
        resValue "string", "app_name", config.prodAppName
    }
}
{% endhighlight %}

&nbsp;

And say we needed the endpoint in another module too, plus the new timeout field: 

{% highlight java %}
productFlavors {
    staging { 
        buildConfigField "String", "BASE_ENDPOINT", config.stagingBaseEndpoint
        buildConfigField "long", "TIMEOUT", config.stagingTimeout
    }
    prod { 
        buildConfigField "String", "BASE_ENDPOINT", config.prodBaseEndpoint
        buildConfigField "long", "TIMEOUT", config.prodTimeout
    }
}
{% endhighlight %}

&nbsp;

Now despite having two modules we can look at our top-level `build.gradle` to see all our configuration options. We have a single source of truth again! A massive improvement!

&nbsp;

This is where I started when I began my quest. This is how I, and every developer I have worked with, set up build configurations. But there is something... ugly.. about this solution and I have never liked it. If this was Java code, not a single one of us would settle with this solution. And the key reason is duplication. We need to have a {flavour}Field for each different configuration and on top of that we need to maintain our lists of flavours and variants in each file. If I want to add another flavour to this mix I need to go into every `build.gradle` file and update it. For our setup here this isn't the worst of of messes, but in commercial applications we soon see this degrade into pure awful.

&nbsp;

> One other issue I have with this solution is that it is up to us to keep our values neat and in order. Any dev could come in and stick another new value completely outside our neat commented blocks and suddenly things are a lot harder to read. Now this should be caught by code reviews etc, but you can't guarantee you will be around (holidays, other projects, etc) to enforce it when the time comes.

&nbsp;

---

## A better way

In an ideal world I could change `stagingBaseEndpoint` and all other variants of that field into `baseEndpoint` and have each flavour uses the right one, forcing everyone to follow a neat grouping in the process. Well actually in an ideal world the fact that there is a `baseEndpoint` for in a unique group should create a flavour for me! But that would be ridiculous.... Wouldn't it?

&nbsp;

So step one: get a consistent name and grouping. This actually isn't too hard, we simply need to introduce another layer of arrays in our `config`

&nbsp;

{% highlight java %}
import java.util.concurrent.TimeUnit // Only needed to dodge using the fully qualified name in our gradle file, you may or may not want to do this

allprojects {
    ext {
        config = [
            // Where you should set up things like: applicationId, versionCode, versionName, compileSdkVersion, buildToolsVersion, etc
        
            staging = [
              baseEndpoint :  "https://api.staging.seaplain.com",
              appName :  "Example App Stage",
              timeout :  "${TimeUnit.MINUTES.toMillis(3)}",
            ],

            prod = [
              baseEndpoint :  "https://api.seaplain.com",
              appName :  "Example App",
              timeout :  "${TimeUnit.SECONDS.toMillis(30)}",
            ]
        ]
    }
}
{% endhighlight %}

&nbsp;

Which we use like:

&nbsp;

{% highlight java %}
productFlavors {
    staging { 
        buildConfigField "String", "BASE_ENDPOINT", config.staging.baseEndpoint
        resValue "string", "app_name", config.staging.appName
    }
    prod { 
        buildConfigField "String", "BASE_ENDPOINT", config.prod.baseEndpoint
        resValue "string", "app_name", config.prod.appName
    }
}
{% endhighlight %}

&nbsp;

Brilliant! Thats a nice step forward. We have forced future developers (ourselves included) to keep our related configurations together. And in doing so have made our variables smaller, more legible and uniform which I am all for. But this isn't perfect.

&nbsp;

We still have this weird duplication in each of our flavours. They are doing exactly the same thing with just a minor variation (the array being used). In Java we would immediately make a method to solve this, but things are a little different here. However there is a very nifty solution:

&nbsp;

> Note: For this solution I am going to add in a few more values just to give you some ideas of what we can do

&nbsp;

First lets break out our flavours from any other values we may have (you would do the same for variants):

{% highlight java %}
import java.util.concurrent.TimeUnit // Only needed to dodge using the fully qualified name in our gradle file, you may or may not want to do this

allprojects {
    ext {
        config = [
            applicationId : "com.seaplain.android.example.staging.debug",
            versionName : "1.0.0",
            // Other values, versionCode, compileSdkVersion, buildToolsVersion, etc
        ]

        flavours = [        
            staging = [
              applicationId : config.applicationId + ".stage",
              versionName : config.versionName + "-stage",
              baseEndpoint :  "https://api.staging.seaplain.com",
              appName :  "Example App Stage",
              timeout :  "${TimeUnit.MINUTES.toMillis(3)}",
            ],

            prod = [
              applicationId : config.applicationId,
              versionName : config.versionName,
              baseEndpoint :  "https://api.seaplain.com",
              appName :  "Example App",
              timeout :  "${TimeUnit.SECONDS.toMillis(30)}",
            ]
        ]
    }
}
{% endhighlight %}

&nbsp;

And now we have an array that solely defines each flavour, lets loop through them to build our config.

&nbsp;

{% highlight java %}
productFlavors {
    flavours.each { name, flavour ->
        "$name" {
            applicationId = flavour.applicationId
            versionName = flavour.versionName
            buildConfigField "String", "BASE_ENDPOINT", flavour.baseEndpoint
            resValue "string", "app_name", flavour.appName
        }
    }
}
{% endhighlight %}

&nbsp;

**WOAH WAIT!** What just happened there? Things just changed very radically indeed! Now for each unique array in our top level `flavours` we are assigning the name of the array to `name` and the contents to `flavour`. We then create ourselves a flavour **inside the loop** using `name`. Inside this new flavour we use `flavour` to fill the fields.

&nbsp;

So what does this mean? Well now in each module's `build.gradle` we can set up this loop, using just the configuration values we need. Then whenever we need a new flavour we can just add a new entry in `flavours` and **without any modification of any other `build.gradle`** our flavour will be set up for us! You can't get much cleaner!

&nbsp;

---

## Summary

This has been an area that has bothered me for a long time, and this solution goes a long way to cleaning up our scripts. For larger applications this can remove significant chunks of duplicated code whilst also providing us with increased flexibility and readability. We should all be making the most of what variants and flavours have to offer, they are incredibly powerful tools that are now hopefully one step friendlier.

&nbsp;

> A quick final note: I have considered taking this a step further to start sending through the types and names for each field. Or possibly even looping through each flavour once this is in place, thus meaning our module's `build.gradle` files needn't be modified each time we add a new field. In my opinion this is overkill and the headache you will create in dealing with edge cases and unneeded values bleeding across modules etc is simply not worth it.