---
layout: post
title: Instrumenting Java Bytecode With ASM
categories: [Programming, Reverse, Engineering]
tags: [Java, Class, Bytecode, Instrumentation, ASM, IntelliJ]
---

 Having gone over the Java classes of the application, you now wish to instrument them by adding logging information to let you track their activity. Since manually going through every file will take an exorbitant amount of time however, we will instead create a script to automatically edit the bytecode for us with the help of a useful library called [ASM](https://asm.ow2.io/).




 ASM works by utilizing the [Visitor Pattern](https://en.wikipedia.org/wiki/Visitor_pattern) to modify bytecode. In a sense, if ASM was the device that controlled the complicated main job of reading through and parsing all the code within the `.class` file, the Visitor that you provide would be the simple set of rules that decided what to do with the specific piece of data that they were passed down.


 You can refer to [this tutorial](http://web.cs.ucla.edu/~msb/cs239-tutorial/) for a closer look at how to get started with ASM.
 
 To help me analyze the application which I was reverse engineering, I created a visitor that modified every single method of a given class to print logging information upon activating, as well as before and after any calls inside the function itself:
 
```java
class AddLeaveEnterLoggingToMethodAdapter extends MethodVisitor implements Opcodes {
    private final String className;
    private final String methodName;
    private final String currentDesc;
    
    public AddLeaveEnterLoggingToMethodAdapter(final MethodVisitor mv, String className, String name, String desc) {
        super(ASM5, mv);
        this.className = className;
        this.methodName = name;
        this.currentDesc = desc;
    }

    @Override
    public void visitCode() {
        mv.visitLdcInsn("ENTERED_METHOD");
        mv.visitLdcInsn(className + "." + methodName + currentDesc);
        mv.visitMethodInsn(INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
        mv.visitInsn(POP);
        mv.visitCode();
    }

    @Override
    public void visitMethodInsn(int opcode, String owner, String calledName, String calledDesc, boolean itf) {
        mv.visitLdcInsn("BEGIN_CALL");
        mv.visitLdcInsn(className + "." + methodName + currentDesc +": CALL " + owner + "." + calledName + calledDesc);
        mv.visitMethodInsn(INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
        mv.visitInsn(POP);
        
        mv.visitMethodInsn(opcode, owner, calledName, calledDesc, itf);
        
        mv.visitLdcInsn("END_CALL");
        mv.visitLdcInsn(className + "." + methodName + currentDesc + ": CALL " + owner + "." + calledName + calledDesc);
        mv.visitMethodInsn(INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
        mv.visitInsn(POP);
    }

    @Override
    public void visitInsn(int opcode) {
        switch (opcode) {
            case Opcodes.IRETURN:
            case Opcodes.FRETURN:
            case Opcodes.ARETURN:
            case Opcodes.LRETURN:
            case Opcodes.DRETURN:
            case Opcodes.RETURN:
                mv.visitLdcInsn("LEFT_METHOD");
                mv.visitLdcInsn(className + "." + methodName + currentDesc);
                mv.visitMethodInsn(INVOKESTATIC, "android/util/Log", "i", "(Ljava/lang/String;Ljava/lang/String;)I", false);
                mv.visitInsn(POP);
                break;
            default:
        }
        mv.visitInsn(opcode);
    }
}
```
 Since ASM uses no decompilation in the process, it means we are free to instrument and modify any `.class` that we want regardless of its complexity and whether it was protected from decompilation or not.