---
date: 2014-03-11 14:30:40
layout: post
title: "55分钟学会正则表达式(译)"
tags:
    - regular expressions
categories:
    - regular expressions
---

<link href="../assets/css/regular-code.css" type="text/css" rel="stylesheet"></link>

<script type="text/javascript">
	// For flipping between displayed things
	var toggle = function (id) {
		var e = document.getElementById(id);
		e.style.display = e.style.display === "none" ? "" : "none";
	};
</script>

[原文地址-Sam Hughes](http://qntm.org/files/re/re.html)

翻译水平有限，如有谬误，欢迎评论斧正或者[Pull Request](https://github.com/JChord/jchord.github.io)。

正则表达式（“regexes”）即**增强查找/字符串替换操作**。当在文本编辑器中编辑文字时，正则表达式经常用于：

* 检查文本是否包含一个给定的模式
* 查找任何匹配的模式
* 从文本中拉取信息（比如截断）
* 修改文本

和文本编辑器一样，绝大多数高级编程语言支持正则表达式。在本文中，“文本”仅仅是一个字符串变量，但是有效的操作却是一致的。某些编程语言（Perl，JavaScript）甚至为正则表达式提供专用的语法。

**但是正则表达式是什么？**

一个正则表达式仅仅为一个字符串。它没有长度限制，但是通常该字符串很短。下面看几个例子：

* `I had a \S+ day today`
* `[A-Za-z0-9\-_]{3,16}`
* `\d\d\d\d-\d\d-\d\d`
* `v(\d+)(\.\d+)*`
* `TotalMessages="(.*?)"`
* `<[^<>]>`

这个字符串实际上是一个极小的计算程序，并且正则表达式是一门语法小而简洁，[*领域特定的编程语言*](http://en.wikipedia.org/wiki/Domain-specific_language)。牢记以下几点，它们不该在学习过程中让你感到惊讶：

* 每个正则表达式都能分解成一串*指令*。“找到这个，再找到那个，然后找到其中一个...”
* 一个正则表达式拥有*输入*（文本）和*输出*（模式匹配，和有些时候的自定义文本）。
* 存在语法错误——不是每个字符串都是合法的正则表达式！
* 语法有些怪异，也可以说是恐怖。
* 一个正则表达式有时候可以被*编译*以便更快运行。  

正则实现一直有着显著的改变。对于本文，我所关注的是那些几乎每个正则表达式都实现了的核心语法。  

<div class="exercise">
<h4>练习</h4>
<p>获取一个支持正则的文本编辑器。我推荐<a href="http://notepad-plus-plus.org/">Notepad++</a>。</p>
<p>下载一篇很长的散文故事比如<a href="http://www.gutenberg.org/cache/epub/35/pg35.txt">Gutenberg出版社出版的H. G. Wells的<i>《时光机器》</i></a>然后打开它。</p>
<p>下载一部字典，比如<a href="../assets/file/learn-regular-expressions-in-about-55-minutes/words.zip">这个</a>，解压然后打开。</p>
<p>一切准备就绪，稍后开始练习。</p>
</div>

<div class="warning">
<p><strong>提示：</strong> 正则表达式与<a href="http://en.wikipedia.org/wiki/Glob_%28programming%29">文件通配符</a>语法完全不兼容，比如<code>*.xml</code>。</p>
</div>


##正则表达式基础语法

###字面值(Literals)

正则表达式由只代表自身的*字面值*和代表特定含义的*元字符*组成。

这里也有一些例子。我会对元字符进行高亮。

* <code>I had a <em>\S+</em> day today</code>
* <code><em>[A-Za-z0-9\-_]{3,16}</em></code>
* <code><em>\d\d\d\d</em>-<em>\d\d</em>-<em>\d\d</em></code>
* <code>v<em>(\d+)(\.\d+)*</em></code>
* <code>TotalMessages="<em>(.*?)</em>"</code>
* <code>&lt;<em>[^&lt;&gt;]*</em>&gt;</code>

大部分字符，包括字母数字字符，会以字面值的形式出现。这意味着它们查找的是自身。比如，正则表达式`cat`代表“先找到`c`，接着找到`a`，最后找到`t`”。

目前为止感觉良好。这的确很像

* 一个普通的查找对话框
* Java中的`String.indexOf()`函数
* PHP中的`strpos()`函数
* 等等

提示：除非特别说明，正则表达式是大小写敏感的。然而，绝大多数实现都会提供一个*标记*来开启不区分大小写的功能。

###句点（dot）

我们第一个元字符是句号（译者注：句点，英文句号），`.`。一个`.`表示匹配任何单个字符。下面这个正则表达式`c.t`代表“先找到`c`，接着找到任何单个字符，再找到`t`”。

在一段文本中，这个表达式将会找到`cat`，`cot`，`czt`，甚至字面值为`c.t`的字符串（`c`，句点，`t`），但是不包括`ct`或者`coot`。

任何元字符如果用一个反斜杆`\`进行转义就会变成字面值。所以上述的正则表达式`c\.t`就代表“先找到`c`，接着找到句号，再找到`t`”。

反斜杠是一个元字符，这意味着它也可以使用反斜杠转义。所以正则表达式`c\\t`代表“先找到`c`，接着找到反斜杆，再找到`t`”。

<div class="warning">
<p><strong>注意！</strong> 在一些实现中，<code>.</code> 会匹配任意字符除了 <em>换行符</em>。这意味着“换行符”在不同的实现中也会变化。 要查看你的文档。在这篇文章中， 我会确保<code>.</code> 会匹配任意字符。</p>
<p>在其它情况下， 通常会有一个标记来调整这种行为，那就是`DOTALL`或类似的标记</p>
</div>

<div class="exercise">
<h4>练习</h4>
<p>使用你目前所学，在字典中使用正则表达式，匹配一个有两个<code>z</code>的单词，其中这两个<code>z</code>离得越远越好。</p>
<h4><button onclick="toggle('a1');">答案</button></h4>
<p id="a1" style="display: none;">你最终的正则表达式应该是<code>z.......z</code>会匹配到四个单词: <code>razzamatazz</code>，<code>razzamatazzes</code>，<code>zwischenzug</code>以及<code>zwischenzugs</code>。</p>
</div>

<div class="exercise">
<h4>练习</h4>
<p>在<i>《时光机器》</i>这本书中，使用正则表达式来查找以介词收尾的句子。</p>
<h4><button onclick="toggle('a2');">答案</button></h4>
<p id="a2" style="display: none;">你的正则表达式应该类似这样<code>up\.</code>。</p>
</div>

###字符类（Character classes）

字符类是字符在方括号中的集合。表示“找到其中任意的字符”。

* 正则表达式`c[aeiou]t`表示“找到`c`后跟一个元音字母，再找到`t`”。在一段文本中，将会匹配到`cat`，`cet`，`cit`，`cot`和`cut`。
* 正则表达式`[0123456789]`表示找到一个数字
* 正则表达式`[a]`和`a`意义相同：“找到`a`”

一些转义的例子：

* `\[a\]`表示“找到一个左方括号紧跟着一个`a`，再跟着一个右方括号”。
* `[\[\]ab]`表示“匹配一个左方括号或者右方括号或者`a`或者	`b`”。
* `[\\\[\]]`表示“匹配一个反斜杆或者一个左方括号或者一个右方括号”。（呕！）

在字符类中顺序和重复字符并不重要。`[dabaaabcc]`跟`[abcd]`一样。

<div class="warning">
<h4>重要的提示</h4>
<p>在字符类内部的“规则”和在字符类内部的规则有所不同。一些字符在字符类内部扮演着元字符的角色，但在字符类外部则充当字面值。还有一些字符做着相反的事。一些字符在<em>两种情形</em>都为元字符，但在各自情形里代表不同的含义。
</p>
<p>特别地, <code>.</code>表示“匹配任意字符”，但是<code>[.]</code>表示“匹配句点”。不能并为一谈。</p>
</div>

<div class="exercise">
<h4>练习</h4>
<p>结合目前所学，在字典中，使用正则表达式查找有连续的元音和连续的辅音的单词。</p>
<h4><button onclick="toggle('a3');">答案</button></h4>
<p id="a3" style="display: none;"><code>[aeiou][aeiou][aeiou][aeiou][aeiou][aeiou]</code>匹配到六元音单词<code>euouae</code> and <code>euouaes</code>，而可怕的<code>[bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz]</code>找到了有十个辅音的华丽的<code>sulphhydryls</code>。我们将会很快看到如何简化这些恐怖的表达式。</p>
</div>

###字符类区间（ranges）

你可以在字符类中使用连字符来表示一个字母或数字的*区间*：

* `[b-f]`和`[bcdef]`都表示“找到一个`b`或`c`或`d`或	`e`或`f`”。
* `[A-Z]`和`[ABCDEFGHIJKLMNOPQRSTUVWXYZ]`都表示“匹配大写字母”。
* `[1-9]`和`[123456789]`都表示“匹配一个非零数字”。

连字符在字符类外部使用时并没有特别都含义。正则表达式`a-z`表示“找到一个`a`接着跟着一个连字符，然后匹配一个`z`”。

区间和单独的字符可能会共存于同一个字符类：

* `[0-9.,]`表示“匹配一个数字或者一个句点或者一个逗号”。
* `[0-9a-fA-F]`表示“匹配一位十六进制数”。
* `[a-zA-Z0-9\-]`表示“匹配一个字母数字字符或连字符”。

虽然你可以*尝试*在区间内以非字母数字字符结束（比如`abc[!-/]def`），但这在其它实现中的语法不一定对。即使语法正确，但在这个区间内很难看出包含了哪个字符。请谨慎使用（我的意思是不要这么干）。

同样的，区间端点的范围应该一致。即使像`[A-z]`这种表达式在你选择的实现中合法，但结果可能不如你愿。（补充：可以有`Z`到`a`的区间范围）。

<div class="warning">
<p><strong>注意。</strong> 区间是字符的区间，不是数字的区间。正则表达式<code>[1-31]</code>表示“找到一个<code>1</code>或一个 <code>2</code>或一个<code>3</code>”，<em>不是</em>“找到一个从<code>1</code>到<code>31</code>的整数"。</p>
</div>

<div class="exercise">
<h4>练习</h4>
<p>使用目前学习，编写一个查找以YYYY-MM-DD为格式的日期的正则表达式。</p>
<h4><button onclick="toggle('a4');">答案</button></h4>
<p id="a4" style="display: none;">目前我们能写出来的是<code>[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]</code>。同样地，我们将能够很快简化这个式子。</p>
</div>

###字符类的否定（negation）

你可以通过在最开始的位置使用插入符号（译者注：`^`）来*否定*一个字符类。

* `[^a]`表示“匹配除了`a`的任意字符”。
* `[^a-zA-Z0-9]`表示“找到一个非字母也非数字的字符”。
* `[\^abc]`表示“找到一个插入符或者`a`或者`b`或者`c`”。
* `[^\^]`表示“找到除了插入符外的任意字符”。（呕！）  

<div class="exercise">
<h4>练习</h4>
<p>在字典中，使用正则表达式去找到这个规则的反例“<code>i</code>位于<code>e</code> 前面并且不出现在<code>c</code>的后面”。</p>
<h4><button onclick="toggle('a5');">答案</button></h4>
<p id="a5" style="display: none;"><code>cie</code>和<code>[^c]ei</code>都会找到很多的反例，比如<code>ancient</code>，<code>science</code>，<code>veil</code>，<code>weigh</code>。</p>
</div>

###字符类补充

正则表达式`\d`含义与`[0-9]`一致：“匹配一个数字”。（为了匹配一个反斜杆后跟一个`d`，可以使用`\\d`。）

`\w`的含义与`[0-9A-Za-z_]`一致：“匹配一个单词字符（译者注：字母或数字或下划线或汉字）”。

`\s`表示“匹配任意空白字符（空格，tab，回车或者换行）”。

此外，

* `\D`同`[^0-9]`：“匹配任意非数字的字符”。
* `\W`同`[^0-9A-Za-z_]`：“匹配任意非单词字符（译者注：匹配任意不是字母，数字，下划线，汉字的字符）”。
* `\S`表示“匹配任意不是空白符的字符”。

这些字符类都很常见，你必须学会。

你可能也注意到了，句点`.`本质上是*一个包含任意字符的字符类*。

许多实现提供了很多额外的字符类或标记，它们通过扩展现有的字符类来覆盖ASCII之外范围的字符。提示：Unicode包含更多的“数字字符”而不仅仅是`0`到`9`，这一点同样对于“单词”和“空格”也适用。注意你的文档所写。

<div class="exercise">
<h4>练习</h4>
<p>简化正则表达式<code>[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]</code>。</p>
<h4><button onclick="toggle('a6');">答案</button></h4>
<p id="a6" style="display: none;"><code>\d\d\d\d-\d\d-\d\d</code>.</p>
</div>

###乘法器（Multipliers）

你可以在一个字面值或者字符类后跟着一个大括号来使用*乘法器*。

* 正则表达式`a{1}`同`a`，表示“匹配一个`a`”。
* `a{3}`表示“找到一个`a`后再跟一个`a`，最后找到一个`a`”。
* `a{0}`表示“匹配空字符”。就其本身而言，这似乎没有用处。如果你在任何一段文本中使用该表达式，你会在你刚开始搜索的端点处立即得到一个匹配。即使你的文本为空字符串结果也为真。
* `a\{2\}`代表“找到一个`a`，跟着一个左大括号，接着跟匹配一个`2`，然后跟着一个右大括号”。
* 在字符类中大括号没有特别的含义。`[{}]`代表“匹配一个左大括号或者一个右大括号”。

<div class="warning">
<p><strong>注意。</strong> 乘法器没有记忆。该正则表达式<code>[abc]{2}</code>表示“匹配<code>a</code>或者<code>b</code>或者<code>c</code>，接着匹配<code>a</code>或者<code>b</code>或者<code>c</code>。这跟“匹配<code>aa</code>或<code>ab</code>或<code>ac</code>或<code>ba</code>或<code>bb</code>或<code>bc</code>或<code>ca</code>或<code>cb</code>或<code>cc</code>”相同。这跟“匹配<code>aa</code>或<code>bb</code>或<code>cc</code>”含义<em>不同</em>！
</div>

<div class="exercise">
<h4>练习</h4>
<p>简化以下正则表达式：</p>
<ul>
	<li><code>z.......z</code></li>
	<li><code>\d\d\d\d-\d\d-\d\d</code></li>
	<li><code>[aeiou][aeiou][aeiou][aeiou][aeiou][aeiou]</code></li>
	<li><code>[bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz][bcdfghjklmnpqrstvwxyz]</code></li>
</ul>
<h4><button onclick="toggle('a6A');">答案</button></h4>
<ul id="a6A" style="display: none;">
	<li><code>z.{7}z</code></li>
	<li><code>\d{4}-\d{2}-\d{2}</code></li>
	<li><code>[aeiou]{6}</code></li>
	<li><code>[bcdfghjklmnpqrstvwxyz]{10}</code></li>
</ul>
</div>

###乘法器区间

乘法器可能会有*区间*：

* `x{4,4}`跟`x{4}`一样。
* `colou{0,1}r`表示“匹配`colour`或`color`。
* `a{3,5}`表示“匹配`aaaaa`或`aaaa`或`aaa`”。

值得注意的是优先选择更长的匹配，因为乘法器是*贪婪*的。如果你输入的文本是`I had an aaaaawful day`，该正则表达式就会在`aaaaawful`中匹配到`aaaaa`。不会在第三个`a`后就停止匹配。

乘法器是贪婪的，但它不会忽略一个更好的匹配。如果你的输入文本为`I had an aaawful daaaaay`，之后这个正则表达式会在第一次的匹配中于`aaawful`找到`aaa`。只有在你说“给我找到另一个匹配”的时候，它才会继续搜索然后在`daaaaay`中找到`aaaaa`。

乘法器区间可能是开区间：

* `a{1,}`表示“在一列中找到一个或多个a”。然而你的乘法器将会是贪婪的。在找到第一个`a`后，它将会尽可能匹配到更多的`a`。
* `.{0,}`表示“匹配任何情形”。不管你的输入文本是什么——甚至为空——这个正则表达式都会匹配整个字符串然后返回给你。    

<div class="exercise">
<h4>练习</h4>
<p>编写一个能匹配双引号字符串的正则表达式。同时该字符串可以拥有任意数量的字符。</p>
<p>用你已经学到的之时，修改上面的正则表达式，来找到了双引号字符串，但它们之间没有多余的双引号。</p>
<h4><button onclick="toggle('a7');">答案</button></h4>
<p id="a7" style="display: none;"><code>".{0,}"</code>，然后是<code>"[^"]{0,}"</code>。</p>
</div>

###乘法器补充

`?`代表的含义与`{0,1}`相同。比如说，`colour?r`表示“匹配`colour`或`color`”。

`*`等于`{0,}`。比如说，`.*`表示“匹配一切”，跟上面提到的一样。

`+`等于`{1,}`。比如说，`\w+`表示“匹配一个单词”。这里的“单词”是1个或多个“单词字符”的序列，就像`_var`或`AccountName1`。

这些乘法器都很常见，你必须掌握。还有：

* `\?\*\+`表示“匹配一个问号，接着找到一个星号，然后跟着一个加号”。
* `[?*+]`表示“找到一个问号或者一个星号或者一个加号”。  

<div class="exercise">
<h4>练习</h4>
<p>简化下面的正则表达式：</p>
<ul>
	<li><code>".{0,}"</code>和<code>"[^"]{0,}"</code></li>
	<li><code>x?x?x?</code></li>
	<li><code>y*y*</code></li>
	<li><code>z+z+z+z+</code></li>
</ul>
<h4><button onclick="toggle('a8');">答案</button></h4>
<ul id="a8" style="display: none;">
	<li><code>".*"</code>和<code>"[^"]*"</code></li>
	<li><code>x{0,3}</code></li>
	<li><code>y*</code></li>
	<li><code>z{4,}</code></li>
</ul>
</div>

<div class="exercise">
<h4>练习</h4>
<p>编写一个表达式来查找非单词字符分隔的两个单词。如果改为三个单词或者六个单词又该怎么写？</p>
<h4><button onclick="toggle('a9');">答案</button></h4>
<p id="a9" style="display: none;"><code>\w+\W+\w+</code>，<code>\w+\W+\w+\W+\w+</code>，<code>\w+\W+\w+\W+\w+\W+\w+\W+\w+\W+\w+</code>。当然，我们之后会学习如何简化它们。</p>
</div>

###惰性（Non-greed）

正则表达式`".*"`表示“找到一个双引号，接着找到尽可能多的字符，最后再找到一个双引号”。注意一下被`.*`匹配的内部字符，很可能包含多个双引号。这通常不是非常有用。

乘法器可通过追加问号来实现*惰性*。这里对优先顺序进行了反转：

* `\d{4,5}?`表示“匹配`\d\d\d\d`或`\d\d\d\d\d`”。其实跟`\d{4}`行为一致。
* `colou??r`就是`colou{0,1}?r`，表示“找到`color`或`colour`”。和`colou?r`行为一致。
* `".*?"`表示“匹配一个双引号，跟着一个*尽可能少*的字符，再跟着一个双引号”。这个不像上面两个例子，实际上很有用。  

###分支（Alternation）

你可以使用管道符号来实现匹配多种选择：

* `cat|dog`表示“匹配`cat`或`dog`”。
* `red|blue|`和`red||blue`以及`|red|blue`都是同样的意思，“匹配`red`或`blue`或空字符串”。
* `a|b|c`跟`[abc]`一样。
* `cat|dog|\|`表示“匹配`cat`或`dog`或`管道符号`”。
* `[cat|dog]`表示“找到`a`或`c`或`d`或`d`或`g`或`o`或`t`或一个管道符号”。  

<div class="exercise">
<h4>练习</h4>
<p>尽你所能简化下述正则表达式：</p>
<ul>
	<li><code>s|t|u|v|w</code></li>
	<li><code>aa|ab|ba|bb</code></li>
	<li><code>[abc]|[^abc]</code></li>
	<li><code>[^ab]|[^bc]</code></li>
	<li><code>[ab][ab][ab]?[ab]?</code></li>
</ul>
<h4><button onclick="toggle('a10');">答案</button></h4>
<ul id="a10" style="display: none;">
	<li><code>[s-w]</code></li>
	<li><code>[ab]{2}</code></li>
	<li><code>.</code></li>
	<li><code>[^b]</code></li>
	<li><code>[ab]{2,4}</code></li>
</ul>
</div>

<div class="exercise">
<h4>练习</h4>
<p>编写一个正则表达式匹配1到31（含）之间的整数。 记住，<code>[1-31]</code>不是正确答案。</p>
<h4><button onclick="toggle('a11');">答案</button></h4>
<p id="a11" style="display: none;">有几种方法都可以做到这一点。我认为其中<code>[1-9]|[12][0-9]|3[01]</code>可读性最好。</p>
</div>


###组合（Grouping）

你可以使用圆括号来*组合*表达式：

* 在一周中找到一天，使用`(Mon|Tues|Wednes|Thurs|Fri|Satur|Sun)day`。
* `(\w*)ility`等同于`\w*ility`。都表示“找到以`ility`结尾的单词”。为什么第一种形式更有用，后面会看到...
* `\(\)`表示“匹配一个左圆括号后，再匹配一个右圆括号”。
* `[()]`表示“匹配一个左圆括号或一个右圆括号”。

<div class="exercise">
<h4>练习</h4>
<p>在<i>《时光机器》</i>这本书中，使用正则表达式来查找包裹在括号中的句子。接着，修改你的答案来查找没有被括号包裹的句子。</p>
<h4><button onclick="toggle('a12');">答案</button></h4>
<p id="a12" style="display: none;"><code>\(.*\)</code>，然后是<code>\([^()]*\)</code>。</p>
</div>


组合可能会包含空字符串：

* `(red|blue|)`表示“匹配`red`或`blue`或`空字符串`”。
* `abc()def`等同于`abcdef`

可能你会在组合中使用乘法器：

* `(red|blue)?`等同于`(red|blue|)`。
* `\w+(\s+\w+)*`代表“找到一个或多个单词，它们以空格隔开”。  

<div class="exercise">
<h4>练习</h4>
<p>简化<code>\w+\W+\w+\W+\w+</code>和<code>\w+\W+\w+\W+\w+\W+\w+\W+\w+\W+\w+</code>。</p>
<h4><button onclick="toggle('a13');">答案</button></h4>
<p id="a13" style="display: none;"><code>\w+(\W+\w+){2}</code>，<code>\w+(\W+\w+){5}</code>。</p>
</div>

###单词边界（Word boundaries）

单词边界是一个单词字符和非单词字符之间的位置。记住，一个单词字符是`\w`，它是`[0-9A-Za-z_]`，一个非单词字符是`\W`，也就是`[^0-9A-Za-z_]`。

文本的开头和结尾总是当作单词边界。

输入的文本`it's a cat`有八个单词边界。如果我们在`cat`后追加一个空格，这里就会有九个单词边界。

* 正则表达式`\b`表示“匹配一个单词边界”。
* `\b\w\w\w\b`表示“匹配一个三个字母的单词”。
* `a\ba`表示“找到`a`，跟着一个单词边界，接着找到`b`”。不管输入文本是什么，这个正则表达式永远都不会成功找到一个匹配。

单词边界不是字符。它们宽度为零.下面的正则表达式表示相同的含义：

* `(\bcat)\b`
* `(\bcat\b)`
* `\b(cat)\b`
* `\b(cat\b)`  

<div class="exercise">
<h4>练习</h4>
<p>查找字典中最长的单词。</p>
<h4><button onclick="toggle('a14');">答案</button></h4>
<p id="a14" style="display: none;">在一些试验和错误之后，这个正则表达式就是<code>\b.{45,}\b</code>，在字典中找到唯一一个结果： <code>pneumonoultramicroscopicsilicovolcanoconiosis</code>。</p>
</div>


###行边界（Line boundaries）

每一块文本会分解成一个或多个行，用换行符分隔，像这样：

* 行
* 换行
* 行
* 换行
* ...
* 换行
* 行

注意文本不是以换行符结束，而是以行结束。然而，任何行，包括最后一行，可以包含零个字符。

*起始行*位置是在一个换行符和下一行的第一个字符之间。与单词边界一样，在文本的开头也算作一个起始的行。

*结束行*位置是在行的最后一个字符和换行符之间。与单词边界一样，文本结束也算作行结束。

所以我们都细分为：

* 起始行，行，结束行
* 换行
* 开始行，行，结束行
* 换行
* ...
* 换行
* 开始行，行，结束行

在此基础上，有：

* 正则表达式`^`表示“匹配开始行”。
* 正则表达式`$`表示“匹配结束行”。
* `^$`表示“匹配空行”。
* `^.*$`将会匹配整个文本，因为换行符是一个字符，所以`.`会匹配它。为了匹配单行，要使用惰性乘法器，`^.*?$`。
* `\^\$`表示“匹配尖符号后跟着一个美元符号”。
* `[$]`表示“匹配一个美元符”。*然而*，`[^]`是非法单正则表达式。要记住的是尖符号在方括号中时有*不同*的特殊含义。把尖符号放在字符类中，这么用`[\^]`。

像单词边界一样，行边界也不是字符。它们宽度为零。下面的正则表达式表示相同的含义：

* `(^cat)$`
* `(^cat$)`
* `^(cat)$`
* `^(cat$)`  

<div class="exercise">
<h4>练习</h4>
<p>适用正则表达式查找<i>《时光机器》</i>中最长的一行。</p>
<h4><button onclick="toggle('a15');">答案</button></h4>
<p id="a15" style="display: none;">Gutenberg出版社的这个版本一行最多有73个字符，使用该<code>^.{73,}$</code>表达式。许多行都是这个长度。</p>
</div>

###文本边界（Text boundaries）

很多实现提供一个标记，通过改变它来改变`^`和`$`的含义。从“行开始”和“行结束”变成“文本开始”和“文本结束”。

其它的一些实现提供单独的元字符`\A`和`\z`来达到这个目的。

##捕获和替换

这里就是正则表达式开始变得异常强大的地方。

###捕获组

你已经知道，括号是用来表示组。它们也可以用来捕获子串。如果正则表达式是一个很小的电脑程序，这个*捕获组*就是它的输出（的一部分）。

正则表达式`(\w*)ility`表示“找到一个以`ility`结束的单词”。捕获组1就是匹配了部分内容的`\w*`。举个例子，如果我们的文本包含单词`accessibility`，捕获组1就是`accessib`。如果我们的文本自身只包含`ility`，捕获组1就是空字符串。

你可以拥有多个捕获组，它们甚至可以嵌套使用。捕获组从左到右进行编号。只要计算左圆括号。

假设我们到正则表达式是`(\w+) had a ((\w+) \w+)`。如果我们的输入文本是`I had a nice day`，那么

* 捕获组1是`I`。
* 捕获组2是`nice day`。
* 捕获组3是`nice`。

在一些实现中，你可能可以访问捕获组0，即完整匹配：`I had a nice day`。

是的，这确实意味着圆括号有些重复。一些实现就提供了一个独立语法来声明“非捕获组”，但是这个语法不符合标准，所以这里我们不涉及。

<div class="warning">
<p>从一个成功返回的匹配中捕获组数量<em>总是</em>等于原来正则表达式中捕获组的数量。记住这一点，因为它可以帮助你理解一些令人困惑的情形。</p>
<p>正则表达式<code>((cat)|dog)</code>表示“匹配<code>cat</code>或<code>dog</code>”。这里<strong>总是存在两组捕获组</strong>。如果我们的输入文本是<code>dog</code>，那么捕获组1是<code>dog</code>，捕获组2是空字符串，因为另一个选择未被使用。</p>
<p>正则表达式<code>a(\w)*</code>表示“匹配一个以<code>a</code>开头的单词”。这里<strong>总是只有一个捕获组（译者注：除去捕获组0）</strong>：</p>
<ul>
	<li>如果输入文本是<code>a</code>，捕获组1是空字符串。</li>
	<li>如果输入文本是<code>ad</code>，捕获组1是<code>d</code>。</li>
	<li>如果输入文本是<code>avocado</code>，捕获组1是<code>v</code>。然而，捕获组0会是整个单词，<code>avocado</code>。</li>
</ul>
</div>


###替换

一旦你用了正则表达式来查找字符串，你可以指定另一个字符串来替换它。第二个字符串时*替换表达式*。首先，就像：

* 传统的替换对话框
* Java的`String.replace()`函数
* PHP的`String.replace()`函数
* 等等

<div class="exercise">
<h4>练习</h4>
<p>使用<code>r</code>替换<i>《时间机器》</i>中所有的元音字母。确保使用正确的大小写！</p>
<h4><button onclick="toggle('a16');">答案</button></h4>
<p id="a16" style="display: none;">分别使用正则表达式<code>[aeiou]</code>和<code>[AEIOU]</code>，替换表达式<code>r</code>和<code>R</code>。</p>
</div>

然而，你可以在你的替换表达式中引用捕获组。这是你可以在替换表达式唯一能的特殊的事，它是令人难以置信的强大，因为它意味着你不必完全销毁你刚刚发现的东西。

比方说，你尝试去用ISO 8691格式的日期（YYYY-MM-DD）去替换美式日期（MM/DD/YY）。

* 通过正则表达式`(\d\d)/(\d\d)/(\d\d)`开始。注意这里有三个捕获组：月，日和两个数字表示的年。
* 通过使用一个反斜杆和一个捕获组号来引用一个捕获组。所以，你的替换表达式为`20\3-\1-\2`。
* 如果我们的输入文本是`03/04/05`（表示 3月4号，2005年），那么
	* 捕获组1是`03`
	* 捕获组2是`04`
	* 捕获组3是`05`
	* 替换字符串为`2005-03-04`

你可以在替换表达式中多次引用捕获组。

* 使用正则表达式`([aeiou])`和替换表达式`\1\1`来让元音翻倍。

在替换表达式中的反斜杆必须进行转义。举个例子，你有一些在计算机程序的字面值中使用的文本。那就意味着你需要在普通文本中的每个双引号或者反斜杆前放置一个反斜杆。

* 正则表达式`([\\"])`中，捕获组1是双引号或者反斜杆。
* 替换表达式`\\\1`中，一个字面值反斜杆后跟着一个匹配的双引号或者反斜杆。  


###后向引用（Back-references）

你可以在同样的表达式中引用同一个捕获组。这称为*后向引用*。

举个例子，再次调用前面的表达式`[abc]{2}`表示“匹配`aa`或`ab`或`ac` or `ba`或`bb`或`bc`或`ca`或`cb`或`cc`”。但是表达式`([abc])\1`表示“匹配`aa`或`bb`或`cc`”。

<div class="exercise">
<h4>练习</h4>
<p>在字典中，找到出现两次相同字符串的最长的单词（比如<code>papa</code>， <code>coco</code>）。</p>
<h4><button onclick="toggle('a17');">答案</button></h4>
<p id="a17" style="display: none;"><code>\b(.{6,})\1\b</code>匹配到<code>chiquichiqui</code>。如果我们不关心完整的单词，我们可以舍去单词边界断言，使用<code>(.{7,})\1</code>会找到<code>countercountermeasure</code>和<code>countercountermeasures</code>。</p>
</div>

##结合正则表达式编程

一些具体的注意事项：

###过度反斜线综合征（Excessive backslash syndrome）

在一些编程语言中，如Java，对于含有正则表达式的字符串没有提供特别的支持。字符串有自己的转义规则，这些规则与正则表达式的转义规则叠加，通常会导致反斜杆过多（overload）。比如（还是Java）：

* 为了匹配一个数字，正则表达式`\d`在源代码中变成`String re = "\\d;"`。
* 为了匹配一个双引号字符串，`"[^"]*"`变成`String re = "\"[^\"]*\"";`。
* 为了匹配一个反斜杆或者一个左方括号或者一个又方括号，正则表达式`[\\\[\]]`变成`String re = "[\\\\\\[\\]]";`。
* `String re = "\\s";`和`String re = "[ \t\r\n]";`是一样的。注意不同的转义“优先级”。

在其它编程语言里，通过一个特殊标记来标识正则表达式，通常是正斜杆`/`。这里有一些JavaScript例子：

* 为了匹配一个数字，`\d`变成`var regExp = /\d/;`。
* 匹配一个反斜杆或者一个左方括号或者一个右方括号，`var regExp = /[\\\[\]]/;`。
* `var regExp = /\s/;`和`var regExp = /[ \t\r\n]/;`一样。
* 当然，这意味着必须对正斜杠而不是双引号进行转义。匹配URL的前面部分：`var regExp = /https?:\/\//;`。

基于这一点，我希望你明白为什么我对你反复提及反斜杆。

###偏移量（Offsets）

在文本编辑器中，会在你光标所在处开始搜索。这个编辑器会向前开始搜索文字，然后停在第一个匹配的地方。下一次搜索会在第一次完成搜索的地方的右侧开始。

当编程的时候，文本的*偏移量*是必须的。这个偏移量会在代码中有明确的支持，或保存在包含文本的对象中（如Perl），或包含正则表达式的对象中（如JavaScirpt）。（在Java里，这是一个由正则表达式和复合对象的字符串。）在任何情况下，默认值为0，表示文本的开始。搜索后，偏移量会自动更新，或者作为输出的一部分返回。

无论什么情况，通常很容易去使用循环来解决这个问题。

<div class="warning">
<p><strong>注意。</strong>正则表达式匹配空字符串是完全可能的。 你可以立马实现的一个简单的例子是<code>a{0}</code>在这种情况下，新的偏移量等于旧偏移量，从而导致死循环。</p>
<p>一些实现可能保护你避免发生这些情况，但要查下对应的文档。</p>
</div>

###动态正则表达式

动态地构造一个正则表达式字符串时一定要小心。如果你使用的字符串不是固定的,那么它可能包含意想不到的元字符。这会导致语法错误。更糟糕的是，它可能产生一个语法正确，但行为不可预期的正则表达式。

有bug的Java代码：
	
	String sep = System.getProperty("file.separator");
	String[] directories = filePath.split(sep);

这个bug就是：`String.split()`认为`sep`是一个正则表达式。但是在Windows下，`sep`是由犯斜杆组成的字符串`"\\"`.这不是一个语法正确的正则表达式。结果是：一个异常`PatternSyntaxException`。

任何一个优秀的编程语言都提供了一种机制，用以转义在一个字符串中出现的所有元字符。在Java中，你可以这么做：

	String sep = System.getProperty("file.separator");
	String[] directories = filePath.split(Pattern.quote(sep));

###循环内的正则表达式

把正则表达式字符串编译进一个正在运行的“程序”中是一个代价昂贵的操作。如果你能避免在循环内这么做的话能提高程序性能。

##各类建议

###输入验证

正则表达式能用于用户输入验证。但过于严格的验证会让用户感到难受。下面举几个例子：

**支付卡号**

我在网页上输入我的卡号如`1234 5678 8765 4321`。会被这个站点拒绝。因为它使用`\d{16}`来进行验证。

该正则表达式允许出现空格和连字符。

其实，为什么不直接去掉所有非数字字符，然后再进行验证？要做到这一点，使用正则表达式`\D`和空字符串来替换表达式。

<div class="exercise">
<h4>练习</h4>
<p>编写一个正则表达式，可以验证我的卡号而不用让我删去非数字字符。</p>
<h4><button onclick="toggle('a18');">答案</button></h4>
<p id="a18" style="display: none;"><code>\D*(\d\D*){16}</code>是能多种实现中的一个方式。</p>
</div>

**名字**

不要使用正则表达式来验证用户的名字。其实，不需要验证名字，你无能无力。

[Falsehoods programmers believe about names](http://www.kalzumeus.com/2010/06/17/falsehoods-programmers-believe-about-names/)提到了：

* 名字不能包含空格。
* 名字不能包含标点符号。
* 名字只能使用ASCII字符。
* 名字会被限制在*任何*特定的字符集。
* 名字总是有像*M*字符那么长。
* 人总是有且只有一个用的名字。
* 人总是有且仅有一个中间名。
* 人总是有且只有一个姓。
* ...

**邮件地址**

不要使用正则表达式来验证邮件地址。

首先，这很难保证正确无误。电子邮件地址确实符合一个正则表达式，但是这个表达式[长又复杂地让人联想到世界末日](http://www.ex-parrot.com/pdw/Mail-RFC822-Address.html)。任何缩略都会可能产生遗漏（false negatives）。（你知道吗？电子邮件地址可以包含注释！）

其次，即使所提供的电子邮件地址符合正则表达式，但也并不能证明它的存在。**验证电子邮件地址的唯一方法是发送电子邮件给它。**

###标记

在正式的应用中，不要使用正则表达式来解析HTML或XML。解析HTML/XML是

1. 不可能使用简单的正则
2. 一般来说很难
3. 一个已解决了的问题。

不妨找一个已有的解析库来为你搞定这些工作。


##这就是55分钟内容

总结：

* 字面值：`a b c d 1 2 3 4`等等。
* 字符类：`. [abc] [a-z] \d \w \s`
	* `.`表示“任何字符”
	* `\d`表示“一个数字”
	* `\w`表示“一个单词字符”，`[0-9A-Za-z_]`
	* `\s`表示“一个空格，tab，回车或一个换行符”
	* 否定字符类：`[^abc] \D \W \S`
* 乘法器：`{4} {3,16} {1,} ? * +`
	* `?`表示“没有或一个”
	* `*`表示“没有或多个”
	* `+`表示“一个或多个”
	* 乘法器是贪婪的除非你在之后使用`?` 
* 分支和组合：`(Septem|Octo|Novem|Decem)ber`
* 词、行和文本边界：`\b ^ $ \A \z`
* 反向捕获组：`\1 \2 \3`等等。（在替换表达式和匹配表达式中同时生效）
* 元字符列表：`. \ [ ] { } ? * + | ( ) ^ $`
* 字符类中使用到元字符列表：`[ ] \ - ^`
* 你总是可以使用反斜杆对元字符进行转义：`\`  

###感谢阅读
正则表达式无处不在，令人难以置信的有用。那些在编辑文本和写电脑程序方面将花费大量时间的人们应该学会如何使用它们。
到目前为止，我们只接触了冰山一角。

<div class="exercise">
<h4>练习</h4>
<p>继续阅读你选择的正则表达式实现的对应文档。我保证在我们这里所讨论的部分之外还有更多的特性并未涉及。</p>
</div>
