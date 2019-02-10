---
layout: post
title: Modifying Java Bytecode With ASM
categories: [Programming, Reverse, Engineering]
tags: [Java, Class, Bytecode, Instrumentation, ASM, IntelliJ]
---

 In a previous blog post, we covered how to use the ASM library to [instrument Java code](/Instrumenting-Bytecode). In conjunction with instrumentation, ASM can also be used to modify classes and change how they behave entirely, often in dramatic ways.
 
 <!--more-->

 During the process of reverse engineering, you may encounter a method which checks an unwanted condition that leads to a behavior which you would like to elimitate. In this case, simply removing the method body altogether might be a possibility. Here is an example of a visitor that does just that:
 
```java
class RemoveCodeMethodAdapter extends MethodVisitor implements Opcodes {
    private final String className;
    private final String methodName;

    public RemoveCodeMethodAdapter(final MethodVisitor mv, String className, String name) {
        super(ASM5, mv);
        this.className = className;
        this.methodName = name;
    }

    @Override
    public void visitCode() {
        mv.visitLdcInsn("SKIP");
        mv.visitLdcInsn(className + "." + methodName + "()");
        mv.visitMethodInsn(INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
        mv.visitInsn(POP);
        mv.visitInsn(RETURN);
        super.visitCode();
    }
}
```
  In the process of removal, the Visitor will add a helpful logging statement notifying you of whenever the method body that you deleted is called. 
  
  Lastly, you may want to add your own custom Java code to the beginning of a method. Instead of trying to re-write it all inside the target `.class` file, it is much simpler to compile it as a separate file, and add a call to it in the original.
    
```java
  class InsertCustomMethodCall extends MethodVisitor implements Opcodes {
    private final MethodVisitor methodVisitor;
 
    public InsertCustomMethodCall(final MethodVisitor mv) {
        super(ASM5, mv);
        this.methodVisitor = mv;
    }

    @Override
    public void visitCode() {
        mv.visitMethodInsn(INVOKESTATIC, "com/package/ExampleClass", "exampleMethod", "()V", false);
        mv.visitCode();
    }
}
```
  
  When the procedure finishes, the target method will call your `static void exampleMethod()`, from your `com.package.ExampleClass`.