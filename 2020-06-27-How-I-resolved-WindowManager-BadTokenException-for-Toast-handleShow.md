<div style="text-align:center">
<h1> How I resolved WindowManager BadTokenException for Toast#handleShow()?
</h1>
Blog by <a href="http://thedroidlady.com/">theDroidLady</a>
</div>
<br/>
<br/>
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/toast.png">
</div>
<br/>
<br/>
## Introduction

“Every problem is a gift — without problems, we would not grow.” ― Anthony Robbins

When we have to display information for a short span of time in Android, we use [Toast](https://developer.android.com/guide/topics/ui/notifiers/toasts).
Here are some key features:

1) It provides simple feedback about an operation in a small popup.

2) It is a view containing a quick little message for the user.

3)It only fills the amount of space required for the message and the current activity remains visible and interactive.

Recently, I started getting a major number of crashes for <b>Toast#handleShow()</b> on Crashlytics.

Here are the logs:

```
Fatal Exception: android.view.WindowManager$BadTokenException
Unable to add window -- token android.os.BinderProxy@f839de9 is not valid; is your activity running?
android.view.ViewRootImpl.setView (ViewRootImpl.java:697)
android.view.WindowManagerGlobal.addView (WindowManagerGlobal.java:347)
android.view.WindowManagerImpl.addView (WindowManagerImpl.java:94)
android.widget.Toast$TN.handleShow (Toast.java:463)
android.widget.Toast$TN$2.handleMessage (Toast.java:346)
android.os.Handler.dispatchMessage (Handler.java:102)
android.os.Looper.loop (Looper.java:163)
android.app.ActivityThread.main (ActivityThread.java:6377)
java.lang.reflect.Method.invoke (Method.java)
com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run (ZygoteInit.java:904)
```


Before the analysis, it's important for you to know about <b>WindowManager.BadTokenException</b>.

>Exception that is thrown when trying to add view whose LayoutParams [LayoutParams#token](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#token) is invalid.

Lets, Look at the WindowManagerService.java file.
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/window_token.png">
</div>
WindowManagerService (WMS) is a system service that manages the windows on Android.

### Window Tokens:

>As the name suggest, a window token is a special type of Binder token used by window manager to uniquely identify a window in the system. Window tokens are important for security reasons because they make it impossible for malicious applications to draw on top of the windows of other applications. The window manager protects against this by requiring applications to pass their application’s window token as part of each request to add or remove a window. If the tokens don’t match, the window manager rejects the request and throws a BadTokenException. Without window tokens, this necessary identification step wouldn’t be possible and the window manager wouldn’t be able to protect itself from malicious applications.

### A real-world scenario:

>When an application starts up for the first time, the <b>ActivityManagerService</b> creates a special kind of window token called an application window token, which uniquely identifies the application’s top-level container window. The activity manager gives this token to both the (*application) and the window manager, and it sends the token to the window manager each time it wants to add a new window to the screen. This ensures secure interaction between the application and the window manager (by making it impossible to add windows on top of other applications), and also makes it easy for the activity manager to make direct requests to the window manager.

And the code that throws “BadTokenException” :

<b>Return WindowManagerGlobal.ADD_BAD_APP_TOKEN</b>

If returned the <b>WindowManagerGlobal.ADD_BAD_APP_TOKEN</b>, the exception occurs when the WMS.addWindow () function is called to check whether the window to be added is not in violation of the policy according to the android window policy.

So now, after all, my research and analysis on BadTokenException, what I noticed about all the crashes there on Crashltyics:

All were on Android [7.1](https://en.wikipedia.org/wiki/Android_Nougat) (API level 25).

I compared the API level 25 Toast class and others for the difference and I found an interesting thing there:

From API 25, Android added a new param <b>IBinder windowToken</b> for Toast#handleShow(), and It brought an exception. 😪

Let me show you the code:
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/api_difference.png">
</div>
As you can see here, they try-catch the <b>mWM.addView(mView, mParams)</b>

>Since the notification manager service cancels the token right after it notifies us to cancel the toast there is an inherent race and we may attempt to add a window after the token has been invalidated. Let us hedge against that.

However, <b>API level 25</b> is still at risk. We had to capture it on our own else it would have continued to produce an exception for the users.

This exception occurs regardless of whether the <b>Context</b> you passed to Toast is an Activity or ApplicationContext or Service. And you can not try-catch it.

### Solution :

So I created a library that possesses a <b>ToastHandler</b> class which is responsible for showing Toast smoothly on all API versions and replaced the base Context to a ToastContextWrapper, it will hook the <b>WindowManagerWrapper.addView(view, params)</b> method and fix the exception.

You just need to add this in your app/build.gradle:

```
implementation 'com.toastfix:toastcompatwrapper:{latest_version}'
```

Now, you are all good to use your Toast :

```
ToastHandler.INSTANCE.showToast(this, "Hi,I am Toast", Toast.LENGTH_LONG)
```

You can find the [source code](https://github.com/niharika2810/ToastHandler) here.

Happy Coding !!







