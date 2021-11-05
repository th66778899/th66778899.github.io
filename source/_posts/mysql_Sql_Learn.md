---
title: Sql练习
date: 2021-11-01 17:35:49
tags: Mysql
categories:
- Mysql
index_img: /img/mysql.png
---

Sql语句练习

<!--more-->

# Mysql 练习题

**我使用的Mysql版本是5.7.19。答案可能会因版本会有少许出入**。

## 练习数据

### 数据表

--1.学生表
Student(SId,Sname,Sage,Ssex) 

--SId 学生编号,Sname 学生姓名,Sage 出生年月,Ssex 学生性别

--2.课程表 
Course(CId,Cname,TId) 
--CId --课程编号,Cname 课程名称,TId 教师编号

--3.教师表 
Teacher(TId,Tname)
 --TId 教师编号,Tname 教师姓名

--4.成绩表 
SC(SId,CId,score)
 --SId 学生编号,CId 课程编号,score 分数

### 创建测试数据

> 总的运行脚本

```SQL
/*
 Navicat Premium Data Transfer

 Source Server         : localhost
 Source Server Type    : MySQL
 Source Server Version : 80025
 Source Host           : localhost:3306
 Source Schema         : sql_exercise

 Target Server Type    : MySQL
 Target Server Version : 80025
 File Encoding         : 65001

 Date: 01/11/2021 17:45:49
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for course
-- ----------------------------
DROP TABLE IF EXISTS `course`;
CREATE TABLE `course`  (
  `CId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Cname` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `TId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of course
-- ----------------------------
INSERT INTO `course` VALUES ('01', '语文', '02');
INSERT INTO `course` VALUES ('02', '数学', '01');
INSERT INTO `course` VALUES ('03', '英语', '03');

-- ----------------------------
-- Table structure for sc
-- ----------------------------
DROP TABLE IF EXISTS `sc`;
CREATE TABLE `sc`  (
  `SId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `CId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `score` decimal(18, 1) NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of sc
-- ----------------------------
INSERT INTO `sc` VALUES ('01', '01', 80.0);
INSERT INTO `sc` VALUES ('01', '02', 90.0);
INSERT INTO `sc` VALUES ('01', '03', 99.0);
INSERT INTO `sc` VALUES ('02', '01', 70.0);
INSERT INTO `sc` VALUES ('02', '02', 60.0);
INSERT INTO `sc` VALUES ('02', '03', 80.0);
INSERT INTO `sc` VALUES ('03', '01', 80.0);
INSERT INTO `sc` VALUES ('03', '02', 80.0);
INSERT INTO `sc` VALUES ('03', '03', 80.0);
INSERT INTO `sc` VALUES ('04', '01', 50.0);
INSERT INTO `sc` VALUES ('04', '02', 30.0);
INSERT INTO `sc` VALUES ('04', '03', 20.0);
INSERT INTO `sc` VALUES ('05', '01', 76.0);
INSERT INTO `sc` VALUES ('05', '02', 87.0);
INSERT INTO `sc` VALUES ('06', '01', 31.0);
INSERT INTO `sc` VALUES ('06', '03', 34.0);
INSERT INTO `sc` VALUES ('07', '02', 89.0);
INSERT INTO `sc` VALUES ('07', '03', 98.0);

-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`  (
  `SId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Sname` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Sage` datetime NULL DEFAULT NULL,
  `Ssex` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES ('01', '赵雷', '1990-01-01 00:00:00', '男');
INSERT INTO `student` VALUES ('02', '钱电', '1990-12-21 00:00:00', '男');
INSERT INTO `student` VALUES ('03', '孙风', '1990-05-20 00:00:00', '男');
INSERT INTO `student` VALUES ('04', '李云', '1990-08-06 00:00:00', '男');
INSERT INTO `student` VALUES ('05', '周梅', '1991-12-01 00:00:00', '女');
INSERT INTO `student` VALUES ('06', '吴兰', '1992-03-01 00:00:00', '女');
INSERT INTO `student` VALUES ('07', '郑竹', '1989-07-01 00:00:00', '女');
INSERT INTO `student` VALUES ('09', '张三', '2017-12-20 00:00:00', '女');
INSERT INTO `student` VALUES ('10', '李四', '2017-12-25 00:00:00', '女');
INSERT INTO `student` VALUES ('11', '李四', '2017-12-30 00:00:00', '女');
INSERT INTO `student` VALUES ('12', '赵六', '2017-01-01 00:00:00', '女');
INSERT INTO `student` VALUES ('13', '孙七', '2018-01-01 00:00:00', '女');

-- ----------------------------
-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher`  (
  `TId` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `Tname` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of teacher
-- ----------------------------
INSERT INTO `teacher` VALUES ('01', '张三');
INSERT INTO `teacher` VALUES ('02', '李四');
INSERT INTO `teacher` VALUES ('03', '王五');

SET FOREIGN_KEY_CHECKS = 1;

```

> 各个表的运行脚本

学生表 Student

```SQL
create table Student(SId varchar(10),Sname varchar(10),Sage datetime,Ssex varchar(10));
insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2017-12-30' , '女');
insert into Student values('12' , '赵六' , '2017-01-01' , '女');
insert into Student values('13' , '孙七' , '2018-01-01' , '女');
```

科目表 Course

```sql
create table Course(CId varchar(10),Cname nvarchar(10),TId varchar(10));
insert into Course values('01' , '语文' , '02');
insert into Course values('02' , '数学' , '01');
insert into Course values('03' , '英语' , '03');
```

教师表 Teacher

```SQL
create table Teacher(TId varchar(10),Tname varchar(10));
insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四');
insert into Teacher values('03' , '王五');
```

成绩表 SC

```SQL
create table SC(SId varchar(10),CId varchar(10),score decimal(18,1));
insert into SC values('01' , '01' , 80);
insert into SC values('01' , '02' , 90);
insert into SC values('01' , '03' , 99);
insert into SC values('02' , '01' , 70);
insert into SC values('02' , '02' , 60);
insert into SC values('02' , '03' , 80);
insert into SC values('03' , '01' , 80);
insert into SC values('03' , '02' , 80);
insert into SC values('03' , '03' , 80);
insert into SC values('04' , '01' , 50);
insert into SC values('04' , '02' , 30);
insert into SC values('04' , '03' , 20);
insert into SC values('05' , '01' , 76);
insert into SC values('05' , '02' , 87); 
insert into SC values('06' , '01' , 31);
insert into SC values('06' , '03' , 34);
insert into SC values('07' , '02' , 89);
insert into SC values('07' , '03' , 98);
```

## 练习题目

### 1.

​	查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

```SQL
select
	t1.SId as `学生id`,
	t1.score as `课程01分数`,
	t2.score as `课程02分数`
from (select SId ,score from sc where sc.CId='01')as t1 , (select SId ,score from sc where sc.CId='02') as t2
where t1.SId=t2.SId
and   t1.score>t2.score
```



1.1 查询同时存在" 01 "课程和" 02 "课程的情况

```SQL
select
	t1.SId as `学生id`,
	t1.score as `课程01分数`,
	t2.score as `课程02分数`
from (select SId ,score from sc where sc.CId='01')as t1 , (select SId ,score from sc where sc.CId='02') as t2
where t1.SId=t2.SId

```

1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )

