# Scoped Storage

<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/scoped.jpeg">
</div>


## Introduction

Storage preventing apps from having unrestricted access to the filesystem and helping in reducing file clutter.

If I talk about the recent improvements in Android at the OS level, it is all about protecting the app & user data and providing access in a more organised form.

The reason for a change is good, but it means more work for developers.

With Android 11, Some major changes and restrictions are added to enhance user privacy, including the following:

* [Scoped storage enforcement](https://developer.android.com/preview/privacy/storage#scoped-storage): Access into external storage directories is limited to an app-specific directory and specific types of media that the app has created.

* [Permissions auto-reset](https://developer.android.com/preview/privacy/permissions#auto-reset): If users haven't interacted with an app for a few months, the system auto-resets the app's sensitive permissions.

* [Background location access](https://developer.android.com/preview/privacy/location#background-location): Users must be directed to system settings in order to grant the background location permission to apps.

* [Package visibility](https://developer.android.com/preview/privacy/package-visibility): When an app queries for the list of installed apps on the device, the returned list is filtered.

---

Recently, I deep dive into the concept of Scoped Storage to understand the intention and prepare my Android app for the change accordingly.

## Storage Structure (Before Android 10)

Before going into the implementation part, Let's first understand the way data was organised before Android 10:
<div style="text-align:center">
<img align="center" width="500" height="500" src="/Images/Article/diagram_one.png">
</div>
---

* Private Storage: All apps have their own private directory in internal storage, i.e Android/data/{package name}, not visible to other apps.

* Shared Storage: Apart from private storage, the rest of the storage was called shared storage. This includes all the media and non-media files being stored and an app with storage permission can easily access them.

---

### Any Problem with the Current Structure & Access? 

Do you see any problem with this structure or anything you ever wanted Google to change? 

Let me elaborate on some problems:

* If I am an e-commerce app, which needs storage access only to ask the user to upload their profile picture, which means access to a certain file or file system. You can take another example as the chat application where you want to upload media or write a media file into the system. which means access to a single media file. So why am I asking for the whole storage access?

* Have you ever got worried about your sensitive data like your doctor prescriptions or bank documents being getting accessed by all the apps installed on your device? 

* If I am uninstalling an application and needed the data/files related to it till the time I was using the application, Do we have any easy way to clear all the clutter?

I am sure this must have made you think about your app & data security, privacy and organisation. 


>Don't worry, Google's recent Android update related to storage comes to the rescue.

---

## Scoped Storage

### Idea

 Google gives two valid reasons why it's making this change: Security and to reduce leftover "app clutter."

[Application Sandboxing](https://source.android.com/security/app-sandbox) is a core part of Android's design, isolating apps from each other. In Android Q, taking the same fundamental principle from Application Sandboxing, Google has introduced Scoped Storage.

>These changes were originally slated to apply to every app on a phone running Android 10 or later, but because of developer backlash Google changed course and only required the use of Scoped Storage for apps that target Android API level 29, which is Android 10. But with Android 11 Scoped Storage is back, and Google isn't likely to change its mind this time.

* <b>Better Attribution</b>: An app would be provided access to the storage blocks which has relevant data for the app.

Look at the <b>Storage structure</b> on Android 10 & above:
<div style="text-align:center">
<img align="center" width="500" height="500" src="/Images/Article/diagram_two.png">
</div>
>Now, **Private Storage** is the same as before but Shared Storage is further divided into Media & Download(Non-Media Files) Collection.

* **Reducing File Clutter**: The system will bind storage to owner apps so that it becomes easier for the system to locate relevant files, corresponding to an app. It is useful when the App is uninstalled, so all the data related to the app is also uninstalled.

* **Preventing the abuse of READ_EXTERNAL_STORAGE permission**: Granting this permission for an app today, gives access to the entire external storage where we save things like photos, private documents, videos, and other potentially sensitive files. With Scoped Storage enforced, apps only can see their own data folders plus certain media types like music files using other storage APIs.

Wow, Now this appears something cool, Isn't it?

Being a user, I have more control over my files and to whom I am giving access. Also, It will bring more extra space available for my device if non-required files are getting deleted with the app itself.

---

### Is there any change with permissions to access the files now?

Yes, we do have.
<div style="text-align:center">
<img align="center" width="600" height="500" src="/Images/Article/diagram_latest.png">
</div>
* **Unrestricted access to its individual app storage**: App will have unlimited access to their internal and external storage for both reads & write operation. So, with Android 10, you don't need to provide storage permission to write files to your own app directory on the SD card.

* **Unrestricted access to media files and download collections (Contributed)**: You have unrestricted access to contribute files to the media collections and downloads of your own app. So, no need to take permission if you want to save any image, video, or any other media file in the media collection. (as long as the file is stored in the organised collection.)

* **Media collection contributed by other apps** can be accessed using **'READ_STORAGE_PERMISSION'** permission. **'WRITE_STORAGE_PERMISSION'** permission will be deprecated from the next version and if used, will work the same as **'READ_STORAGE_PERMISSION'**. 

* Non-media files contributed by other apps will need **Storage Access Framework APIs** for reading as well as write operations.

>NOTE: If your app uses the legacy storage model and previously targeted Android 10 or lower, you might be storing data in a directory that your app cannot access when the scoped storage model is enabled. Before you target Android 11, migrate data to a directory that's compatible with scoped storage. In most cases, you can migrate data to your app-specific directory.
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/upgrade.png">
</div>

 Woohoo!! More specific permissions. I am happy and feeling more secured now. :-)


---

**Is there any way to opt-out of this storage structure for apps targetting Android 10 and above?**

>To give developers additional time for testing, apps that target Android 10 (API level 29) can still request the requestLegacyExternalStorage attribute. This flag allows apps to temporarily opt out of the changes associated with scoped storage, such as granting access to different directories and different types of media files.

---

Any app that is targeted for **Android 11** or later must use the new storage APIs, and that includes Scoped Storage. Changes to Google Play's developer agreement say that starting **August 1, 2020**, all new apps submitted to Google Play must target Android 10 or later, and all updates to existing apps must target Android 10 or later as of **November 1, 2020**. Expect this same behaviour and next year apps will likely be required to target Android 11.

This is the time. Learn, understand and implement it :-)

---

## Storage Operations

Let me brief out some frequently done storage operations and how to do them:

* **Select a file**: Use [ACTION_OPEN_DOCUMENT](https://developer.android.com/reference/android/content/Intent.html#ACTION_OPEN_DOCUMENT) which then opens the system's file picker app to allow the user to choose the file to open. To show only the types of files that your app supports, specify a MIME type.

```
// Request code for selecting a PDF document.
const val PICK_PDF_FILE = 2

fun openFile(pickerInitialUri: uri) {
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
        addCategory(Intent.CATEGORY_OPENABLE)
        type = "application/pdf"

        // Optionally, specify a URI for the file that should appear in the
        // system file picker when it loads.
        putExtra(DocumentsContract.EXTRA_INITIAL_URI, pickerInitialUri)
    }

    startActivityForResult(intent, PICK_PDF_FILE)
```

* **Select a folder**: Intent ACTION_OPEN_DOCUMENT can be replaced with [ACTION_OPEN_DOCUMENT_TREE](https://developer.android.com/reference/android/content/Intent#ACTION_OPEN_DOCUMENT_TREE).

```
fun openDirectory(pickerInitialUri: Uri) {
    // Choose a directory using the system's file picker.
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT_TREE).apply {
        // Provide read access to files and sub-directories in the user-selected
        // directory.
        flags = Intent.FLAG_GRANT_READ_URI_PERMISSION

        // Optionally, specify a URI for the directory that should be opened in
        // the system file picker when it loads.
        putExtra(DocumentsContract.EXTRA_INITIAL_URI, pickerInitialUri)
    }

    startActivityForResult(intent, your-request-code)
}
```

> Note: This access will be valid till user reboots device. If app wants to persist the access, while accessing URI using content resolver , content resolver has to call [takePersistableUriPermission](https://developer.android.com/reference/android/content/ContentResolver#takePersistableUriPermission(android.net.Uri,%20int)) method.

Also, If you iterate through a large number of files within the directory that's accessed using ACTION_OPEN_DOCUMENT_TREE, your app's performance might be reduced.

* **Create a file**: Use [ACTION_CREATE_DOCUMENT](https://developer.android.com/reference/android/content/Intent#ACTION_CREATE_DOCUMENT) to save a file in a specific location.

>Note: ACTION_CREATE_DOCUMENT cannot overwrite an existing file. If your app tries to save a file with the same name, the system appends a number in parentheses at the end of the file name.

```
// Request code for creating a PDF document.
const val CREATE_FILE = 1

private fun createFile(pickerInitialUri: Uri) {
    val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
        addCategory(Intent.CATEGORY_OPENABLE)
        type = "application/pdf"
        putExtra(Intent.EXTRA_TITLE, "invoice.pdf")

        // Optionally, specify a URI for the directory that should be opened in
        // the system file picker before your app creates the document.
        putExtra(DocumentsContract.EXTRA_INITIAL_URI, pickerInitialUri)
    }
    startActivityForResult(intent, CREATE_FILE)
}
```

Wait, I am not able to get the metadata i.e location of my image, is this changed ? How can I can get it?

## Media location permission

If your app targets Android 10 (API level 29) or higher, in order for your app to retrieve unredacted Exif metadata from photos, you need to declare the [ACCESS_MEDIA_LOCATION](https://developer.android.com/reference/android/Manifest.permission#ACCESS_MEDIA_LOCATION) permission in your app's manifest, then request this permission at runtime.

```
<uses-permission android:name="android.permission.ACCESS_MEDIA_LOCATION"/>
```

>Caution: Because you request the ***ACCESS_MEDIA_LOCATION*** permission at runtime, there is no guarantee that your app has access to unredacted Exif metadata from photos. Your app requires explicit user consent in order to gain access to this information.

### What if I am a file manager kind of back up app that needs to access everything?

We have a way out for this too. You need to follow these simple steps:

* An app can request special app access called All files access from the user by doing the following:

  * Declare the [MANAGE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission#MANAGE_EXTERNAL_STORAGE) permission in the manifest.
  * Use the [ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION](https://developer.android.com/reference/android/provider/Settings#ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION) intent action to direct users to a system settings page where they can enable the following option for your app: Allow access to manage all files.

## Legitmitate apps need these special permissions.

* Once the user grants the permission to have broad access, then the user will get an unfiltered view of MediaStore that include non-media file.
* Only apps granted by Google will have complete access to storage. For that, submit the declaration form to Google Play. Now, these are only [Whitelisted](https://support.google.com/a/answer/7281227?visit_id=637338122728852013-3109567423&rd=1) apps by Google.

---

## What if my app uses custom file picker which displays exact data directories?

There is nothing you can do about it. You might want to use system picker from now on.

---

---

## Is there anything which I should remember while implementing it?

* Don't use a static path. Lockdown the file path access.

* Use [MediaStore](https://developer.android.com/training/data-storage/shared/media) (recommended).

* MediaStore should be used properly, for example, don't put your music files in the picture directory.

* Non-media files should be in the download directory (recommended).

* To access non-media files by other apps, use [System Picker](https://developer.android.com/guide/topics/providers/document-provider) with SAF( Storage Access Framework). Runtime permission will be requested for complete access to that app.

---


You can check the [Github](https://github.com/niharika2810/ScopedStorageDemo) sample for the implementation.

References
 
* [Link 1](https://www.androidcentral.com/what-scoped-storage)

* [Link 2](https://developer.android.com/preview/privacy/storage)