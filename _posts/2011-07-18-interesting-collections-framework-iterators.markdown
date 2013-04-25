---
author: labuxiaojue
comments: true
date: 2011-07-18 03:07:54
layout: post
slug: interesting-collections-framework-iterators
title: 有趣的集合框架迭代器Iterator
wordpress_id: 170
categories:
- Java
tags:
- Java
---

现在是2：16A.M.失眠了@_@，更更博文。此刻大脑混沌，作为入门菜鸟，各位多多包涵。

众所周知，我们遍历某个数组中的元素可以用到while{..}、或者for循环结构，当然也可以用到foreach语法_(1)_。

而当我们需要遍历集合（容器）中的元素（例如Collection或者Mao子类实例中的成员）时，就需要请到迭代器接口Iteractor_(2)_出场啦！

接口作为一种契约关系，只要某类实现(implements)了该接口，就意味着此类遵循和获得了该接口制定的标准和功能，而Interator作为一个轻量级对象，实现了它就可以对不同类型的集合进行遍历和操作集合序列中的对象，而如何去取元素就交由hasNext()与next()方法去操心了。
<!-- more -->
怎样让一个集合返回一个Iterator，从而通过此Iterator进行访问集合中的元素？我们需要调用集合类中的iterator()方法，具体形式请看以下的语句范例:

    
    List students = new ArrayList();
    
    Iterator iter = students.iterator


拿到这个迭代器后，我们就可以开始准备获得集合的第一个元素了。

    
    while(iter.hasNext()){
    
        Student st = iter.next();
    
        System.out.println(st.getName());
    
    }


分析上例的next()方法，我们发现“next”这个语义是"下一个"的意思，而如此一来岂不是直接跳过第一个元素而访问的是第二个元素了？而实际代码执行后输出情况并非如此，它的确是逐个访问了。因此，我们需要将印象中的Iterator迭代器所处的访问位置从"元素身上"调整到"元素之间"，那么假设有n个元素，那迭代器所处位置的情况有n+1个。也就是说，当迭代器调用next()方法时，迭代器就会越过一个元素，到这个元素和下一个相邻元素之间，然后返回刚刚越过的那个元素的引用。见下图：

[![](/assets/post_img/interesting-collections-framework-iterators/iterator.png)](/assets/post_img/interesting-collections-framework-iterators/iterator.png)

由上面经验我们知道，迭代器操作(或访问)一个元素首先需要“越过”它，同理可以引申到remove()方法，我们现在要删除Students中的某个学生，就需要首先调用next()方法“越过”这个学生，如果再次调用remove()而没有next下个学生，就会抛出IllegalStageException异常，即不能直接删除两个相邻的元素。

    
    iter.remove();
    iter.remove();//非法，IllegalStateException异常
    
    iter.remove();
    iter.next();
    iter.remove();//正常删除


当然，如果刚好iter位于位于最后一个元素之前，我们调用两次next()方法，将会抛出一个NoSuchElementException异常。

    
    iter.next();//执行完此句iter位置处于最后一个元素之后
    iter.next();//非法，NoSuchElementException异常


补充一点，我们通过查看API，还可以发现有一个listIterator接口，它是Iterator的子类型，主要用于List类的访问，它相较Iterator的强大之处在于能进行双向遍历集合中的元素。因为对于一个无序的序列来说，遍历其中的元素的方式也是无序的，我们只要保证每个元素都能访问到即可，这也就是为什么Set做为一个无序存放对象的集合并不去关心单向遍历还是双向遍历的问题，因此它没有实现listIterator接口。

另外加点料（摘自网络）：
foreach语法是通过Iterable接口实现移动的，所以如果需要改变foreach语句的逻辑功能，其实质就是覆盖底层Iterable中Iterator接口中三个方法的逻辑实现。下面，我们实现反向的Foreach“效果”。

    
     import java.util.ArrayList;
     import java.util.Arrays;
     import java.util.Collection;
     import java.util.Iterator;
     @SuppressWarnings("serial")
     class ReversibleArrayList extends ArrayList {
         public ReversibleArrayList(Collection c) {
            super(c);
         }
    
         public Iterable reversed() {
            return new Iterable() {
                public Iterator iterator() {
                   return new Iterator() {
                       int current = size() - 1;
    
                       public boolean hasNext() {
                          return current > -1;
                       }
    
                       public T next() {
                          return get(current--);
                       }
    
                       public void remove() {
                       }
                    };
                }
            };
         }
     }
    
     public class AdapterMethodIdiom {
         public static void main(String[] args) {
            ReversibleArrayList ral = new ReversibleArrayList(
                   Arrays.asList("To be or not to be".split(" ")));
            for (String s : ral)
                System.out.print(s + " ");
            System.out.println();
            for (String s : ral.reversed())
                System.out.print(s + " ");
         }
     }


结果：
> To be or not to be

> be to not or be To

在这个例子中，ReversibleArrayList类又实现了一个Iterable接口，名为reversed()，将其底层Iterator接口中的三个方法的逻辑实现为反向的遍历，就达到了反向Foreach的效果。从这个示例中，我们还可以注意到：
ReversibleArrayList ral = new ReversibleArrayList（…）;
for (String s : ral)
//此处的ral其实是Iterable接口的引用，因为ReversibleArrayList类是实现Iterable接口的。

**_注释：_**
_ (1)Java SE5引入的一种新的更加简洁的for语法用于数组或容器，表示不必创建int变量去对由访问项构成的序列进行计数，foreach将自动产生每一项。_

_(2)当然迭代集合也可以使用foreach结构让代码更简洁，但我们会错过一些有趣的事儿，其实foreach语法是通过Iterable接口实现移动的，就是说任何实现Iterable接口的类，都可以将它用于foreach语句中。而且如此为之也就没有本文讨论的必要了^_*。_

**_参考资料:_**  
_(1)Think in Java_  
_(2)Core Java,Volume I_
