+++
date = "2017-02-24T21:47:40+01:00"
title = "life without gapps"
draft = false
categories = ["Android"]

+++

# Life without Gapps

## Guesture typing
In order to enjoy guesture typing with the AOSP keyboard you need to copy two (proprietary) libs into your `system/lib64`. This is if you're actually on a arm64 enabled device of course, otherwise you copy it to `system/lib`

The following require libs can be obtained from the OpenGapps Github repository.

[libjni_keyboarddecoder](https://github.com/opengapps/arm64/raw/master/lib64/23/libjni_keyboarddecoder.so)  
[libjni_latinimegoogle](https://github.com/opengapps/arm64/raw/master/lib64/23/libjni_latinimegoogle.so)

After you've copied these to the appropriate folder be sure to set the correct permissions. `-rw-r--r--` or for the numeric types amongst you `644`.

Reboot and you're all set!

Mind you, this is not for the purists, these are proprietary binaries and can therefore never be completely trusted.

---

## Location services
As far as location services go you're left with

- UnifiedNlp (no GAPPS)
- microG Services Framework proxy
- Fakestore


Move `/data/app/gms` to `/system/priv-app`
Set permissions of the main dir to `drwxr-xr-x` `755`
