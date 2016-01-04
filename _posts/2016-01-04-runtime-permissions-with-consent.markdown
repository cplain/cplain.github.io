---
layout: post
title: Runtime Permissions With Consent
subtitle: I made a library!
tags: Android Permissions Runtime
author: Coby Plain
res: /assets/2016-01-04-runtime-permissions-with-consent
---

---

Runtime permissions have been done to death, we have been scared of them, excited by them and finally just over them. However despite this there is no nice way manage them. Google has provided a number of useful methods in the support library, but as far as using them and managing the logic flow you are on your own. I recently had a look at a number of libraries out there and in playing around with them I noticed a few things I wasn't so happy with. So I developed a utility for my own use that I think is a nicer take on permissions. I quite like my little utility so today I decided to make it public! 

<!--end_excerpt-->

---

## Introducing Consent


Consent is here to keep things simple. It provides a clean interface that keeps everything together. I won't go over the permissions api or how it works, there is more than enough fantasic material out there on how it all works which I highly recommend you read. Knowing the api however, one of my biggest gripes with it is that it splits up the positve and negative user flows. 

&nbsp;

Splitting up your code like that often makes thing a lot harder to manage. It also means you have to spend effort determining where results came from and now best to manage them. In an effort to fix this, Consent wrangles the flows together into three clear callbacks, `onPermssionsGranted()`, `onPermissionsDeclined()` and `onExplanationRequested()`.

&nbsp;

Effort has also been taken to provide you with as many helpers, default options and utilties as possible so that you can get down to writing the good stuff.

&nbsp;

Using consent is simple:

&nbsp;

{% highlight java %}
Consent.request(new PermissionRequest(this, READ_CONTACTS, ACCESS_FINE_LOCATION) {
    @Override
    protected void onPermissionsGranted() {
        // perform desired action
    }

    @Override
    protected void onPermissionsDeclined(@NonNull DeclinedPermissions declinedPermissions) {
        // perform rejected action
        // Note: DeclinedPermissions contains all the permissions rejected in this request as well as various
        // helpers, such as ones to extract permissions rejected because the user selected "Never Ask Again"
    }

    @Override
    protected AlertDialog.Builder onExplanationRequested(@NonNull AlertDialog.Builder builder, @NonNull String[] permissionsToExplain) {
        // return the builder if you want to show a dialog or null if you want to handle the explanation yourself
        // if you are handling the explanation yourself be sure to call onExplanationCompleted() when you want the request to continue
        return builder.setTitle(R.string.explanation_title_multiple).setMessage(R.string.explanation_message_multiple);
    }
});
{% endhighlight %}

&nbsp;

And due to how Android processes permissions you also need to add the following code (which can be placed in a base activity or fragment if it suits you)

&nbsp;

{% highlight xml %}
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    Consent.handle(requestCode, permissions, grantResults);
}
{% endhighlight %}

&nbsp;

{% raw %}
Note: Currently Consent works via the Activity (which should extend from AppCompatActivity) this is because of some known defects in the Fragment api which I still consider worth covering. If you wish to use Consent in a Fragment you simply need to call through to `getActivity()`.
{% endraw %}

&nbsp;

That's all there is! Simple and friendly! You can download it today from jcenter using:

{% highlight java %}
compile 'com.seaplain:consent:1.0.1'
{% endhighlight %}

&nbsp;

and feel free to check out the source on [GitHub][GitHub]

[GitHub]: https://github.com/cplain/consent

