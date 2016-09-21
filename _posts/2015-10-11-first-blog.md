---
layout: post
title: JDK工具javap的使用
tags: nothing
categories: Java
---

<div class="toc"></div>

<br/>
javap是JDK自带的反汇编器，可以查看java编译器为我们生成的字节码。通过它，可以对照源代码和字节码，从而了解很多编译器内部的工作。
windows环境下查看javap自带工具选项：  
![](http://i.imgur.com/jex8Ghb.png)

平常工作中 -c 反汇编比较常用，下面写个简单的例子看一下

	public class Synchronized {
	    public static void main(String[] args) {
	        synchronized (Synchronized.class) {
	
	        }
	
	        m();
	    }
	
	    public static synchronized void m() {
	
	    }
	}

<br/>
执行 javap Synchronized
    
	Compiled from "Synchronized.java"
	public class Synchronized {
	  public Synchronized();
	  public static void main(java.lang.String[]);
	  public static synchronized void m();
	}

由此可以看出如果没有构造函数，编译器会默认生成一个。

执行 javap -c Synchronized  对代码进行反汇编

	Compiled from "Synchronized.java"
	public class Synchronized {
	  public Synchronized();
	    Code:
	       0: aload_0
	       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
	       4: return
	
	  public static void main(java.lang.String[]);
	    Code:
	       0: ldc           #2                  // class Synchronized
	       2: dup
	       3: astore_1
	       4: monitorenter
	       5: aload_1
	       6: monitorexit
	       7: goto          15
	      10: astore_2
	      11: aload_1
	      12: monitorexit
	      13: aload_2
	      14: athrow
	      15: invokestatic  #3                  // Method m:()V
	      18: return
	    Exception table:
	       from    to  target type
	           5     7    10   any
	          10    13    10   any
	
	  public static synchronized void m();
	    Code:
	       0: return

这里显示的是编译器执行具体的字节码，可以看到Synchronized类中对于同步块的实现使用了monitorenter和monitorexit指令。  
<br/>
执行javap -v Synchronized 输出附加信息
  
	 public static void main(java.lang.String[]);
	   descriptor: ([Ljava/lang/String;)V
	   flags: ACC_PUBLIC, ACC_STATIC
	   Code:
	     stack=2, locals=3, args_size=1
	        0: ldc           #2                  // class Synchronized
	        2: dup
	        3: astore_1
	        4: monitorenter
	        5: aload_1
	        6: monitorexit
	        7: goto          15
	       10: astore_2
	       11: aload_1
	       12: monitorexit
	       13: aload_2
	       14: athrow
	       15: invokestatic  #3                  // Method m:()V
	       18: return
	     Exception table:
	        from    to  target type
	            5     7    10   any
	           10    13    10   any
	     LineNumberTable:
	       line 6: 0
	       line 8: 5
	       line 10: 15
	       line 11: 18
	     LocalVariableTable:
	       Start  Length  Slot  Name   Signature
	           0      19     0  args   [Ljava/lang/String;
	     StackMapTable: number_of_entries = 2
	       frame_type = 255 /* full_frame */
	         offset_delta = 10
	         locals = [ class "[Ljava/lang/String;", class java/lang/Object ]
	         stack = [ class java/lang/Throwable ]
	       frame_type = 250 /* chop */
	         offset_delta = 4
	
	 public static synchronized void m();
	   descriptor: ()V
	   flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
	   Code:
	     stack=0, locals=0, args_size=0
	        0: return
	     LineNumberTable:
	       line 15: 0

上边是截取的一段输出信息，print stack size, number of locals and args for methods，从输出的flags信息可以看到同步方法是依靠方法修饰符上的ACC_SYNCHRONIZED来完成同步的。

<br/>