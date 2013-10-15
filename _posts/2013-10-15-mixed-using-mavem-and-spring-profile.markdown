---
date: 2013-10-15 11:05:55
title: Maven与Spring profile的结合使用
layout: post
tags:
    - Maven
    - Spring
    - Profile
categories:
    - Maven
---

需求描述
-------

在实际开发中，我们常常需要应对多类环境，针对不同的环境来更改相应的配置，比如常见开发环境、测试环境以及客户的实际部署环境。

因此我们会遇到下面的场景，在开发环境中数据源的获取方式是直连数据库，部署环境中需要连接的是JNDI，如何避免项目打包完每次人工更改配置文件的繁琐工作呢？

解决方案
-------
通过引入Spring和Maven的profile特性来实现不同环境自动切换不同配置的功能。

>###**声明Spring profile**

1、 定义两个beans，分别对应两个环境下的数据源配置：在Spring的配置文件`applicationContext.xml`中定义两个profile的beans。

    <beans profile="development">
			<bean id="dataSource"  class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName">
				<value>com.microsoft.sqlserver.jdbc.SQLServerDriver</value>
			</property>
			<property name="url">
				<value>jdbc:sqlserver://x.x.x.x:1433;databaseName=trunk</value>
			</property>
			<property name="username">
				<value>sa</value>
			</property>
			<property name="password">
				<value>123</value>
			</property>
			<property name="maxActive">
				<value>200</value>
			</property>
			<property name="maxIdle">
				<value>30</value>
			</property>
			<property name="maxWait">
				<value>600</value>
			</property>
		</bean>
	</beans>
	
	<beans profile="production">
		 <bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
	        <property name="jndiName">
	             <value>java:comp/env/jdbc/test</value>
	        </property>
    	</bean>		
	</beans>

>###**更改web.xml**

那么根据Spring的profile特性，我们只要在`web.xml`文件中定义如下形式配置：

    <context-param>
		<param-name>spring.profiles.active</param-name>
		<param-value>development</param-value>
	</context-param>

或

    <context-param>
		<param-name>spring.profiles.active</param-name>
		<param-value>production</param-value>
	</context-param>

就会启用相应的profile配置，Spring根据指定的配置来注入依赖。

>###**为spring.profiles.active赋值**

那么如何在打包时自动更改`spring.profiles.active`的值呢？这就需要Maven的profile特性。

我们将上述`spring.profiles.active`的param-value的值更改为{profiles.active}，这是Maven的属性值替换的占位符，Maven的资源过滤插件（Maven Resources plugin）将会在构建期间替换该值，为了在打包时启用资源过滤，需要我们配置Maven打包插件（Maven war plugin）：
    
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
		    <artifactId>maven-war-plugin</artifactId>
			<version>2.4</version>
			<configuration>
				<warName>${project.artifactId}</warName>
						
					<!-- 激活spring profile -->
					<webResources>
						<resource>
                            <filtering>true</filtering>
                            <directory>src/main/webapp</directory>
                            <includes>
                                <include>**/web.xml</include>
                            </includes>
							</resource>
					</webResources>
                    <warSourceDirectory>src/main/webapp</warSourceDirectory>
                    <webXml>src/main/webapp/WEB-INF/web.xml</webXml>
						
            </configuration>
    </plugin>

>###**启用Maven profile**

最后在项目的`pom.xml`或m2文件夹下的`setting.xml`中指定Maven的profile：

	<!-- 不同的打包环境 -->
	<profiles>
		<!-- 开发环境，默认激活 -->
		<profile>
			<id>development</id>
			<properties>
				<profiles.active>development</profiles.active>
			</properties>
			
		</profile>
		
		<!-- 部署环境 -->
		<profile>
			<id>production</id>
			<properties>
				<profiles.active>production</profiles.active>
			</properties>
			<activation>
				<!-- 默认启用的是dev环境配置 -->
				<activeByDefault>true</activeByDefault>
			</activation>
		</profile>
	</profiles>
	
根据上述配置，输入打包命令`mvn clean package`默认启用的是production，数据源由JNDI提供，如果需要激活另一个profile，只要更改打包命令为`mvn clean package -P development`。
