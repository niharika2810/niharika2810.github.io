---
layout: post
title: Understanding the core of Aarogya Setu-Bluetooth
---
![AarogyaSetuApp](/Images/Article/aarogya_setu.jpeg)

## Introduction

>[Aarogya Setu](https://play.google.com/store/apps/details?id=nic.goi.aarogyasetu&hl=en_IN) is a mobile application developed by the Government of India to connect essential health services with the people of India in our combined fight against COVID-19.Do visit this [link](https://aarogyasetu.gov.in/) for more details.

<b>AarogyaSetu</b> started with an idea of automatic contact tracing. Proximity logging within an app addresses a key limitation of manual contact tracing: it is dependent on a person’s memory and is therefore limited to the contacts that a person is acquainted with and remembers having met.

For contact tracing, we had two options to go with: <b>Bluetooth or GPS</b>.

### Bluetooth vs GPS:

<b>Bluetooth<.b> was chosen because it is able to classify close contacts with a significantly <b>lower false-positive</b> rate than GPS.

Given that GPS accuracy decreases in indoor environments, entire shopping malls or skyscrapers would be within the margin of error of a single GPS point.

![Bluetooth](/Images/Article/bluetooth.jpeg)

So, with an idea that is more scalable, accurate and less resource-intensive, we created an app which:

1) Can help us in tracing the nearby contacts through Bluetooth.

2) Consumes low battery power.

3) Performs continuous.

4) Is compatible with both Android and iOS.

### Implementation

We have used [Bluetooth Low Energy(BLE)](https://developer.android.com/guide/topics/connectivity/bluetooth-le) technology that has built-in platform support in the central role and provides APIs that apps can use to discover devices, query for services, and transmit information since Android 4.3(API level 18).

In contrast to Classic Bluetooth, BLE is designed to provide significantly lower power consumption.

### Permissions required:

"android.permission.BLUETOOTH"

"android.permission.BLUETOOTH_ADMIN"

"android.permission.ACCESS_COARSE_LOCATION"

>Note: Some of the devices (above 6.0) requires location permission to use Bluetooth. You can read more [here](https://www.polidea.com/blog/a-curious-relationship-android-ble-and-location/).

BLE works with advertisement and scanning. The device in the central role scans, looking for advertisement, and the device in the peripheral role makes the advertisement.

### Advertising Over BLE

We have [BluetoothLeAdvertiser](https://developer.android.com/reference/android/bluetooth/le/BluetoothLeAdvertiser) class for starting and stopping advertising.

An advertiser(Added in API level 21) can broadcast up to 31 bytes of advertisement data represented by [AdvertiseData](https://developer.android.com/reference/android/bluetooth/le/AdvertiseData).

### Advertising Settings :

The [AdvertiseSettings](https://developer.android.com/reference/android/bluetooth/le/AdvertiseSettings) provide a way to adjust advertising preferences for each Bluetooth LE advertisement instance.

1) setTxPowerLevel: We have set the lowest transmission (TX) power level to restrict the visibility range of advertising packets.

2) setAdvertiseMode: We are setting the advertising power and latency based on the number of unique scans/interval and changing it accordingly to provide lower power consumption.

It can only be one of [AdvertiseSettings#ADVERTISE_MODE_LOW_POWER](https://developer.android.com/reference/android/bluetooth/le/AdvertiseSettings#ADVERTISE_MODE_LOW_POWER), [AdvertiseSettings#ADVERTISE_MODE_BALANCED](https://developer.android.com/reference/android/bluetooth/le/AdvertiseSettings#ADVERTISE_MODE_BALANCED), or [AdvertiseSettings#ADVERTISE_MODE_LOW_LATENCY](https://developer.android.com/reference/android/bluetooth/le/AdvertiseSettings#ADVERTISE_MODE_LOW_LATENCY).

3) setConnectable: We have used it for making our advertisement connectable(GATT connection for iOS, will explain it later).This flag takes 1 byte+2 bytes header.

### Advertising Data:

This is the data to be advertised as well as the scan response data for active scans.

1) <b>setIncludeDeviceName</b>: We are setting an 8 bytes(+2 bytes header) unique id as Bluetooth name for distinguishing a packet during the scan. So, we have made this flag true.

2) <b>addServiceUuid</b>: Setting a 16 bytes(+2 bytes header) UUID that is used for identifying our packets when they are advertised.

So now, we know how all 31 bytes are getting consumed. For more clarification on byte structure, please check this [class](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/bluetooth/le/BluetoothLeAdvertiser.java).

### GATT server:

We need to connect to the GATT server on the device to make devices talk to each other.

After opening a [GATT](https://developer.android.com/guide/topics/connectivity/bluetooth-le#connect) server for connection, we have added one service with 2 characteristics (UniqueID & Pinger characteristics).

### Reason for implementing:

As per the iOS limitation, iOS is not able to scan if it goes in background.

So, we found that making advertisement connectable through GATT can actually minimize this limitation by pinging the devices for connection and getting the unique id from there so that the app remains active and the connection breaks only if the device goes too far.

Furthermore, you don't need to worry about multiple connections at the same time.GATT itself handles it for a good number of connections.

### Scanning through BLE

We have [BluetoothLeScanner(Added in API level 21)](https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner) to perform scan related operations for Bluetooth LE devices. An application can scan for a particular type of Bluetooth LE devices using [ScanFilter](https://developer.android.com/reference/android/bluetooth/le/ScanFilter).

### Scan Filter:
A ScanFilter allows clients to restrict scan results to only those that are of interest to them.

1) <b>setServiceUUID</b>: We are filtering using the Service UUID which we sent in our advertisement packet.

### ScanSettings:

Bluetooth LE scan settings are passed to [BluetoothLeScanner#startScan](https://developer.android.com/reference/android/bluetooth/le/BluetoothLeScanner#startScan(android.bluetooth.le.ScanCallback)) to define the parameters for the scan.

1) <b>setScanMode</b>: We are setting the scan mode based on the number of unique scans/interval and changing it accordingly to provide lower power consumption.

The scan mode can be one of [ScanSettings#SCAN_MODE_LOW_POWER](https://developer.android.com/reference/android/bluetooth/le/ScanSettings#SCAN_MODE_LOW_POWER), [ScanSettings#SCAN_MODE_BALANCED](https://developer.android.com/reference/android/bluetooth/le/ScanSettings#SCAN_MODE_BALANCED) or [ScanSettings#SCAN_MODE_LOW_LATENCY](https://developer.android.com/reference/android/bluetooth/le/ScanSettings#SCAN_MODE_LOW_LATENCY).

2) <b>setMatchMode</b>(Added in API level 23): We have set it MATCH_MODE_AGGRESSIVE (h/w will determine a match sooner even with feeble signal strength and few numbers of sightings/match in a duration).

3) <b>setPhy</b>(Added in API level 26): Setting the Physical Layer to use during this scan. This is used only if [ScanSettings.Builder#setLegacy](https://developer.android.com/reference/android/bluetooth/le/ScanSettings.Builder#setLegacy(boolean)) is set to false. To improve scan performance, we have used PHY_LE_1M channel for scanning. Read more about it [here](https://developer.android.com/reference/android/bluetooth/le/ScanSettings.Builder#setPhy(int)).

### ScanCallback:

Scan results are reported using these callbacks.

In onScanResult,

We get out scanResult from which we parse the device name, RSSI and txPower(Added in API 26).

1) Change the scan and advertisement mode for the device accordingly.
2) Store the scanResult in our local storage i.e database.

