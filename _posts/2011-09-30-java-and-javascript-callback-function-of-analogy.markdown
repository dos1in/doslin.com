---
author: labuxiaojue
comments: true
date: 2011-09-30 20:48:33
layout: post
slug: java-and-javascript-callback-function-of-analogy
title: Java与JavaScript的回调函数类比
wordpress_id: 223
categories:
- Java
tags:
- Java
- Javascript
---

本月中旬开的题，现在终于把坑添好了，Orz。
Java与JavaScript（下文简称JS）除了名称上的相似之外，其它方面也并非风马牛不相及，因为同属OO家族，有些特性还是可以进行相互类比来帮助理解的：例如下面我们所要讨论的回调（Callback）：

所谓回调，全称回调函数（callback function）。回，转也。是指函数A的功能片段传递给函数B（注册的过程），当B触发了某条件后，去调用该功能片段的指向（实现），这时A即称为回调函数。如果用Java的方式来叙述的话，就是类A中封装了一个接口InterfaceC的方法，类B按InterfaceC的约定实现该方法后再把自己传回给A，由A进行调用。因为Java没有函数的概念，这时就不是“函数回调”，而是“类回调”了，但是思想还是一样的。网上看到句挺好的解释，回调就是：


> **If you call me, I will call back.**


这样做的好处是什么呢？最显著的一点即是分离方法与具体实现，让调用者与被调者解耦合。
<!-- more -->
一、先来看看JS中的回调应用

把回调的思想应用于JS，可以实现功能与内容的分离。一般来说，对于一个完善的网页页面，代码有三部分组成：内容（HTML代码）、外观（CSS代码）以及功能（JS代码）。而分离功能与内容，可以让我们的代码更容易建立与维护。杂糅总是抓狂的代名词。

JS中把函数看成变量，即把函数主体看成值，而函数名称则为变量名称。

普通的，JS中声明一个函数

    
    function callBack(){
        alert(“I'm a function”);
    }


再来看另一种函数声明方式：

    
    var callBack = function(){
        alert(“I'm a function”);
    }


上述这种方式如果舍去了函数变量名，只有一个function函数体的状态称为函数字面量（function literal）或匿名函数（anonymous function）。

因为变量直接可以保存函数的引用也就意味着对于函数，我们能像操纵变量一样去操纵它。

最简单而且是最常见的例子：

以前我们喜欢把JS代码指派给HTML的属性：`<body onload=”callBack()”>`

现在，更优雅的方式是在`<script type=”text/javascript”><script>`的JS代码内部中，使用函数引用设定回调函数，即`window.onload = callBack`。

此处调用函数在于网页载入完成后触发，相当于在浏览器调用`window.onload()`函数时，实际上调用的是`callBack()`函数，因为我们把后者的引用覆盖了前者属性（变量）中原先存放的引用。

但是现在有一个问题：假如我们现在的callBack()函数有参数传入该怎么办呢？因为变量名中一旦出现了“(args)“即意味着函数调用。

很简单，这时就需要我们刚刚提到的函数字面量了：

    
    window.onload = function(args){
        alert(args);
    }


此时，匿名函数一声明完，它的引用未通过中间变量的保存，直接覆盖了原函数变量中保存的引用。而对于其它事件的指派，只需要在onload中指派的匿名函数体中指派就可以了——让onload事件处理函数成为事件初始化函数。

关于回调，Ajax处理服务器响应的数据时也有其中的应用。

二、下面看看Java的回调方式：

2.1定义一个回调接口

    
    
    package callbackdemo.ijava.me;
    
    /**
    * 回调函数接口
    * @author doslin
    */
    public interface CallBack {
        public void execute();
    }


2.2答题者回答完立即通知提问者

    
    
    package callbackdemo.ijava.me;
    
    /**
    * 问题解决者
    * @author DosLin
    */
    public class Solver {
        private CallBack cb = null;//回调引用
    
        public void getQuestion(CallBack asker){
            cb = asker;
        }
    
        public void soloveIt(){
            System.out.println("I'm Solver,the problem is solved.");
        cb.execute();
        }
    
    }


2.3 提问者实现了回调接口，意味着有回调方法

    
    package callbackdemo.ijava.me;
    /**
    * 提问者
    * @author DosLin
    */
    public class Asker implements CallBack{
        public Asker(){
            System.out.println("I'm Asker,I hava a question..");
        }
    
        // 问题解决后响应方式
        @Override
        public void execute() {
            System.out.println("Thank you..");
        }
    
        public static void main(String[] args){ //测试
            Asker asker = new Asker();//提问者
            Solver solver = new Solver();//解决者
    
            solver.getQuestion(asker);//看到提问者发问
            solver.soloveIt();//通知提问者问题被解决
        }
    }


综上，回调函数的重要性不言而喻，它可以取代代码中的直接调用函数，使之成为创建回调函数，当且仅当特定事件触发后才会启动回调函数。
