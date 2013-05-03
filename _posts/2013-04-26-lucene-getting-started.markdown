---
date: 2013-04-26 14:45:10
title: Lucene官方示例入门
layout: post
tags:
    - Java
    - Lucene
categories:
    - Java
---
配置环境
-------
首先下载最新版的[Lucene分发包](http://www.apache.org/dyn/closer.cgi/lucene/java/)（如：lucene-4.2.1.tgz），并解压到工作目录，找到以下四个jar包：

>core\lucene-core-{version}.jar  
>queryparser\lucene-queryparser-{version}.jar  
>analysis\common\lucene-analyzers-common-{version}.jar  
>demo\lucene-demo-{version}.jar  

将上述jar文件放入某个目录（如：D:\lucene）并配置到CLASSPATH中，追加环境变量中的CLASSPATH内容为

>;D:\lucene\lucene-core-4.2.1.jar;D:\lucene\lucene-queryparser-4.2.1.jar;D:\lucene\lucene-analyzers-common-4.2.1.jar;D:\lucene\lucene-demo-4.2.1.jar;

运行例子
-------
确保你的CLASSPATH设置无误之后，我们下面使用演示例子创建索引文件。在控制台中键入以下命令：

    cd D:\lucene
    java org.apache.lucene.demo.IndexFiles -docs {path-to-lucene}/src

这将会对lucene的源码目录创建索引，并将索引文件放在D:\lucene下的index目录中。

之后我们根据创建的索引文件来搜索文件，键入：

    java org.apache.lucene.demo.SearchFiles
    
程序将提示我们输入要查询的字符`（Enter query:）`，输入`string`回车，将会返回给我们十个匹配结果，输入`n`回车，会再返回十个结果，输入`q`退出。

源码解析
-------
下面我们将浏览一遍这个lucene命令行例子，分解一下它的函数以便于理解如何在我们的程序中应用Lucene。  
[IndexFiles.java](http://lucene.apache.org/core/4_2_1/demo/src-html/org/apache/lucene/demo/IndexFiles.html)是创建索引的代码。  
[SearchFiles.java](http://lucene.apache.org/core/4_2_1/demo/src-html/org/apache/lucene/demo/SearchFiles.html)是根据索引来检索的代码。


>###**索引文件**

main()函数接受命令行中的参数，之后通过构造一个[Directory](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/store/Directory.html)对象，并实例化[StandardAnalyzer](http://lucene.apache.org/core/4_2_1/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html)和[IndexWriterConfig](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/index/IndexWriterConfig.html)作为实例化[IndexWriter](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/index/IndexWriter.html)的准备工作。

命令行中的`-index`参数接收的是存放索引文件的目录名称。如果在相对路径下用`-index`参数或者没有`-index`参数的情况下（将会使用默认的索引文件夹名index）调用IndexFiles，这个索引文件夹将会在当前工作目录下作为子目录创建（如果还没创建的话）。但在有一些平台中，这个索引文件夹的创建路径可能不同（比如会在用户的主目录中创建）。

`-docs`参数接收的是存放那些要进行创建索引的目标文件的文件夹路径。

`-update`参数告诉IndexWriter类：不要删除已经存在的索引文件。如果没给出`-update`参数，IndexFiles将会在创建新的索引文件前清空旧索引。

IndexFiles中使用的[Directory](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/store/Directory.html)类是为了在索引中储存信息。另外这个类的一个实现[FSDirectory](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/store/FSDirectory.html)，它有三个子类分别实现向内存、数据库等等其它介质中写入索引。

Analyzer类也称为将文本拆分为可被索引的词元（tokens）的处理管道。它会对这些词元中的词（Term）进行一些可选操作，比如大小写转换，插入同义词以及过滤不想要的词元等等。我们这里使用的词法分析器是StandardAnalyzer，它使用在[Unicode Standard Annex #29](http://unicode.org/reports/tr29/)中指定的Unicode Text Segmentation算法作为单词拆分规则，接着将词元转换成小写形式，然后过滤出停词（stopword）。停词指的是一般语言中的冠词（a,an,the,etc。）和那些没有检索价值的词元。值得注意的是不同的语言有不同的拆分规则。目前Lucene针对不同的语言提供了许多[分析器](http://lucene.apache.org/core/4_2_1/analyzers-common/overview-summary.html)。

IndexWriterConfig的实例保存了IndexWriter类的配置。比如说，我们根据`-update`参数设置了OpenMode。

往下看，在IndexWriter实例后面，你可以看到indexDocs()代码。这个递归函数爬取指定目录下的所有文件并为它们建立[Document](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/document/Document.html)对象。Document对象是表示相应文件的文本内容，创建时间以及位置等信息的简单数据对象。这些实例被添加到IndexWriter中。如果给定了`-update`参数，IndexWriterConfig OpenMode将被设置为[OpenMode.CREATE_OR_APPEND](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/index/IndexWriterConfig.OpenMode.html#CREATE_OR_APPEND)，代表IndexWriter将对已经拥有相同标识符（在上述例子中，文件路径作为该标识符）的已建立索引的文档的索引进行更新操作，不存在则创建。反之，不给定`-update` 参数将表示删除先前已索引的文档，在目录中创建新索引。

>###**搜索文件**

[SearchFiles](http://lucene.apache.org/core/4_2_1/demo/src-html/org/apache/lucene/demo/SearchFiles.html)类很简单。它主要通过和[IndexSearcher](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/search/IndexSearcher.html)，[StandardAnalyzer](http://lucene.apache.org/core/4_2_1/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html)(在IndexFiles中也用到了）以及[QueryParser](http://lucene.apache.org/core/4_2_1/queryparser/org/apache/lucene/queryparser/classic/QueryParser.html)协作。查询语句解析器是由分析器来构建的，和之前文档解析的方式一样用于解析你的查询语句：搜寻单词边界，大小写转换以及移除那些像 ‘a’，‘an’，和‘the’ 的无用单词。[Query](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/search/Query.html)对象包含了传给扫描器的[QuqeryParser](http://lucene.apache.org/core/4_2_1/queryparser/org/apache/lucene/queryparser/classic/QueryParser.html)的结果。需要注意的是，以编程方式去构造一个富[Query](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/search/Query.html)对象而不是用查询语句解析器是可行的。因为查询语句解析器仅仅是能够将[Lucene query syntax](http://lucene.apache.org/core/4_2_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html#package_description)解码为相应的[Query](http://lucene.apache.org/core/4_2_1/core/org/apache/lucene/search/Query.html)对象。

参考文档
-------
[http://lucene.apache.org/core/4_2_1/demo/overview-summary.html](http://lucene.apache.org/core/4_2_1/demo/overview-summary.html)
