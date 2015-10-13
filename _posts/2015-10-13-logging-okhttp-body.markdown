---
layout: post
title: Getting Your Logs Back In Retrofit 2
subtitle: Hey I was using those
tags: Android OkHttp Networking Logging Network Data 
author: Coby Plain
res: /assets/2015-10-13-logging-in-okhttp
---

---

A final version of Retrofit 2 is fast approaching, and with a number of nice new features we are all excited to start the shift. However amongst the excitement there is one looming issue; we are about to loose all our logging! But fear not! I have you covered.

<!--end_excerpt-->

&nbsp;

---

## Our Weapon of Choice

So to solve our problem we need the right tool; time to meet our friend the `Interceptor`.

&nbsp;

`Interceptor` is an OkHttp interface that, as the name suggests, allows us to intercept requests and responses. This gives us the ability to both read and modify our outgoing and incoming calls. There are a number of uses for this from applying headers to changing data. Today however we will focusing on how it can give us back our logging. So lets get started!

&nbsp;

An `Interceptor` can be applied to our `OkHttpClient` like this:

{% highlight java %}
OkHttpClient okHttpClient = new OkHttpClient();
okHttpClient.networkInterceptors().add(new LoggingInterceptor());

// Create Retrofit using .setClient(new OkClient(client))
{% endhighlight %}

&nbsp;

Looks pretty straightforward so far. I've created a `LoggingInterceptor`, a custom class, which I apply to my OkHttp client. The client will then be used later when I initialise Retrofit. For most readers this is nothing new, all the real fun will happen inside `LoggingInterceptor`.

&nbsp;

> As a note: `okHttpClient.networkInterceptors()` will return a list which will allow us to add multiple interceptors. This means we can avoid creating one interceptor to solve all our problems and instead focus on creating tight, focused, reusable interceptors.

&nbsp;

---

## Lock and Load

Now let's take a look inside our `LoggingInterceptor`:

{% highlight java %}
public class LoggingInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        
        // Log outgoing requests here

        Response response = chain.proceed(request);

        // Log incoming responses here

        return response;
    }
}
{% endhighlight %}

&nbsp;

The `Interceptor` interface requires just one method, `intercept()`. This method is passed a `Chain`, which we can use to access our requests and responses. 

&nbsp;

From the `Chain` we can pull a `Request` object using `chain.request()`. `Request` opens up access to the body, url, headers, method type (get, post, put, etc) and so on. Once we are finished we can call `chain.proceed(request)` which will fire the request off to the network stack and eventually return with a `Response`. It is important to note that we can send any `Request` object we felt like, it needn't have anything to do with the original request in the chain.

&nbsp;

