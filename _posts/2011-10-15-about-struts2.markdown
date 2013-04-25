---
author: labuxiaojue
comments: true
date: 2011-10-15 21:45:12
layout: post
slug: about-struts2
title: 关于Struts2.x的几个注意点
wordpress_id: 231
categories:
- Java
tags:
- Java
- Struts2
---

初学Struts2，整理一下本周遇到的关于Struts2的一些注意点。
**1.标签中引号的问题**
首先对于HTML语言中的标签使用来说，以链接元素举例，现在要为该元素标签的属性href指定值，我们通常以key="value"，即href="someurl"的形式。
如果现在要求该url地址的值是非静态的，它的值保存在于服务器端的某个Aciton类中的对象sample的属性url中，我们就需要用到Struts2自带标签类来对此取值，譬如以下述两种方式：

    
    
    <a href="<s:property value='sample.url'>"></a>
    <a href="<s:property value="sample.url">"></a>
    


因为出现了两种标签——Struts标签与HTML标签，导致了属性区分的包含问题。这就需要借助于单引号与双引号的分隔：如果外围先出现了单引号，那么包含在其中的另一个标签就需要使用双引号，反之亦然。于是乎就有了上述两种形式。当然，这两种形式都无错误之处。不过，如果出现了下面这个情形，上述的表示方式就会有一定问题： 假设我们这边服务端要从数据库中取某个字符，然后在浏览器这边显示，比如数据库中存放"M"与“F”来表示性别。那么为了友好性，显示给用户的需要把相应字符串替换为“男”与“女”。 JSP页面代码部分截取如下：
<!-- more -->

    
    
    <s:if test="emp.gender.equals('M')">男</s:if>
    <s:else>女</s:else>
    


现在问题来了，我们会惊奇的发现不管对象emp的gender值是什么，页面中显示的都为“女“。而直接输出emp.sex属性值是正常的。
这是为什么呢？。排查了许久，终于发现是引号的问题。
虽然第一个例子中引号是有区分元素的作用，但第二个例子中引号还有起到标识引号内元素为字符串的作用。
如果我们以单引号引用M/F的话，就表示该符号是一个字符，我们以字符串的equal方法比较它的话自然永远不能让它们直接划上等号。因为Struts2标签中的属性取值的实现是由OGNL实现的，而后者的底层实现又是由Java的方法组成。既然Java代码中同样字母的字符与字符串之间不相等，那么在Struts标签类中也不可能相等。
测试代码如下：

    
    package me.ijava.test;
    public class CompareCharToString {
        public static void main(String[] args){
            String str = "a";
            char c = 'a';		
            System.out.println(str.equals(c)); //false
        }
    }


因为equal()方法比较的是对象，所以字符'a'自动装箱为Character类型，两者对象比较输出结果为false。
另外插一点题外话：如果用JavaScript的函数比较的话，它是否也对引号问题敏感呢？
测试代码如下：

    
    
    function compareTest(){
    		var str = "a";
    		var c = 'a';
    		alert(str==c);
    	}
    	compareTest();
    


