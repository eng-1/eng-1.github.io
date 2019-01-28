---
layout: post
title: Repacking Modified Android APK Files
categories: [Programming, Reverse, Engineering]
tags: [Android, APK, OBB, File, Repacking, Bash, ]
excerpt_separator: <!--more-->
---

 Having extensively modified your Android application's *APK* file, you now wish to install it on your phone and give it a run. Unfortunately, `adb install` refuses to cooperate, and instead returns `INSTALL_PARSE_FAILED_NO_CERTIFICATES`. To get around this, we will need to imitate an official publisher from the Google Play Store, and sign our new APK file with a useful little command called `jarsigner`.
 
<!--more-->
 
```bash
$ adb install reverseEngineering/apk/finished/evil-app.apk 
Failed to install reverseEngineering/apk/finished/evil-app.apk: Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES: Package /data/app/vmdl1428250932.tmp/base.apk has no certificates at entry modifiedFile.txt]
```

 As can be seen, the addition of any new files to the apk will immediately trigger an error to appear. Even the slightest change in the name of a file will set off *ADB*'s alarms. This prevents us from simply renaming a ".zip" file to ".apk" and installing it on our phone.
 
 The main reason why lies in the design of the Google Play Store system itself: in order to guarantee the integrity of the *APK* as it's being downloaded, Android requires all *APK* files to be signed using a special key file, the credentials of which end up in the `META-INF` folder of the *APK*.
 
 Not to worry! Luckily, there exists a quick and easy way that we can verify our apk for installation. The first step, is to generate the key file itself. 
 
 There are a multitude of ways of going about this, but the simplest and most straightforward, in my opinion, is to use *Android Studio*.
 
 Go into `Build > Generate Signed Bundle / APK... > APK > Next > Create New...` and fill out the form. Here is an example of a simple key that *Android Studio will generate, and place in the home folder upon completion:
 
 ![Key Generation Window](/assets/APK_Repacking_1.png)
 
 Now that the key has been generated, you can delete the existing `META-INF` folder inside the original files of the apk, and use your new *JKS*/*.Keystore* file to sign them and make it valid for install. I created this bash script to combine the process of taking all the files, zipping them, and signing them into one single command:

```bash
#!/bin/bash
set -xe
POS=$HOME/<Your_Working_Directory>/apk
rm -r -f $POS/source/META-INF
cd $POS/source
zip -r $POS/finished/evil-app.apk .
cd
jarsigner -verbose -keystore <Your_Key_Name>.jks -storepass <Your_Key_Password> $HOME/<Your_Working_Directory>/finished/evil-app.apk <Your_Key_Alias>
```



 You are now free to install your modified *APK*, and experiment with the changed you have made.