From `chain.proceed(request)` we retrieve our `Response`. Similar to `Request` this will give us access to headers, bodies, error codes and so on (it doesn't contain the url or request method). When we are finished we `return` the `response` to the rest of the network stack (again this can be a different response).

&nbsp;

---

## Target Practice

So we know the foundations, but there are still a few pitfalls ahead of us. Time to take our new skills for a test-drive.

&nbsp;

Lets say we have a requirement to log all our network errors. But unlike a typical dump we want only a one field from our response. Thankfully all our errors *should* return a body in a standard format which contains amongst other things a message we want to log.

&nbsp;

Shockingly our problem seems a little contrived, but lets see what kind of issues this may draw out.

&nbsp;

First we will need a class to define our error:

{% highlight java %}
public class ErrorResponse {
    @SerializedName("ErrorMessage") private String mErrorMessage;
    // Other fields

    public String getErrorMessage() {
        return mErrorMessage;
    }

    // Other methods
}
{% endhighlight %}

&nbsp;

This looks like every other class using Gson so I'm not going to waste time here, but now we know what we are dealing with.

&nbsp;

Next we will need a method that takes our values and does "something" with them:

{% highlight java %}
private void trackError(Request request, String cause, ErrorResponse errorResponse) {
    trackError(request, code, errorResponse.getErrorMessage());
}

private void trackError(Request request, String cause, String message) {
    String url = request.urlString();
    String method = request.method();

    // Do something with url, method, cause and message
}
{% endhighlight %}

&nbsp;

Again nothing earth-shattering here, what we actually do with our values could be anything, print them to the console, log them, send them to analytics and so on. That isn't the important part. The important part is we are using our `Request` for the url and the method, but will be using the body of our `Response` for the message.

&nbsp;

> Note: we have overloaded `trackError()` this will come into play shortly.

&nbsp;

We now have all the components we need, time to get back to our `Interceptor`:

{% highlight java %}
public class LoggingInterceptor implements Interceptor {
    Gson mGson = new Gson();

    @Override
    public Response intercept(Chain chain) throws IOException {
        final Request request = chain.request();
        Response response = chain.proceed(request);

        if (!response.isSuccessful()) {
            String cause = String.valueOf(response.code());
            ErrorResponse errorResponse = mGson.fromJson(response.body().string(), ErrorResponse.class);

            trackError(request, cause, errorResponse);
        }
        
        return response;
    }
}
{% endhighlight %}

&nbsp;

When the response comes in we are using `response.isSuccessful()` to determine if we have received an error or not. This will be true only if we receive a 200 class response code. From there we retrieve the response code via `response.code()` and parse it into a `String` (this will make more sense later). We then take our body and parse it via Gson into an `ErrorResponse`. Finally we push all our values through to our tracking method.

&nbsp;

At first glance this code looks fine but there are three key mistakes, **one with serious consequences.**

&nbsp;

---

## So we shot ourselves in the foot...

First let's start with the easiest problem, which is `chain.proceed(request)` throws an `IOException`. This exception could actually be any number of `IOException` subclasses, most notably `SocketTimeoutException` which will be thrown if your request times out. 

&nbsp;

This isn't a big deal as `intercept()` also throws the exception, but it does mean we are missing out on potentially useful data. For a complete logging solution we will want to catch these exceptions, log them, and then re-throw.

&nbsp;

Let's put in some simple tracking for this.

{% highlight java %}
public class LoggingInterceptor implements Interceptor {
    Gson mGson = new Gson();

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        Response response;

        try {
            response = chain.proceed(request);
             if (!response.isSuccessful()) {
                String cause = String.valueOf(response.code());
                ErrorResponse errorResponse = mGson.fromJson(response.body().string(), ErrorResponse.class);

                trackError(request, cause, errorResponse);
            }
        } catch (SocketTimeoutException e) {
            trackError(request, "Timeout", e.getMessage());
            throw e;
        } catch (IOException e) {
            trackError(request, "Device Connection Failure", e.getMessage());
            throw e;
        }

        return response;
    }
}
{% endhighlight %}

&nbsp;

 We are catching `SocketTimeoutException` and passing through some custom values for a network timeout. All other possible errors we are tracking as a generic device issue (although we could keep going). This opens up a whole new class of error, definitely useful in our logs!

&nbsp;

> Note: Hopefully the earlier decision to use `String` for our "cause" makes a lot more sense now as we are not just passing int based error codes.

&nbsp;

---

## And the leg...

The next biggest issue also stems from an oversight. While we may have been promised that all our errors will have a body matching `ErrorResponse`, what happens if they don't? If parsing fails Gson will throw an `IOException` which will cause our code to fall into the `Device Connection Failure` catch clause. That means we miss the real cause of our problem and instead get a parsing exception in our logs. Not particularly useful...

&nbsp;

So because our `intercept()` method is getting a little crowded, lets replace our not at all useful `trackError(Request request, ErrorResponse errorResponse)` with something a bit more deliberate:

{% highlight java %}
private void trackError(Request request, Response response) {
    String cause = String.valueOf(response.code());
    final String errorMessage;

    try {
        ErrorResponse errorResponse = mGson.fromJson(response.body().string(), ErrorResponse.class);
        errorMessage = errorResponse.getMessage();
    } catch (NullPointerException | IOException e) {
        errorMessage = "Unknown error";
    }

    trackError(request, cause, errorMessage);
}
{% endhighlight %}

&nbsp;

And in `intercept()`

&nbsp;

