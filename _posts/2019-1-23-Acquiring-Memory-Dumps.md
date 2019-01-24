---
layout: post
title: Acquiring Memory Dumps During Runtime
categories: [Programming, Reverse, Engineering]
tags: [IDA, GDB, memory, dump, file]
---

 At some point or another, you will need to obtain a snapshot of your Android application's RAM. Simple and easy to pull off, doing so will almost certainly be helpful to your reverse-engineering efforts, regardless of the type of application you have. For that, we will use the wonderful program going by the name of *GDB*.

 First and foremost, we have the [*GNU Debugger*](https://en.wikipedia.org/wiki/GNU_Debugger), known more commonly as just *GDB*. Featuring a wide array of commands with its own bash-like interface, *GDB* is a powerful debugger that, when used in conjunction with IDA, lets you circumvent a significant number of anti reverse-engineering protections.

 I recommend using the precompiled *GDB* binary which comes with the Android Native SDK. If you run it, you should be greeted with a short message:

```
$ Android/Sdk/ndk-bundle/prebuilt/linux-x86_64/bin/gdb
GNU gdb (GDB) 7.11
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later...
...
For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb)
 
```

 Now that you are in the menu, you are free to explore try out the various commands. A full documentation can be [found here](http://www.gnu.org/software/gdb/documentation/). When ready, type in `quit` to return to the main console. 

 The next step, is to organize a remote debugging session. Since GDB and your target application are running on two different devices--a Linux computer and Android phone--you will need to copy the special `gdbserver` executable to your Android. Check that it matches your processor type, and move it onto your phone using `adb push`. With everything set up, we are now ready to begin.

 Run your desired application, and through `adb shell`, use the following command to identify the PID of your app:

```bash 
$ ps -A | grep '<Application_Name>'
```

 Then, simply activate your `gdbserver` on any port you want, and prove it with the PID. The rest of the process will be done on the computer:

```bash 
$ ./gdbserver --attach :1234 <Application_PID>
```

 Exit `adb shell`, and run your main *GDB* executable once more to bring up the menu. To connect to our phone, we will use the following command:

```bash 
(gdb) target remote <Phone_IP>:1234
```

With the two devices having been successfully connected, you should see *GDB* display all the libraries that it has loaded in the console. We are now only a few steps away from obtaining our memory dump, also known as a "Core File" on Linux and Android systems.

```bash 
(gdb) generate-core-file /<Path_to_Chosen_Directory>/memory-dump.core
```

Congratulations! Now you only have to wait for the process to finish, and then you'll have access to the entirety of the application's memory. Note that since most memory dump files often exceed a gigabyte in size, it is reccomended to first process it using an automatic program to search for relevant data that you want, before opening it in a hex-viewer for closer inspection.
