---
author: labuxiaojue
comments: true
date: 2011-04-15 13:19:41
layout: post
slug: define-sql-data
title: 数据定义
wordpress_id: 49
categories:
- SQL
tags:
- SQL
---

    /*例5 建立一个“学生”表 Student*/
    CREATE TABLE Student
    (Sno CHAR(9) PRIMARY KEY,   /*列级完整性约束条件，Sno是主码*/
    Sname CHAR(20) UNIQUE,       /*Sname取唯一值*/
    Ssex CHAR(2),
    Sage SMALLINT,
    Sdep CHAR(20)
    );

    /*例6 建立一个“课程表Course”*/
    CREATE TABLE Course
    (Cno CHAR(4) PRIMARY KEY,   /*列级完整性约束条件，Sno是主码*/
    Cname CHAR(40),
    Cpno CHAR(4),
    Ccredit SMALLINT,           /*Cpno的含义是先修课*/
    );

    /*例7 建立学生选课表SC*/
    CREATE TABLE Sc
    (Sno CHAR(9),
    Cno CHAR(4),
    Grade SMALLINT,
    PRIMARY KEY(Sno,Cno),
    /*主码由两个属性构成，必须作为表级完整性进行定义*/
    FOREIGN KEY(Sno) REFERENCES Student(Sno),
    /*表级完整性约束条件，Sno是外码，被参照表是Student*/
    FOREIGN KEY(Cno) REFERENCES Course(Cno),
    /*表级完整性约束条件，Cno是外码，被参照表是Course*/
    );
    
    /*例8 向Student表增加“入学时间”列，其数据类型为日期型*/
    ALTER TABLE Student ADD S_entrance DATETIME;
    
    /*例9 将年龄的数据类型由字符型（假设原来的数据类型是字符型）改为整数*/
    ALTER TABLE Student ALTER COLUMN Sage INT;
    
    /*例10 增加课程名称必须取唯一值的约束条件*/
    ALTER TABLE Course ADD UNIQUE(Cname);
    
    /*例11 删除Student表*/
    DROP TABLE Student CASCADE;
    
    /*例12 若表上建有视图，选择RESTRICTS时表不能删除；CASCADE时可以删除表，视图也自动被删除*/
    CREATE VIEW IS_Student
    AS
    SELECT Sno,Sname,Sage
    FROM Student
    WHERE Sdept = 'IS';
    DROP TABLE Student RESTRICT;    /*删除Student表*/
    --ERROR: cannot drop table Student because other objects depend on it
    /*系统返回错误信息，存在依赖该表的对象，此表不能被删除*/
    DROP TABLE Student CASCADE;     /*删除Student表*/
    --ERROR: drop cascades to view IS_Student
    /*系统返回提示，此表上的视图也被删除*/
    SELECT *  FROM IS_Student;
    --ERROR: relation "IS_Student" does not exist   
    
    /*例13 建立聚簇索引，按照Sname值升序存放*/
    CREATE CLUSTER INDEX Stusname ON Student(Sname);
       
    /*例14 */
    CREATE UNIQUE INDEX Stusno ON Student(Cno);
    CREATE UNIQUE INDEX Coucno ON Course(Cno);
    CREATE UNIQUE INDEX SCno ON SC(Sno ASC,Cno DESC);    
    
    /*例15 删除Student表的Stusname索引*/
    DROP INDEX Stusname;

