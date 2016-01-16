---
layout: post
title: Animating to Transparency
subtitle: A quick note on a nifty trick
tags: Android Animation Color Colors Transparency Black 
author: Coby Plain
res: /assets/2016-01-16-animating-to-transparency
---

---

Today I was playing with a simple little animation, probably one we have all attempted. I was animating the colored background of a button through to transparency. Typically this has a nasty little problem, as you animate you will see the color darken through black right before reaching transparent. It often looks awful and most cases I have seen developers simply swap to animating the alpha of a whole view to get around it. However this isn't always ideal. Earlier today another solution occured to me that I thought was worth sharing.

<!--end_excerpt-->

---

Disclamer: To be honest I have no idea if this is a well known trick and I am out of the loop, or if this is something new to most. But in the case it is the latter I figure there is no harm in sharing.

&nbsp;

This whole trick is dependant on the value of `Color.TRANSPARENT` (also `android:color/transparent`). If we have a quick look we can find it equates to the hex value `#00000000`. 

&nbsp;

>For those unfamiliar with hex values (which I dobut is many), the first two characters are the hex value of the alpha channel (`00` to `FF` which in decimal is `0` to `255`) which is often left out implying `FF`, the next two are the red channel, then green and finally blue.

&nbsp;

Knowing this we have all we need. `Color.TRANSPARENT` has no alpha value, and hence in most cases the rest of the components are irrelvant. But while animating, the `ArgbEvaluator` will be shifting the alpha channel of our starting color towards transparent and it will also be shifting the rest of the components towards `#000000`. For those who haven't caught on, `#000000` is black, and this is where our problem lies. While our animating color is not yet fully transparent, the rest of its channels are being shifted to black.

&nbsp;

So the solution is simple, if we use our own transparent color, we can control what the `ArgbEvaluator` animates towards before reaching transparent. As an example:

&nbsp;

{% highlight xml %}
    <color name="starting_color">#7ED321</color>
    <color name="appropriate_transparent">#007ED321</color>
{% endhighlight %}

&nbsp;


When it occured to me I though this was a nice little trick. Hopefully someone will find useful!