有意思的是：结果为true！这是由于弱类型的JS不区分字符串与字符。
**2.EL表达式的取值问题**
在一个应用了Struts2系列框架的项目中，在JSP页面常常要用Struts附带的标签类对的一些表单元素的取值(框架标签库引入方式)。这些标签很容易让人冷落原Servlet容器中的标签与EL表达式。那么如果使用EL表达式，能否对服务器中的bean类们顺利取值呢？
因为Struts2把容器中原有的HttpServletRequest对象封装成了StrutsRequestWrapper。我们先去瞅瞅Struts2的关于StrutsRequestWrapper类的部分源码。

    
    	public class StrutsRequestWrapper extends HttpServletRequestWrapper {
        /**
         * The constructor
         * @param req The request
         */
        public StrutsRequestWrapper(HttpServletRequest req) {
            super(req);
        }
        /**
         * Gets the object, looking in the value stack if not found
         *
         * @param s The attribute key
         */
        public Object getAttribute(String s) {
            if (s != null && s.startsWith("javax.servlet")) {
                // don't bother with the standard javax.servlet attributes, we can short-circuit this
                // see WW-953 and the forums post linked in that issue for more info
                return super.getAttribute(s);
            }
            ActionContext ctx = ActionContext.getContext();
            Object attribute = super.getAttribute(s);
            if (ctx != null) {
                if (attribute == null) {
                    boolean alreadyIn = false;
                    Boolean b = (Boolean) ctx.get("__requestWrapper.getAttribute");
                    if (b != null) {
                        alreadyIn = b.booleanValue();
                    }
                    // note: we don't let # come through or else a request for
                    // #attr.foo or #request.foo could cause an endless loop
                    if (!alreadyIn && s.indexOf("#") == -1) {
                        try {
                            // If not found, then try the ValueStack
                            ctx.put("__requestWrapper.getAttribute", Boolean.TRUE);
                            ValueStack stack = ctx.getValueStack();
                            if (stack != null) {
                                attribute = stack.findValue(s);
                            }
                        } finally {
                            ctx.put("__requestWrapper.getAttribute", Boolean.FALSE);
                        }
                    }
                }
            }
            return attribute;
        }
    }


我们可以发现，在该类重写的的getAttribute()方法中，首先调用了父类的（即HttpServletRequestRequestWrapper）的getAttribute()方法。之后如果属性值没有得到的话，就再去根属性存储区——值栈ValueStack中获取。
因为EL的取值是通过调用request的getAttribute()方法的，故虽然进行了封装后重写但在前面调用了父类的getAttribute()方法，这一点说明EL表达式可以正常使用，另一方面也带来了一个问题：
在某个action中，有这么一段代码:

    
    public String execute(){
        value="Action Value";
            return "success";
        }
        private String value;
            public String getValue(){
                return value;	
        }
        public void setValue(String value){
            this.value = value;
        }


假设我们配置好了Struts.xml，其中接收到action请求即会调用该execute方法，并且把value属性赋值成"Action Value"，然后在JSP页面中显示其值。取值的方式有两种，
`1)Struts标签：<s:property value="value">
2)EL表达式：${value}` 都会在页面成功显示出“Action Value”。
现在我们改动一点Action的代码：

    
    public String execute(){
        value="Action Value";
        Map request = (Map) ActionContext.getContext().get("request");
        request.put("value", "Request Value");
        return "success";
    }


JSP部分的代码不变，重新部署，发送请求我们又惊奇地发现，页面输出内容为：
`Action Value
Request Value`
我们再来看标签类的实现代码：

    
    public class PropertyTag extends ComponentTagSupport {
        private static final long serialVersionUID = 435308349113743852L;
        private String defaultValue;
        private String value;
        private boolean escape = true;
        private boolean escapeJavaScript = false;
        public Component getBean(ValueStack stack, HttpServletRequest req, HttpServletResponse res) {
            return new Property(stack);
        }
        protected void populateParams() {
            super.populateParams();
            Property tag = (Property) component;
            tag.setDefault(defaultValue);
            tag.setValue(value);
            tag.setEscape(escape);
            tag.setEscapeJavaScript(escapeJavaScript);
        }
        public void setDefault(String defaultValue) {
            this.defaultValue = defaultValue;
        }
        public void setEscape(boolean escape) {
            this.escape = escape;
        }
        public void setEscapeJavaScript(boolean escapeJavaScript) {
            this.escapeJavaScript = escapeJavaScript;
        }
        public void setValue(String value) {
            this.value = value;
        }
    }


镇定下来后的我们可以很直接地找出原因：
由于Struts2中的标签类<s:property />的取值默认是通过OGNL表达式实现的，而后者如果直接以属性名的方式即<s:property value="value" />访问的是ValueStatck的信息，而加了"#"号以<s:property value="#value" />形式的表达式访问的是变量存储区context中的信息。无疑，上述例子自然取出ValueStack中的“Action Value”。
而EL表达式由于Struts2重写getAttribute()方法的缘故，会先于原先的request中取值，也就是“Request Value”如果取不到则使用ActionContext.getContext().getValueStack()的findValue()方法在ValueStack中查找。