{% highlight java %}
@Override
public Response intercept(Chain chain) throws IOException {
    ** same as before **
    try {
        response = chain.proceed(request);
        if (!response.isSuccessful()) {
            trackError(response, request);
        }
    } 
    ** same as before **
}
{% endhighlight %}

&nbsp;

So here we are using a `try/catch` block to protect against conversion issues, in the case we get one, we track an unknown error. Now our logging will focus on the real network problem and not the parsing!

&nbsp;

---

## And the chest...

The last two problems have been oversights that lead to us missing out on useful information. The final one however is a lot more serious. **We are not allowed to read a response body more than once.** By calling `response.body().string()` in our interceptor, Retrofit will not be able to read the body later and instead will throw an error. We have just killed all our network activity (most likely breaking our apps entirely) in one fell swoop!

&nbsp;

This is a big problem, but fortunately there is a solution! We need to return a new request with a shiny new body. However we must make sure to be careful when we do this. Our current code is in a `try/catch` block, for safety's sake we should make sure to put this new response in the `finally` clause.

&nbsp;

Lets look at our updated `trackError()`:

{% highlight java %}
private Response trackError(Request request, Response response) {
    String responseBodyString = response.body().string();
    String cause = String.valueOf(response.code());
    final String errorMessage;
    final Response response;

    try {
        ErrorResponse errorResponse = mGson.fromJson(responseBodyString, ErrorResponse.class);
        errorMessage = errorResponse.getMessage();
    } catch (NullPointerException | IOException e) {
        errorMessage = "Unknown error";
    } finally {
        ResponseBody body = ResponseBody.create(response.body().contentType(), responseBodyString);
        response = response.newBuilder()
                        .body(body)
                        .build();
    }

    trackError(request, cause, errorMessage);
    return response;
}
{% endhighlight %}

&nbsp;

And in `intercept()`

&nbsp;

{% highlight java %}
@Override
public Response intercept(Chain chain) throws IOException {
    ** same as before **
    try {
        response = chain.proceed(request);
        if (!response.isSuccessful()) {
            response = trackError(response, request);
        }
    } 
    ** same as before **
}
{% endhighlight %}

&nbsp;

So now we are creating a new `Response` which we will use as a replacement for our used one. Note that while `response.newBuilder()` will copy across the body, thanks to the magic of pointers it will be the same unusable body as in our existing response. That is why we make pains to recreate a `ResponseBody` object from the `responseString`.

&nbsp;

And crisis averted! We have saved ourselves a world of hurt from a seemingly harmless logging solution.

&nbsp;

---

## Summary

Retrofit is a powerful tool, it has changed the way most Android developers approach their networking stack. We should all be excited for what the future has in store without worry. Hopefully this article has helped arm you to deal with a range of potential problems and empower you to create your own perfect logging solution.

&nbsp;

Our final `Interceptor` can be seen below:

{% highlight java %}
public class LoggingInterceptor implements Interceptor {
    Gson mGson = new Gson();

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        Response response;

        try {
            response = chain.proceed(request);
            if (!response.isSuccessful()) {
                response = trackError(response, request);
            }
        } catch (SocketTimeoutException e) {
            trackError(request, "Timeout", e.getMessage());
            throw e;
        } catch (IOException e) {
            trackError(request, "Device Connection Failure", e.getMessage());
            throw e;
        }

        return response;
    }

    private Response trackError(Request request, Response response) {
        String responseBodyString = response.body().string();
        String cause = String.valueOf(response.code());
        final String errorMessage;
        final Response response;

        try {
            ErrorResponse errorResponse = mGson.fromJson(responseBodyString, ErrorResponse.class);
            errorMessage = errorResponse.getMessage();
        } catch (NullPointerException | IOException e) {
            errorMessage = "Unknown error";
        } finally {
            ResponseBody body = ResponseBody.create(response.body().contentType(), responseBodyString);
            response = response.newBuilder()
                            .body(body)
                            .build();
        }

        trackError(request, cause, errorMessage);
        return response;
    }

    private void trackError(Request request, String cause, String message) {
        String url = request.urlString();
        String method = request.method();

        // Do something with url, method, cause and message
    }
}
{% endhighlight %}