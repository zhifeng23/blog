---
layout: post
title: 异常处理模板
tags: exception template 
categories: Java
---

<div class="toc"></div>

<br/>
----------

以往繁琐的异常处理
---------

正确的异常处理的代码可能写起来很乏味，try-catch模块可能会使代码变的难以阅读，看看下面的例子：

    Input       input            = null;
    IOException processException = null;
    try{
        input = new FileInputStream(fileName);

        //...process input stream...
    } catch (IOException e) {
        processException = e;
    } finally {
       if(input != null){
          try {
             input.close();
          } catch(IOException e){
             if(processException != null){
                throw new MyException(processException, e,
                  "Error message..." +
                  fileName);
             } else {
                throw new MyException(e,
                    "Error closing InputStream for file " +
                    fileName;
             }
          }
       }
       if(processException != null){
          throw new MyException(processException,
            "Error processing InputStream for file " +
                fileName;
    }

在这个例子中，任何异常都没有丢失，如果一个异常时从try块抛出，另外一个异常时在input.close()时抛出，两个异常都将保存在MyException实例中，依次调用堆栈。

为了不丢失任何异常，这得需要多少代码才能处理一个输入流。实际上，他只是捕获了IOException。

RuntimeExceptions从try块抛出但没有捕获，如果input.close（）调用发生了异常，是不是很丑陋？是不是很难去阅读代码？每次处理输入流的时候你会记得处理所有异常代码吗？

幸运的是，有一个简单的设计模式，模板方法，能够帮助你每次都能够正确的处理异常，甚至都不用去看或者改动你的代码。好吧，写模板还是需要的。

你要做的是把所有的异常处理代码放在一个模板内，而模板仅仅是一个普通的类。下面是上述异常处理流的模板类：

    public abstract class InputStreamProcessingTemplate {

    public void process(String fileName){
        IOException processException = null;
        InputStream input = null;
        try{
            input = new FileInputStream(fileName);

            doProcess(input);
        } catch (IOException e) {
            processException = e;
        } finally {
           if(input != null){
              try {
                 input.close();
              } catch(IOException e){
                 if(processException != null){
                    throw new MyException(processException, e,
                      "Error message..." +
                      fileName);
                 } else {
                    throw new MyException(e,
                        "Error closing InputStream for file " +
                        fileName;
                 }
              }
           }
           if(processException != null){
              throw new MyException(processException,
                "Error processing InputStream for file " +
                    fileName;
        }
    }

    //override this method in a subclass, to process the stream.
    public abstract void doProcess(InputStream input) throws IOException;
    }

所有的异常处理都被放在了InputStreamProcessingTemplate这个抽象类中。请注意，在try-catch块内 process（）方法是如何调用doProcess（）方法的。你将通过继承这个模板类，并覆盖doProcess（）方法来使用这个模板。可以这样实现：

     new InputStreamProcessingTemplate(){
        public void doProcess(InputStream input) throws IOException{
            int inChar = input.read();
            while(inChar !- -1){
                //do something with the chars...
            }
        }
     }.process("someFile.txt");

这个例子创建了InputStreamProcessingTemplate类的匿名子类，实例化子类的实例，并调用其process（）方法。

这样写起来简单并且容易阅读。只有域逻辑的代码可见。编译器会检查你是否正确继承了InputStreamProcessingTemplate。当在正式使用的时候，通常会借助于IDE的代码，因为IDE会同时识别doProcess（）和process（）方法。

现在你可以在你的代码任何有文件输入流操作的地方使用InputStreamProcessingTemplate，你也可以通过简单的修改模板以适应于各种不同的输入流，而不仅仅是file。

使用接口而不是继承类
----------

你可以通过实现InputStreamProcessor接口而不是继承InputStreamProcessingTempate的方式。下面来看一下如何实现的：

    
    public interface InputStreamProcessor {
        public void process(InputStream input) throws IOException;
    }
    public class InputStreamProcessingTemplate {

    public void process(String fileName, InputStreamProcessor processor)     {
        IOException processException = null;
        InputStream input = null;
        try{
            input = new FileInputStream(fileName);

            processor.process(input);
        } catch (IOException e) {
            processException = e;
        } finally {
           if(input != null){
              try {
                 input.close();
              } catch(IOException e){
                 if(processException != null){
                    throw new MyException(processException, e,
                      "Error message..." +
                      fileName;
                 } else {
                    throw new MyException(e,
                        "Error closing InputStream for file " +
                        fileName);
                 }
              }
           }
           if(processException != null){
              throw new MyException(processException,
                "Error processing InputStream for file " +
                    fileName;
        }
      }
    }

注意在模板的process()方法中额外的参数InputStreamProcessor，是在try块内processor.process(input)处调用的。下面来看一下如何使用这个模板：

        new InputStreamProcessingTemplate()
        .process("someFile.txt", new InputStreamProcessor(){
            public void process(InputStream input) throws IOException{
                int inChar = input.read();
                while(inChar !- -1){
                    //do something with the chars...
                }
            }
        });

这看起来跟上文提到的方式并没有太大的不同，除了调用InputStreamProcessingTemplate.process() 时更靠近代码的表层，这样看起来更容易阅读一些。

静态模板方法
------
也可以通过静态方法来生成模板方法，这样就不需要再每次调用的使用实例化模板方法，下面来看一下InputStreamProcessingTemplate的静态方法实现。

    public class InputStreamProcessingTemplate {

    public static void process(String fileName,
    InputStreamProcessor processor){
        IOException processException = null;
        InputStream input = null;
        try{
            input = new FileInputStream(fileName);

            processor.process(input);
        } catch (IOException e) {
            processException = e;
        } finally {
           if(input != null){
              try {
                 input.close();
              } catch(IOException e){
                 if(processException != null){
                    throw new MyException(processException, e,
                      "Error message..." +
                      fileName);
                 } else {
                    throw new MyException(e,
                        "Error closing InputStream for file " +
                        fileName;
                 }
              }
           }
           if(processException != null){
              throw new MyException(processException,
                "Error processing InputStream for file " +
                    fileName;
        }
    }}
 process(...)方法也被定义为静态的，下面看一下如何调用这个方法。
 

        InputStreamProcessingTemplate.process("someFile.txt",
        new InputStreamProcessor(){
            public void process(InputStream input) throws IOException{
                int inChar = input.read();
                while(inChar !- -1){
                    //do something with the chars...
                }
            }
        });

总结
--
异常处理模板是一个可以提高代码的质量和可读性的简单而强大的机制。这也增加了你的工作效率，可以减少代码量，并使代码更加的整洁。

模板方法的设计方案可以用于其他目的，而不仅仅是异常处理。输入流的迭代也可以放在一个模板里。在JDBC的ResultSet的迭代也可以放在一个模板里。正确执行JDBC事务，可以放入一个模板。像这样的处理流程还有很多。

本文译自：[Exception Handling Templates in Java](http://tutorials.jenkov.com/java-exception-handling/exception-handling-templates.html)
