---
author: labuxiaojue
comments: true
date: 2011-03-09 15:57:56
layout: post
slug: about-java-override
title: 一个关于Java方法覆写的小注意点
wordpress_id: 23
categories:
- Java
tags:
- Java
---

先看两段代码:

    
    class Person{
        void print(){
            System.out.println("Person -> void print()") ;
        }
        public void fun(){
            this.print() ;
        }
    } 
    
    class Student extends Person{
        public void print(){
            System.out.println("Student -> public void print()") ;
        }
    }
    
    public class OverrideDemo01{
        public static void main(String args[]){
            new Student().fun() ;
        }
    }


上述代码运行结果为


> Student -> public void print()


<!-- more -->
再看下面:

    
    class Person{
        private void print(){
            System.out.println("Person -> void print()") ;
        }
        public void fun(){
            this.print() ;
        }
    }
    
    class Student extends Person{
        void print(){
            System.out.println("Student -> public void print()") ;
        }
    }
    
    public class OverrideDemo02{
        public static void main(String args[]){
            new Student().fun() ;
        }
    }


运行结果为:


> Person -> void print()


可以发现,虽然被子类覆写的方法已经拥有比父类更严格的访问权限,但是方法覆写的权限从private扩大为default不能算是方法覆写.
