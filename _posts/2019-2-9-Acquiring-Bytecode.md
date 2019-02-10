---
layout: post
title: Analyzing Undecompilable Java Classes
categories: [Programming, Reverse, Engineering]
tags: [Java, Class, Javap, Bytecode, Decompile, Bash]
---

 At some point, you will come upon Java classes that resist attempts at decompilation. Whether a method or an entire file, all seem to be impervious to the decompilation software that you use. More often than not, this means that you are on the right track, and that the key to reverse engineering the application lies right ahead.
 
 <!--more-->

 There are numerous ways that exist to trick decompilers and prevent them from properly functioning. Frequently, it involves the usage of a convoluted series of `goto` instructions. Since JVM is much less restrictive than Java when it comes to the ordering of commands, it means that bytecode which runs fine on your computer, might not necessarily be decompilable into proper Java. 
 
 By exploiting these differences in complexity in conjunction with bugs in the decompiler itself, software engineers can create garbage methods that, despite not actually being used by the application, serve to throw off the decompiler.
 
 Luckily, there exists a way with which we can view the bytecode of `.class` files directly, and determine if that is the case--the `javap` command:
 
 ```bash
 $ javap -c evil-file.class
 
 class MethodAdapter extends org.objectweb.asm.MethodVisitor implements org.objectweb.asm.Opcodes {
   public MethodAdapter(org.objectweb.asm.MethodVisitor);
     Code:
        0: aload_0
        1: ldc           #2    // int 327680
        3: aload_1
        4: invokespecial #3    // Method org/objectweb/asm/MethodVisitor."<init>":(ILorg/objectweb/asm/MethodVisitor;)V
        7: return
 
   public void visitMethodInsn(int, java.lang.String, java.lang.String, java.lang.String, boolean);
     Code:
        0: aload_0
        1: getfield      #4    // Field mv:Lorg/objectweb/asm/MethodVisitor;
        4: sipush        178
        ...
 ```
 
 [Here is a good tutorial](https://www.javaworld.com/article/2077233/core-java/bytecode-basics.html) to help familiarize yourself with the basics of JVM bytecode. Now that you have obtained the inner contents of the `.class` file, you are able to go over it, and roughly gauge what it does.
 