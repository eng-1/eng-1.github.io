---
layout: post
title: Decompiling Java Classes
categories: [Programming, Reverse, Engineering]
tags: [Fernflower, IntelliJ, Java, Class, Decompile, Bash]
---

 Often times, you will be faced with compiled Java code that you must analyze. With `.class` files in particular representing a significant amount of most applications' programming, being able to decompile them can grant you unique knowledge of the inner workings of the system.
  
<!--more-->

  
 While it is certainly possible to open them in IntelliJ, doing so for every single `.class` file is tedious, and may not show all of the information. For maximum efficiency, instead of opening them in IntelliJ, we will instead directly use the decompiler engine that powers it--[*Fernflower*.](https://github.com/JetBrains/intellij-community/tree/master/plugins/java-decompiler/engine) Developed by Stiver and maintained by JetBrains, Fernflower offers a simple way to decompile `.class` files in bulk.

 The most immediate step is to acquire the needed source code. Although Fernflower does not come with ready-to-use binaries, it is a simple matter to get them. This command will download the most recent version of the repository, and build for us a `.jar` file that we can use:
 
 ```bash
 git clone --depth=1 https://github.com/JetBrains/intellij-community
 cd intellij-community/plugins/java-decompiler/engine
 gradle build
 ```
 
 Having ran gradle and acquired an executable, we can now finally use Fernflower to decompile any `.class` files located in the APK.
 
 Here is an example of how to use the new `fernflower.jar` file we just built:

```bash
java -jar build/libs/fernflower.jar  <Your_.class_Directory>  <Your_Output_Directory>
```

 To run the newly-made binary, simply give it two additional arguments: the folder containing the classes you want to decompile, and the output directory into which the results will be placed. Now, you are free to open the results in any text editor that you wish, and analyze what each Java file does.
 
 In the case of Fernflower failing to decompile a `.class` file, it will print out an error message that you will be able to notice. In the future, I will cover a few common anti-decompilation tricks used to fool decompilers like Fernflower, and how to get around them.