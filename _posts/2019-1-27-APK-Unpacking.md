---
layout: post
title: Unpacking Android APK Files
categories: [Programming, Reverse, Engineering]
tags: [Android, APK, OBB, File, Unpacking, Bash]
---

  All Android apps come in one basic format: the *APK*, or *Android Package Kit*. Similar to an *EXE* file on windows, when you press install on the Google Store, the *APK* is downloaded onto your phone, and installed. It follows that, by downloading it instead onto your computer, you could theoretically open it, and be able to look around and see all the files that the creator made.

 To start with, we will need to download the actual *APK* file to our desktop. There are a variety of extensions for doing this that exist for Chrome, and all of them function pretty much exactly the same. For our task, I used [this one.](https://chrome.google.com/webstore/detail/apk-downloader/idkigghdjmipnppaeahkpcoaiphjdccm?hl=en) 

 Put in the URL to the Google Store page, generate the link, and download the *APK*. If one exists, choose to also download the *OBB*, or *Opaque Binary Blob*  as well. It typically stores large files such as graphics, models, and sounds.

 Now that you have acquired all the necessary files, we can begin the most enjoyable part of reverse engineering. Thankfully, both *APK* and *OBB* files use standard zip compression, meaning it is a trivial matter to click on them, rename the extension to ".zip", and extract. However--doing so is tedious, and is prone to human error due to the large amount of actions you need to perform each time. 

 As such, it becomes necessary to automate the process. Having made a personal working directory in my home folder with the Android files in it, I created a few bash scripts to help speed up the process of reverse engineering. Feel free to create your own folder layout, and modify the scripts to suit your preferences:

 Reset folder and delete unpacked files:
```bash
#!/bin/bash
set -xe
POS=$HOME/<Your_Working_Directory>
rm -r -f $POS/apk
mkdir $POS/apk
mkdir $POS/apk/patch
mkdir $POS/apk/patch/jar
mkdir $POS/apk/patch/java
mkdir $POS/apk/source
mkdir $POS/apk/finished
```

 Copy the *APK* and *OBB* files to the source code folder:
```bash
#!/bin/bash
set -xe
POS=$HOME/<Your_Working_Directory>
cp -f $POS/evil-app.apk $POS/apk/source
cp -f $POS/evil-app.obb $POS/apk/source
```

 Fully unpack the *APK* and *OBB*:
```bash
#!/bin/bash
set -xe
POS=$HOME/<Your_Working_Directory>/apk/source
unzip $POS/evil-app.apk -d $POS
unzip $POS/evil-app.obb -d $POS
rm $POS/evil-app.apk
rm $POS/evil-app.obb
```

 Congratulations! Now that you have fully extracted the *APK*, you can look over the inner contents and determine the parts that it is made up of. As an example, you can go into the working directory, and use the `find` command to search for various file extensions:

```bash
$ find apk/source/ -name '*.so'
apk/source/lib/x86/libevil.so
apk/source/lib/x86/libunity.so
apk/source/lib/x86/libmono.so
apk/source/lib/armeabi-v7a/libevil.so
apk/source/lib/armeabi-v7a/libunity.so
apk/source/lib/armeabi-v7a/libmono.so

$ find apk/source/ -name '*.dex'
apk/source/classes.dex
apk/source/classes2.dex

$ find apk/source/ -name '*.dll'
apk/source/assets/bin/Data/Managed/mscorlib.dll
apk/source/assets/bin/Data/Managed/Assembly-Sharp.dll
...

```
 From the results, we can immediately see that our application uses native code (.so), Java (.dex), and C Sharp (.dll). While at first glance the amount of files may seem daunting, the steps needed to reverse engineer each are simple and straightforward, and I will go over in detail exactly what needs to be done.

