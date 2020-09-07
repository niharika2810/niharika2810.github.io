
## Introduction
Now we already know [various tools](https://medium.com/1mgofficial/constraintlayout-vs-other-layouts-a-battle-towards-performance-part-2-21d6b5a6054c) that can be used for measuring the performance of layouts with different ViewGroups we have in Android.

Here, we are sharing the result for different layouts testing performed on Coolpad and Android 6.0 Marshmallow.

I also created a [Github project](https://github.com/niharika2810/LayoutPerformance-Analysis-App) for your reference. You can check out the code for various layouts there.

### Disclaimer:

These results may vary device to device and OEMs as well because optimizations vary from device to device and OS level too. You can check with your device.

### Here are the results -

1) View inside another View in the centre-
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/result_1.png">
</div>
2) RecyclerView with 100 Items
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/result_2.png">
</div>
3) Simple UI with TextViews and Images
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/result_3.png">
</div>
4) Complex UI with Images, Text, ScrollView, Buttons etc.
<div style="text-align:center">
<img align="center" width="300" height="300" src="/Images/Article/result_4.png">
</div>
Note*: Now there are lot enhancements with various viewgroups, so these results may vary for you.

>Please Note:- Always measure your layout performance through systrace and GPU overdraw before finalizing any layout. These results can vary from device to device and with respect to the OS version too. On average, however, the results will remain the same.

### Conclusion -

The complete analysis of the various layouts tells us -

1) With Complex UI where we can have more hierarchies, it is wise to use constraint layouts.

2) If you are using LinearLayout with weights/RelativeLayout with hierarchies, replace them with Constraint layout.

3) For simple layouts with just one view centred on the screen, go for Frame layouts.

4) For simple screens, where you have to place views just horizontally and vertically, go for LinearLayout.

### References -

1) <a href="https://medium.com/comparethemarket/three-key-lessons-when-migrating-to-constraintlayout-dff38c31a47">Three-key-lessons-when-migrating-to-constraintlayout</a>

2) <a href="https://github.com/googlesamples/android-constraint-layout-performance">Android-constraint-layout-performance</a>

3) <a href="https://stuff.mit.edu/afs/sipb/project/android/docs/tools/debugging/systrace.html">Systrace</a>

4) <a href="https://medium.com/rosberryapps/make-your-custom-view-60fps-in-android-4587bbffa557">Make-your-custom-view-60fps-in-android</a>

5) <a href="https://medium.com/mindorks/android-app-performance-optimization-cdccb422e38e">Android-app-performance-optimization</a>






