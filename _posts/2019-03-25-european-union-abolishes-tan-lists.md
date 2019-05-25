---
layout: post
title: European Union Abolishes TAN Lists and Banking Apps on Rooted Phones Ridiculousness
tags: Smartphone Sysadmin
---

When I logged into my online banking today I was greeted by an unexpected page. TAN lists on paper no longer work and I was asked to choose between the TAN app of my bank and the so called ChipTAN method popular in Germany. I remembered somebody told me that many banking apps refuse to work on rooted phones, so I mentally prepared for the worst. This is a summary of how I run my banks TAN app on my rooted Android using the open-source rooting tool [Magisk](https://github.com/topjohnwu/Magisk).

## European PSD2 directive

The change is caused by the Payment Services Directive 2 adopted by the European Parliament in 2015. The new law enforces two-factor authentication for banking transactions and therefore no longer allows TAN lists. The final deadline for financial institutes to implement the new TAN procedures is the 14th of September 2019. Although it seems like some banks like the German DKB already made the switch in February.

The benefit of the new methods is that they prevent man-in-the-browser attacks. In the classic scenario when the user does a transaction, malware on the users computer can change the amount and recipient while keeping the TAN entered by the user. With the new procedure the TANs are specific to the transaction details.

## The ChipTAN generator

This ChipTAN method is only used by German and Austrian banks but there are similar devices for example in the UK. When asked to authenticate a transaction the transaction details like the amount and recipient are encoded in a flickering barcode and displayed on the screen. You insert your bank card into your ChipTAN device and scan the barcode with the device. On older devices which don't have scanning functionality you have to enter the transaction details manually. A chip on the bank card will then generate a TAN from a secret identity on the card and the transaction details, which is then displayed on the ChipTAN device.

A device like this costs about 15 € but I have heard of people who got them from their bank.

## Banking apps on rooted phones

Because the ChipTAN method is quite a lot of effort I was favouring the app. The TAN2go app of my bank establishes a connection to my bank account using a secret key once, which is protected by a local password in the future. So to get a TAN for a new transaction I just have to enter a password on my phone.

Many banking and TAN apps including my banks app check if they are running on a rooted phone and are refusing to work in that case. I'm not sure if they store sensible data on flash storage, I suspect it should be protected by my password, or if they are just worried about other apps reading their memory. Overall this choice seems really unreasonable to me, they should at least give you the option with an extra warning.

Obviously you should only grant apps you trust root permissions and I only do that with open-source software. I need root for [Syncthing](https://syncthing.net/) to be able to write to my SD card for example. Recent Android versions allow non-root users only to write to the SD-card over some insane API, but that's another story.

## Using Magisk to trick the app

Android phones are normally rooted using SuperSU. Magisk is an alternative to SuperSU with the extra functionality to hide the fact that the phone is rooted to selected apps.

The first step of switching to Magisk is to unroot your SuperSU rooted phone. I use [LineageOS](https://www.lineageos.org/) and they provide a ZIP file you flash from recovery mode to root and unroot your phone. Download the su removal file to your phone from [here](https://download.lineageos.org/extras). Make sure you choose the right architecture and LineageOS version (you can find it in *About phone*). Now turn off your phone and boot it into recovery mode, on Samsung phones this works by holding the *Volume up*, *Home* and *Power* buttons. Depending on your recovery image tap *Install*, select the su removal ZIP and flash it. After rebooting your phone should be unrooted.

For a detailed guide on how to install Magisk check [the official website](https://magiskmanager.com/). Basically you need to download the Magisk Manager APK from their [GitHub](https://github.com/topjohnwu/Magisk/releases) and install it. You can also find instructions on how to compile the APK from source there. Then you can install the Magisk core to your phone from the Magisk Manager app, it will ask you for the installation method. I chose the *Direct Install* method but there are other installation methods which are documented [here](https://topjohnwu.github.io/Magisk/install.html). For example you could manually flash a ZIP from recovery mode similar to the unrooting step. After restarting the phone any apps which try to use root should ask for permission again. Root permissions can be managed using the Magisk Manager app.

Ensure the *Magisk Hide* option is enabled in the Magisk Manager settings, then select your banking app in the *Magisk Hide* tab. Depending on how smart your banking app is you might need to reinstall it after it complained about your rooted phone. For some people it also helped to use the *Magisk Core Only Mode* in the settings. To make sure uninstalling SuperSU worked you can check the SafetyNet-Status on the Magisk Manager main screen. This downloads some proprietary software from Google that is used to check if the phone is rooted and it shouldn't detect it.

## Conclusion

If you rely on a rooted phone you might be able to trick your banking app using Magisk. I also like that the app gives me control over the apps that have root permissions and there is a way to turn off the pop-ups when an app uses root which I found annoying. Unfornately this doesn't work for every banking app but as a last resort you can always get an extra device like the ChipTAN generator.

Comments on [Hacker News](https://news.ycombinator.com/item?id=19483952).
