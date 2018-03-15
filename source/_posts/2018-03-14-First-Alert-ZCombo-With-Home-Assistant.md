---
title: Connecting the First Alert Zcombo Smoke Alarm to Home Assistant
date: 2018-03-14 22:01:10
tags: Home Assistant, Home Automation
---

After some trial and error, I was able to connect my First Alert Zcombo Alarms to Home Assistant via a USB Aeotec Z-Stick.

## Ensuring the device is excluded

If the device was previously added to the network, or if you just want to be sure you are starting from scratch, start by excluding the device.

* Slide the battery tray out on the alarm.
* In the Home Assistant Z-Wave console, press the `Remove Node` button.
* While holding the the `Test` button on the alarm, slide in the battery tray.  Continue to hold the `Test` button for a few seconds, releasing after the first beep
* Wait 10-20 seconds for the Alarm to beep a second time.  This indicates the exclusion worked.

## Joining the network

The procedure to add the device back to the network is very similar.

* Slide the battery tray out on the alarm.
* In the Home Assistant Z-Wave console, press the `Add Node` button.
* While holding the the `Test` button on the alarm, slide in the battery tray.  Continue to hold the `Test` button for a few seconds, releasing after the first beep
* Wait 10-20 seconds for the Alarm to beep a second time.  This indicates the network join has succeeded. If you hear three bursts of three tones, this is an error and you should exclude the device and try again.

If you are adding multiple devices, I recommend setting a descriptive name for the device just added in the [entity registry](https://home-assistant.io/docs/configuration/entity-registry/) before adding more devices.  Without setting a custom name right away, you will end up with multiple devices with the same name which will be difficult to sort out later.

## Healing the network

The Zcombo devices are somewhat unique in the sense that they do not seem to support waking up on an interval to report their status.  Most battery operated Z-Wave devices can be configured to wake on interval but I have yet to find a way to make it work for Zcombo.  This lack of wake support seems to be corroborated by [other people](https://groups.google.com/d/msg/openhab/TvCiIM-TeYs/3smDUSrmTj8J) on the internet.

However, if you want to heal the network or manually force the device to wake up and report its status, this is possible.  Simply repeat the battery slide/Test button procedure as before.

* Slide the battery tray out on the alarm.
* If you want to heal the network, you can press the `Heal Network` button in the Home Assistant Z-Wave console.  If you are not concerned with healing and just want the device to report it's status, you do not need to press any Home Assistant buttons.
* While holding the the `Test` button on the alarm, slide in the battery tray.  Continue to hold the `Test` button for a few seconds, releasing after the first beep
* Wait 10-20 seconds for the Alarm to beep a second time.  This indicates the device has woken up.

If all has gone well, the devices status in Home Assistant will change to `Initializing (Session)`.  This will persist for 15-30 seconds, then the device will switch to the `Sleeping` state. 
