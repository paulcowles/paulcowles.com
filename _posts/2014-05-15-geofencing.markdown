---
layout: post
title:  "Unlocking things with iOS Geofencing"
date:   2014-05-15 00:20:10
categories: ios
tags: geofencing ios location proximity
permalink: ios-geofencing
---

Recently added to the Social Scavenger platform is the ability to unlock challenges as you reach a predefined location. On the iOS app, we implemented geofencing so that even when the app is not running, or in the background, a notification is given to the user that they've opened up something new by moving into a challenge zone. 

Naturally we wanted the ability to set the unlock zone as close as possible to the target, so we did a series of tests using a specified point extended by a radius to form a circular zone.  We tried a radius of 50m, 100m, 150m, 250m, 500m, 1000m - and tested approach under various conditions like walking, driving, and subway (underground).

The result of our testing was that any radius less than 250m produced alot of false negatives. We could be on top of the center point and in many cases would not be notified as the geofence did not trigger. 250m produced the best results (least false negatives and positives) with the smallest radius. Our subway tests were as expected the worst performers. Coming in and out of the underground often produced erratic results. 

I know when we embarked on this feature we searched for any posts that might hint at how the geofencing would function in small zones so if you're wondering if it will work for your use case, hopefully this is of help.



