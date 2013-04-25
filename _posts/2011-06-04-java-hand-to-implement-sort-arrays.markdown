---
author: labuxiaojue
comments: true
date: 2011-06-04 13:43:44
layout: post
slug: java-hand-to-implement-sort-arrays
title: Java人肉实现数组排序
wordpress_id: 126
categories:
- Java
tags:
- Java
---

这里没有使用Arrays.sort()排序方法，人肉手动实现几种经典排序算法。包括选择排序、冒泡排序、插入排序。

    
    
    public static int[] selectSort(int[] ary) {// 选择排序
    
    		for (int i = 0; i < ary.length - 1; i++) {
    			for (int j = i + 1; j < ary.length; j++) {// 每轮选择一个最小的数与前一个数交换
    				if (ary[i] > ary[j]) {
    					int temp = ary[i];
    					ary[i] = ary[j];
    					ary[j] = temp;
    				}
    			}
    		}
    
    		return ary;
    	}
    
    
    public static int[] bubbleSort(int[] ary) {// 冒泡排序
    		// 比较相邻元素，把每轮最大的一个移到最后
    		for (int i = 0; i < ary.length - 1; i++) {
    			for (int j = 0; j < ary.length - 1 - i; j++) {
    				if (ary[j] > ary[j + 1]) {
    					int temp = ary[j];
    					ary[j] = ary[j + 1];
    					ary[j + 1] = temp;
    				}
    			}
    		}
    
    		return ary;
    	}
    

    public static int[] insertSort(int[] ary) {// 插入排序
    		// 假设前半段是有序数组，将后半段元素插入到有序数组中的合适位置
    		int i;
    		for (int j = 1; j < ary.length; j++) {// 遍历后半段数组
    			int temp = ary[j];
    			for (i = j - 1; i >= 0; i--) {
    				if (temp < ary[i]) {
    					ary[i + 1] = ary[i];// 后移
    				} else {
    					break;
    				}
    			}
    			// 因为发生了i--
    			// 若直接break 则没有进行i-- 意味着下述语句自己对自己赋值，结果不变
    			ary[i + 1] = temp;// 移完之后把临时变量置入
    
    		}
    
    		return ary;
    	}
    
