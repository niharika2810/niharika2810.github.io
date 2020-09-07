---
title: Android allowBackup demystified
---
<br/>
<br/>
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/allow_backup.jpg">
</div>
<br/>
<br/>

## Introduction

There are all sorts of "backups" that help get you out of various situations. Think of the spare tire in your trunk or the extra pieces of webbing in a climbing anchor. If you've ever been rock climbing, you know that it's essential to build redundancies into your anchors. That way, if one part fails, you've got another part as a backup.

Likewise, Android provides you with an impressive setting by the name of allowBackup which helps us automatically backing up application data.

### What Does Android allowBackup Do?

As the [documentation](https://developer.android.com/guide/topics/data/autobackup) says, "Auto Backup for Apps automatically backs up a user's data from apps that target and run on Android 6.0 (API level 23) or later."<br/>

We can leverage auto backup in our Android app so that our users can more quickly recover their data if they ever switch phones or reinstall our app after having uninstalled it.<br/>

If the user deletes the app in any way, including via a factory reset of their device, the data for the app will still be available when the user re-downloads the app and the .apk is installed. Auto Backup also works across devices, meaning that when your user gets a new phone, they won't lose key information from your app.<br/>


### Where Is the Backup Data Stored?

Android preserves app data by uploading it to the user's Google Drive (by default) where it's protected by the user's Google Account credentials.

Data is stored in a private folder in the user's Google Drive account. The saved data does not count towards the user's personal Google Drive quota. Only the most recent backup is stored. When a backup is made, the previous backup (if one exists) is deleted. The backup data can't be read by the user or other apps on the device.

Users can see a list of apps that have been backed up in the Google Drive Android app. On an Android-powered device, users can find this list in the Drive app's navigation drawer under Settings > Backup and reset > App data.


### How Much Data Can Be Backed Up?

Users can store up to 25MB of data, which persists across the lifetime of an app being installed on your device.

In practice, however, this is more than enough if you just need to save settings and preferences.


### What Is Backed Up?

By default, Auto Backup includes files in most of the directories that are assigned to your app by the system:

1) Shared preferences files

2) Files saved to your app's internal storage, accessed by [getFilesDir()](https://developer.android.com/reference/android/content/Context#getFilesDir()) or [getDir(String, int)](https://developer.android.com/reference/android/content/Context#getDir(java.lang.String,%20int))

3) Files in the directory returned by [getDatabasePath(String)](https://developer.android.com/reference/android/content/Context#getDatabasePath(java.lang.String)), which also includes files created with the SQLiteOpenHelper class

4) Files on external storage in the directory returned by [getExternalFilesDir(String)](https://developer.android.com/reference/android/content/Context#getExternalFilesDir(java.lang.String))

You can configure your app to include and exclude particular files.


### How Can We Customize What to Include and Exclude From the Backup?

For this we have [android:fullBackupContent](https://developer.android.com/guide/topics/manifest/application-element#fullBackupContent), which points to an XML file that contains [full backup rules](https://developer.android.com/guide/topics/data/autobackup) for Auto Backup. These rules determine what files get backed up.

You can do this in just a couple of steps:

1) Create an XML file auto_backup_rules.xml in the res/xml directory.

2) Specify what you want to include or exclude from the backup using the syntax below:

  a)If a type is specified in both the include and the exclude tags, the item specified in the exclude tag takes precedence.

  b)You can add a new <include> or <exclude> tag for every new resource you want to specify.

  c)The path tag specifies the path to a resource you want to include or exclude. For example: <exclude domain="database" path="my_db.db"/>

3) Include your backup file in your Android Manifest:
```
    <application ... android:fullBackupContent="@xml/auto_backup_rules">
```

Here is the sample example for the XML file:
```
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <include domain="sharedpref" path="."/>
    <exclude domain="sharedpref" path="device.xml"/>
</full-backup-content>
```

The above sample backups all shared preferences except device.xml.

### When Can the Backups Be Triggered?

Backups occur automatically when all of the following conditions are met:

1) The user has enabled backup on the device. In Android 9, this setting is in Settings > System > Backup.

2) At least 24 hours have elapsed since the last backup.

3) The device is idle.

4) The device is connected to a Wi-Fi network (if the device user hasn't opted-in to mobile-data backups).

### Do I Need to Worry About the Backup?

Android backups rely on the Android Debug Bridge (ADB) command to perform backup and restore. ADB, however, has been a soft target for hackers and is still not trusted by respected developers. The idea that someone can inject malicious code into your backup data is unsettling, to say the least. This generally isn't a problem for end users as it requires debugging to be enabled on the device, but since a lot of Android users are fond of exploring and rooting their devices, it can become a serious problem.

Once backed up, all application data can be read by the user. adb restore allows the creation of application data from a source specified by the user. Following a restore, applications should not assume that the data, file permissions, and directory permissions were created by the application itself.

Therefore, applications that handle and store sensitive information such as card details, passwords etc. should have this setting explicitly set to false - by default, it is set to true - to prevent such risks, or you can also customize what needs to be backed up.

```
<application ...        android:allowBackup="false">
```