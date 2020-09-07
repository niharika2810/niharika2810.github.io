<div style="text-align:center">
<h1> Pitfalls in Android WebView Implementation
</h1>
Blog by <a href="http://thedroidlady.com/">theDroidLady</a>
</div>
<br/>
<br/>
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/webview.gif">
</div>
<br/>
<br/>
## Introduction

In the current era of hybrid mobile architecture, WebView is extensively used hand in hand to deliver dynamic web content.

In this blog, I will be explaining the WebView pitfalls among different Android versions.

This thought of writing on WebView came when one of my colleagues reported some design issues for Android Kitkat and lower versions.

So I tried to understand why WebView is behaving differently in different OS versions. Then while reading about them, I found some other inconsistencies. So thought of sharing that through a blog.

Before going into much detail, lets first understand the term <b>WebView</b>.

### What is Android WebView?

<b>WebView</b> is an Android provided View like <b>TextView, EditText, Button etc</b>.

By using WebView, we can load the web(Html) pages as a part of our activity layout.

It uses <b>Webkit rendering engine</b> to display web pages and includes methods to navigate forward and backwards through a history, zoom in and out, perform text searches and more.

To learn more about WebView, read this [document](https://developer.android.com/reference/android/webkit/WebView).

Now, let’s talk about the pitfalls:

### 1) No WebView update in Kitkat and lower versions

1) In older versions of Android (4.3 and below), WebView uses code based on Apple’s Webkit — the same tech behind the Safari browser.

A [vulnerability](https://www.zdnet.com/article/half-of-all-android-devices-still-vulnerable-to-privacy-disaster-browser-bug/) was discovered in the WebView tool that affects anyone using Android 4.3 or earlier.

This vulnerability allows WebView to be exploited. When the vulnerability was reported, Google decided it would leave the patching of the 4.3 and earlier releases up to someone else([OEMs](https://us.nuumobile.com/android-oems-vs-odms-5-things-you-should-know/)).

Google fixed the Jelly Bean WebView issue with Android 4.4 KitKat patch.

 * The WebView shipped with [Android 4.4](https://www.androidcentral.com/android-kitkat) (KitKat) is based on the same code as Chrome(which uses Google’s Blink engine) for Android version 30. This WebView does not have full feature parity with Chrome for Android and is given the version number 30.0.0.0.

The updated WebView shipped with Android 4.4.3 has the version number <b>33.0.0.0</b>.

Then came a design issue in <b>Android KitKat</b> that left the OS exposed to newly-disclosed bugs in the <b>Chromium</b> project. Android 5.0 was released with WebView updates to fix it.


>In all versions of Android(KitKat and below), WebView has been bundled with Android firmware. From a security standpoint, that wasn’t good, because if a new flaw was found in WebView, the only way a fix would reach end-users was through the unwieldy process of delivering it from Google to OEMs and carriers and finally, if ever, to the end-user.

 * Then In [Android 5.0](https://www.androidcentral.com/lollipop), WebView was broken out as a separate app, presumably to allow timely updates through Google Play without requiring firmware updates to be issued. So, Now one can update WebView from Google Play.

### 2) WebView on Android 7 and above doesn’t render the page

```
webView.setWebViewClient(new WebViewClient() {

        public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
            //do something here
        }

        @Override
        public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
            return true;
        }
    });
    webView.loadUrl("https://google.com/");
```

This code perfectly works on <7.0 android version devices.

When you try to debug it, you will see it loads something(google URL appears in <b>shouldOverrideUrlLoading</b> method) but it shows nothing. No error logs will appear.

The <b>reason</b> is right here -

For Pre Android N,

```
@Deprecated
public boolean shouldOverrideUrlLoading(WebView view, String url) {
    return false;
}
```

and this is since Android N,

1) @return True if the host application wants to leave the current WebView and handle the URL itself, otherwise return false.

```
@Override
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
        return true;
    }
```

Solution -

Returning <b>false</b> from your code will solve your problem 😃.

### 3) Failed to load WebView provider: No WebView installed (Android Internal crash).

```
Caused by android.util.AndroidRuntimeException:
android.webkit.WebViewFactory$MissingWebViewPackageException:
Failed to load WebView provider: No WebView installed
at android.webkit.WebViewFactory.getProviderClass(WebViewFactory.java:371)
at android.webkit.WebViewFactory.getProvider(WebViewFactory.java:194)
at android.webkit.WebView.getFactory(WebView.java:2325)
at android.webkit.WebView.ensureProviderCreated(WebView.java:2320)
at android.webkit.WebView.setOverScrollMode(WebView.java:2379)
at android.view.View.<init>(View.java:4371)
at android.view.View.<init>(View.java:4519)
at android.view.ViewGroup.<init>(ViewGroup.java:582)
```

This problem seems to be happening in Lenovo, Oneplus, Samsung, Motorola, Xiaomi etc.(Non-rooted Android 5 and above devices).

It’s very hard to believe that such devices don’t have WebView installed.

Here’s what I discovered while digging the <b>reason</b> for this issue:

As we know that WebView has been shipping as a separate app from android 5.0,<b> at the time view is being inflated, the WebView package is being updated by the OS and therefore it can’t find the WebView package for those few moments</b>.

People on the GitHub bug tracker said chrome and android were not going to fix this because it’s normal behaviour and up to the developer to handle it.

So here’s a hacky solution that prevents crashes (You can catch the specific exception too):

```
try {
        // the inflating code that's causing the crash

    } catch (Exception e) {
        if (e.getMessage() != null && e.getMessage().contains("webview")) {
            // If the system failed to inflate this view because of the WebView (which could
            // be one of several types of exceptions), it likely means that the system WebView
            // is either not present (unlikely) OR in the process of being updated (also unlikely).
            // It's unlikely but we have been receiving a lot of crashes.
            // In this case, show the user a message and finish the activity
        }
  }
```

So next time when the user opens the app, probably they won’t see the crash again as WebView must be updated by then.

Happy Coding !!