```sql
SELECT
	t1.SId AS `学生id`,
	t1.score AS `课程01分数`,
	t2.score AS `课程02分数` 
FROM
	( SELECT SId, score FROM sc WHERE sc.CId = '01' ) AS t1
	LEFT JOIN 
	( SELECT SId, score FROM sc WHERE sc.CId = '02' ) AS t2 
ON 
	t1.SId = T2.SId
```



1.3 查询不存在" 01 "课程但存在" 02 "课程的情况

```SQL
SELECT
	SId,
	CId as `课程序号`,
	score AS `02课程成绩`
FROM
	sc 
WHERE 
	sc.SId Not in (SELECT SId FROM sc WHERE sc.CId = 01)
	and sc.CId = 02
```

### 2.

查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩

```SQL
SELECT
	student.*,
	avgScore 
FROM
	student
	INNER JOIN (	
	SELECT
		sc.SId,
		AVG( sc.score ) AS avgScore 
	FROM
		sc 
	GROUP BY
		sc.SId 
	HAVING
		AVG( sc.score ) >= 60 
	) AS t1 
WHERE
	t1.SId = student.SId
```

### 3.查询在 SC 表存在成绩的学生信息

```SQL
SELECT
 DISTINCT student.* 
FROM
	student, sc
WHERE
	student.SId = sc.SId
```

或

```SQL
SELECT
 student.* 
FROM
	student
WHERE
	student.SId in (SELECT SId FROM sc)
	
```

### 4.

查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为null)  

```SQL
SELECT
	student.*,
	sco.courseSum,
	sco.scoreSum 
FROM
	student
	INNER JOIN (
	SELECT
		sc.SId,
		COUNT( sc.CId ) AS courseSum,
		SUM( sc.score ) AS scoreSum 
	FROM
		sc 
	GROUP BY
		sc.SId 
	) AS sco 
WHERE
	student.SId = sco.SId                                         
```

4.1 查有成绩的学生信息

```SQL
SELECT
	student.* 
FROM
	student 
WHERE
	EXISTS (SELECT sc.SId FROM sc WHERE sc.SId = student.SId) 

```

### 5.查询「李」姓老师的数量 

```SQL
SELECT
	COUNT(*)
FROM
	teacher
WHERE 
	teacher.Tname LIKE '李%'
```

### 6.查询学过「张三」老师授课的同学的信息 

```SQL
SELECT 
	student.*
FROM
	student, sc, course, teacher
WHERE
	student.SId = sc.SId
AND
	sc.CId = course.CId
AND
	course.TId = teacher.TId
AND
	teacher.Tname = '张三'

```

### 7.查询没有学全所有课程的同学的信息 

```SQL
SELECT DISTINCT
	student.* 
FROM
	( SELECT student.SId, course.CId FROM student, course ) AS t1
	LEFT JOIN ( SELECT sc.SId, sc.CId FROM sc ) AS t2 ON t1.SId = t2.SId 
	AND t1.CId = t2.CId,
	student 
WHERE
	t2.SId IS NULL 
	AND t1.SId = student.SId
```

### 8.

查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```SQL
SELECT
	DISTINCT student.*
FROM
	student, sc
WHERE
	student.SId = sc.SId
	and
	sc.CId in (SELECT CId FROM sc WHERE sc.SId = '01')
	and
	student.SId != '01'
```

### 9.

查询和" 01 "号的同学学习的课程完全相同的其他同学的信息 

```SQL
select DISTINCT student.*
from (
select student.SId,t.CId
from student ,(select sc.CId from sc where sc.SId='01') as t) as t1 LEFT JOIN sc on t1.SId=sc.SId and t1.CId=sc.CId,student
where sc.SId is null 
and   t1.SId=student.SId
```

