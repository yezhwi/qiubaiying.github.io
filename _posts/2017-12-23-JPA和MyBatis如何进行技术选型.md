---
layout:     post
title:      JPA和MyBatis如何进行技术选型
subtitle:   
date:       2017-12-23
author:     Yezhiwei
header-img: img/WechatIMG38.jpeg
catalog: true
category: java
tags:
    - MyBatis
    - JPA
    - MySQL
---

### 背景

在我们平时的项目中，大家都知道可以使用JPA或者Mybatis作为ORM 层。对JPA和Mybatis如何进行技术选型？（http://www.spring4all.com/question/112）

我将从以下几个方面进行总结。

### MySQL数据库使用规范 

#### 核心规范

* 不在数据库中做运算，复杂运算移到程序端进行。
* 控制单表数据量，建议单库不超300-400个表，单表不超过50个纯INT字段，单表不超过20个CHAR(10)字段，单行不超过200Byte。
* 操持表字段少而精，IO高效、表修复快、提高并发、alter table快，单表字段控制在20-50个内，并且使用合适的字段类型。
* 平衡范式、适当的字段冗余
* 拒绝大SQL、大事务、大批量，避免使用存储过程

#### 字段类规范

* 使用合适取值范围的数值字段类型
* 时间类型字段使用bigint型
* 避免使用NULL字段，很难进行查询优化，无法对列加索引
* 少用并拆分TEXT、BLOB类型的字段，如必须使用则拆分到单独的表中；TEXT类型处理性能远低于VARCHAR
* 不在数据库里存图片
* 避免关键字

#### 索引类规范

* 合理的索引，增加查询速度，但会减慢更新，并不是索引越多越好
* 如果字符字段建索引，必须建前缀索引
* 不在索引列做运算处理，无法使用索引，导致全表扫描
* 不使用强外键关系，外键关系体现在程序中保证约束

#### SQL类规范

* 避免复杂查询
* 尽量不用select *，只取需要的列
* 改写or为in操作，in的个数据，建议在200内
* 避免负向查询和%前缀模糊查询，使用不了索引，导致全表扫描
* 减少count(*)
* limit分页，偏移量越大会越慢，推荐`select id from user where id > 20000 limit 11`
* 高并发DB不建议进行两个表以上的join

### 两者区别

#### MyBatis

MyBatis是一个半自动化的持久层框架，支持定制化SQL、存储过程以及高级映射；避免了几乎所有的 JDBC代码和手动设置参数以及获取结果集，并且可以使用简单的XML或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录。

* 优点

1. 简单易学；
2. 灵活，MyBatis不会对应用程序或者数据库的现有设计强加任何影响。 注解或者使用SQL写在XML里，便于统一管理和优化。通过SQL基本上可以实现我们不使用数据访问框架可以实现的所有功能，或许更多。
3. 解除SQL与程序代码的耦合，SQL和代码的分离，提高了可维护性。
4. 提供映射标签，支持对象与数据库的OMR字段关系映射。
5. 提供对象关系映射标签，支持对象关系组建维护。
6. 提供XML标签，支持编写动态SQL。

#### JPA

最早接触JPA是在几年前使用seam框架的时候，JPA做为seam的持久层。JPA的宗旨是为POJO提供持久化标准规范，实现使用的Hibernate，Hibernate是一个全自动的持久层框架，并且提供了面向对象的SQL支持，不需要编写复杂的SQL语句，直接操作Java对象即可，从而大大降低了代码量，让即使不懂SQL的开发人员，也使程序员更加专注于业务逻辑的实现。对于关联查询，也仅仅是使用一些注解即可完成一些复杂的SQL功能。但是碰到复杂业务的SQL时，想要优化Hibernate这种全自动的OMR显得很费力。但是如果按照上面数据库的使用规范，将复杂的查询放到业务层进行处理，这是没有问题的。

工作以来这两个框架都接触一些，可能一般传统公司、个人开发用Hibernate（JPA）多些，互联网公司更多在用MyBatis。但对于以上两种框架，都提供了很方便的开发方式。

### 业务场景或团队开发习惯

* 对于要快速实现业务需求，JPA（Spring Data JPA）更快一些，只需要继承 `JpaRepository ` 或 `CrudRepository `，就实现了常用方法。
* 另个最新发现使用Spring Data Rest，是在JPA的基础上自动转成了RESTful风格接口，使用JPA加分。

* 但是，MyBaits也有自己的代码生成插件，使用起来也挺方面的，基本上不用自己写代码，就能生成基本的操作。
* 如果公司的开发人员已经习惯了之前的方法命名方式，而JPA是它自己的，改起来不是很方便。为了兼容以前的开发习惯，定制MyBatis更方便些。


### 借助工具，代码生成器

对于开发效率上来说，可以自己写一个代码生成工具，把通用的代码一键生成。


### 总结

* 如果能有很好的数据库规范的话，使用这两个哪个都不会差；
* 如果有能力并且想掌控SQL，MyBaits更方便些；
* 否则就依赖JPA的魔力来快速完成业务开发；
* 另外，Spring官方提供了Spring Data JPA，没有看到MyBatis的哈
* 如果是我个人，倾向使用Spring Data JPA，不用管SQL。




