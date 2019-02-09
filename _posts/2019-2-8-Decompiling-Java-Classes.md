---
layout: post
title: Decompiling Java Classes
categories: [Programming, Reverse, Engineering]
tags: [Fernflower, Java, Class, Decompile, Bash]
---

 Often times, you will be faced with Java code that you must edit. With `.class` files in particular representing a significant amount of most applications' programming, being able to decompile them can grant you unique knowledge of the inner workings of the system.
  
 For this task, I used an open-source decompiler called [*Fernflower*.](https://github.com/JetBrains/intellij-community/tree/master/plugins/java-decompiler/engine) Developed by Jetbrains--the creators of the Intellij and CLion IDEs among others--Fernflower offers a simple way to prepare `.class` files for analysis.

 The most immediate step is to acquire the needed source code. Since we are only going to be using Fernflower, it is fine to download the entire `intellij-community` repository and delete everything except the `plugins/java-decompiler` folders.
 
 Now that you have downloaded the material, we can proceed. Although Fernflower does not actually come with an executable that you can run from the get-go, it is nonetheless incredibly easy to obtain one. For that, we will simply go into the `intellij-community/plugins/java-decompiler/engine` directory, and type in:
 
```bash
gradle build
```
 
 Upon doing so, the `gradle` command will run, and create for us a fully-functioning `.jar` file inside the `build/libs` folder. Having acquired it, we can now finally use Fernflower to decompile any `.class` files located in the APK.
 
 Here is an example of how to use the new `fernflower.jar` file we just built. Switch back to your console, and type:

```bash
java -jar $HOME/intellij-community-master/plugins/java-decompiler/engine/build/libs/fernflower.jar  $HOME/<Your_Working_Directory>/evil.class  $HOME/<Your_Working_Directory>
```

 When you hit enter, your computer will run the newly-made `fernflower.jar` file, along with two additional arguments: the class you want to decompile, and the output directory. Now, you are free to open up and analyze the decompiled `.java` file in any text editor that you wish.
 
 In the case of Fernflower failing to decompile the `.class` file, it will print out an error message that you will be able to notice. In the future, I will cover a few common anti-decompilation tricks used to fool decompilers like Fernflower, and how to get around them.