Also, we are checking the duplicacy of a scan record within a few seconds before we add it to our local storage.

>Android 7.0 introduced a BLE scan timeout, where any scan running for 30 minutes or more is effectively stopped automatically and can only resume “opportunistically”. So we are starting the scan again after an interval.

### Adaptive Scanning

We have implemented adaptive scanning through which we are changing the scan and advertisement mode of the scanning device based on the number of unique scans/interval.

As scan and advertisement modes are directly related to power consumption, this has helped us in adjusting the consumption based on the number of close contacts.

So, you can now worry even less for your battery power usage. Cheers !!

### App to scan continuously

To make our app scan continuously, we are doing all this in a [Service](https://developer.android.com/guide/components/services) and to cater to the background service restrictions in Android API 26 or higher, we are using <b>startForegroundService</b> which is similar to [startService](https://developer.android.com/reference/android/content/Context#startService)(android.content.Intent)(android.content.Intent), but with an implicit promise that the Service will call [startForeground](https://developer.android.com/reference/android/app/Service#startForeground(int,%20android.app.Notification))(int, android.app.Notification) to make the service run in the foreground.

We use this method if killing your service would be disruptive to the user. Read more about the foreground mechanism [here](https://developer.android.com/reference/android/app/Service#startForeground(int,%20android.app.Notification)).

### Bluetooth Vulnerabilities

There are vulnerabilities in Bluetooth technology that have to be patched at the operating system-level, and we, therefore, urge users to ensure that their operating systems are regularly patched.

We have tried to make the most out of the Bluetooth and we are constantly improving.

Check out my [talk](https://youtu.be/K4SLf3y6zmE) on Understanding the BLE in integration in AarogyaSetu:

[![Talk Link]](https://youtu.be/K4SLf3y6zmE)


This is the link for [slides](https://speakerdeck.com/niharika28/understanding-the-core-of-aarogyasetu-app-bluetooth-low-energy-ble).

Also, You can check the [source code](https://github.com/nic-delhi/AarogyaSetu_Android).

Thank You!!

I am fortunate enough to be a part of the magnificently electrifying [@SetuAarogya](https://twitter.com/SetuAarogya) team. Thanks to the <b>Government Of India</b> for giving me this opportunity to work on a project that has an impact on human lives. Stay Safe!!

### मैं सुरक्षित, हम सुरक्षित, भारत सुरक्षित!
