---
layout: post
title: MySQL原理
description: 记录
tag: 笔记
---

[TOC]



## 一、存储引擎

### 1.MySQL体系结构

![1.png](https://s2.loli.net/2024/10/11/BzcbPkRofES7x5q.png)

- 连接层：一些客户端和连接服务，主要完成一些类似于连接处理、授权认证及相关的安全方案。服务器也会为安全接入的每个客户端验证它所具有的操作权限。
- 服务层：第二层架构主要完成大多数的核心服务功能，如 SQL 接口，并完成缓存的查询、SQL的分析和优化、部分内置函数的执行。所有跨存储引擎的功能也在这一层实现。
- 引擎层：存储引擎真正的负责了 MySQL 中数据的存储和提取，服务器通过 API 和存储引擎进行通信。不同的存储引擎具有不同的功能。
- 存储层：将数据存储在文件系统之上，完成与存储引擎的交互。



### 2.存储引擎

- 存储引擎简介：存储引擎就是存储数据、建立索引、更新/查询数据等技术的**实现方式**。存储引擎是基于表的，**不同的表可以用不同的存储引擎**，因此存储引擎也被称为**表类型**。

![1.png](https://s2.loli.net/2024/10/11/CTX2eud8OYJW5kP.png)



![1.png](https://s2.loli.net/2024/10/11/PYZ63as8gji9QAR.png)

图中可以看出 **MySQL 的默认存储引擎是 InnoDB**。





- **InnoDB**

  - 介绍：一种兼顾高可靠性和高性能的通用存储引擎，MySQL 5.5 之后，InnoDB 是默认的存储引擎。
  - 特点
    - DML 操作遵循 ACID 模型，支持**事务**
    - **行级锁**，提高并发访问性能
    - 支持**外键**约束，保证数据的完整和可靠性
  - 文件：*xxx.ibd* ：*xxx* 代表的是表名，InnoDB 引擎的每张表都对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引
  - 以下是 InnoDB 的逻辑存储结构

  ![1.png](https://s2.loli.net/2024/10/11/SdZrBR51cXMNJ83.png)



- **存储引擎的选择**

![1.png](https://s2.loli.net/2024/10/11/MQyCxX9EPGhHVtu.png)









## 二、索引

### 1.索引概述

- 概念：索引（index）是帮助 *MySQL* **高效获取数据**的**数据结构**（有序）。这些数据结构满足特定查找算法，并且以某种方式指向数据，这样就可以在这些数据结构上实现高级查找算法。

- 优缺点：

| 优势                                                        | 劣势                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| 提高数据检索效率，降低数据库 IO 成本                        | 索引列也要占据空间                                           |
| 通过索引列对数据进行排序，降低数据排序成本，降低 CPU 的消耗 | 索引大大提高了查询效率，但降低了表的更新速度（更新表时需要操作对应的索引） |



### 2.索引结构

- 索引结构：MySQL 的索引在存储引擎层实现，不同的存储引擎有不同的结构

![1.png](https://s2.loli.net/2024/10/14/izNCLWFbGBVUDad.png)

![1.png](https://s2.loli.net/2024/10/14/gTdYnLGwFSVuQfK.png)

以 **B-Tree和 B+Tree** 为例

1. *B-Tree* （多路平衡查找树）

下面是一棵最大度数为 5 的 *B-Tree* ，每层的节点都会携带数据

![1.png](https://s2.loli.net/2024/10/14/WqCQUXkgfl6oYr3.png)

2. *B+Tree*

下面是一棵最大度数为 4 的 *B+Tree* ，只有叶子节点会携带数据，其他节点仅仅用作索引。并且在叶子节点层，节点与节点之间会形成一个有序的链表。

![1.png](https://s2.loli.net/2024/10/14/uDCqNXSKy4Ggra6.png)





- *MySQL* 中的 *B+Tree*

*MySQL* 索引数据结构对经典的 *B+Tree* 进行了优化。在原有的基础上，增加了一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的 *B+Tree* ，提高区间访问的性能。

![1.png](https://s2.loli.net/2024/10/14/BKE72RHNQ8vCriY.png)

补充：*MySQL* 具有自适应 *hash* 功能，*hash* 索引是存储引擎根据 *B+Tree* 索引在指定条件下自动构建的。

**为什么 *InnoDB* 引擎选择使用 *B+Tree* 索引结构？**

1. 相对于二叉搜索树与红黑树，层级更少，搜索效率高；
2. 对于 *B-Tree* ，无论是叶子节点还是非叶子节点，都会保存数据，这样**导致一页中存储的键值总数减少**，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
3. 相对 *Hash* 索引，*B+Tree* 支持范围匹配及排序操作





### 3.索引分类

![1.png](https://s2.loli.net/2024/10/14/H4DCPA5EkN1lQJT.png)

在 *InnoDB* 存储引擎中，根据索引的存储形式，可以分为以下两种：

![1.png](https://s2.loli.net/2024/10/14/7bUJmVwdjyGNeMZ.png)

聚集索引选取规则：

1. 如果存在主键，主键就是聚集索引
2. 如果不存在主键，将使用第一个唯一（*UNIQUE*）索引作为聚集索引
3. 如果表没有主键，或没有合适的唯一索引，则 *InnoDB* 会自动生成一个 *rowid* 作为隐藏的聚集索引

上面的讲解可能比较抽象，下面这张图可以比较清楚的理解**聚集索引和二级索引的区别**：

![1.png](https://s2.loli.net/2024/10/14/fCKgrNwbjXY4mSz.png)

解释如下：

1. 上图中的表结构有主键 *id* ，因此 *InnoDB* 将 *id* 作为聚集索引来构建一棵 *B+Tree* ，**并且聚集索引唯一**。**这棵 *B+Tree* 的叶子节点存放的是行数据**。
2. 然后我们用 *name* 字段去构建一棵 *B+Tree* 作为二级索引辅助查找，**这棵 *B+Tree* 的叶子节点存放的是对应的主键**。

以一条 SQL 查询为例：`select * from user where name = 'Arm';`

查询过程：查询条件给的是 *name* 字段，因此需要在二级索引中查找到 *Arm* 这个叶子节点，然后查出该叶子节点存放的主键 *id* 数据。因为需要返回的是一行数据，所以还需要回到聚集索引的 *B+Tree* 中，根据二级索引中查到的主键 *id* 查找一整行的数据。**这个过程被称为回表查询**。

![1.png](https://s2.loli.net/2024/10/14/4yHa1jueFlKE6S9.png)





### 4.索引语法

- 创建索引

```mysql
CREATE [UNIQUE|FULLTEXT] INDEX index_name ON table_name (index_col_name,...);
```

- 查看索引

```mysql
SHOW INDEX FROM table_name;
```

- 删除索引

```mysql
DROP INDEX index_name ON table_name;
```





### 5. SQL 性能分析

- SQL 执行频率

![1.png](https://s2.loli.net/2024/10/21/OX1AFlUa32fnLmi.png)

![1.png](https://s2.loli.net/2024/10/21/pB5oNvZkw7A8WfH.png) 



- 慢查询日志

首先需要修改 *mysql* 配置文件 `/etc/my.cnf` ，在其中添加如下内容：

![1.png](https://s2.loli.net/2024/10/21/dYZDcXmfjy4F9vJ.png)

然后重启 mysql ，再登录 mysql 看慢查询日志是否已经启动

![1.png](https://s2.loli.net/2024/10/21/PkmdA2jStz4QWwu.png)

启动之后，慢查询日志将记录到 `/var/lib/mysql/localhost-slow.log` 文件中



- profile 详情

`show profiles` 能够在做 SQL 优化时帮助我们了解时间耗费在哪里。通过 `have_profiling` 参数，能够看到 *mysql* 是否支持 *profile* 操作

![1.png](https://s2.loli.net/2024/10/21/1gKPtG36cWNbh7a.png) 

默认 *profiling* 是关闭的，可以通过 *set* 语句在 *session/global* 级别开启 *profiling*：

![1.png](https://s2.loli.net/2024/10/21/uaDWFHS7UQAsTtk.png) 



开启之后就可以通过如下指令查看之前 SQL 指令的执行耗时：

```mysql
#查看每一条 SQL 的耗时基本情况
show profiles;

#查看指定 query_id 的 SQL 语句各个阶段的耗时情况
show profile for query query_id;

#查看指定 query_id 的 SQL 语句 CPU 的使用情况
show profile cpu for query query_id;
```





- *explain* 执行计划

![1.png](https://s2.loli.net/2024/10/21/dnSjNq4guXO5R39.png)

![1.jpg](https://s2.loli.net/2024/10/21/EMb2376jRvqHBxw.jpg)

![aa8739ff2bcb098721ddc3dc5a43502d.png](https://s2.loli.net/2024/10/21/sqMZBkHdcQaKwEx.png)





### 6.索引使用

1. 最左前缀法则：如果索引关联了多列（联合索引），要遵守该法则。最左前缀法则指的是查询从索引的**最左列**开始，并且**不跳过**索引中的列。如果跳跃某一列，**索引将部分失效（后面的字段索引失效）**

比如下面的第三个 *explain* ，profession 是最左列，但此处并没有用到最左列，所以**索引全部失效**。

![1.jpg](https://s2.loli.net/2024/10/21/6UaebCTWRi7k9nc.jpg)

2. 范围查询：联合索引中，出现范围查询(>,<)，**范围查询右侧的列索引失效**。在业务允许的情况下，如果需要进行范围查询，尽量用**大于等于或小于等于**这种范围查询，这样所有的列索引都不会失效。
3. 索引列运算：不要在索引列上进行运算操作，**索引会失效**。
4. 字符串不加单引号：字符串类型字段使用时，**不加引号索引将失效**。
5. 模糊查询：如果仅仅是**尾部模糊匹配**，索引**不会失效**。如果是**头部模糊匹配**，索引**会失效**。
6. or 连接的条件：用 or 分割开的条件，如果 or 前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。
7. 数据分布影响：如果 *mysql* 评估使用索引比全表扫描更慢，则不使用索引。

**补充：**

1. SQL 提示

![1.jpg](https://s2.loli.net/2024/10/21/aVg3HLKE57XtoBO.jpg)

2. 覆盖索引

覆盖索引是指当一个索引包含了查询语句中所需要的所有列时，MySQL可以不必加载表中的实际数据行，而是直接从索引中获取查询结果。这意味着查询可以直接使用索引来找到并返回所需的数据，而不需要回到主键索引或者实际的数据行（这个过程叫做回表查询），这可以减少磁盘I/O操作，提高查询效率。

![1.png](https://s2.loli.net/2024/10/21/CTNz9lMPfgiqGYV.png)

3. 前缀索引

![1.png](https://s2.loli.net/2024/10/21/vhalfVSy72O43r8.png)

![1.png](https://s2.loli.net/2024/10/21/7Uc9KPBsWHnSF4g.png)





### 7.索引设计原则

1. 针对数据量较大（超过100w），且查询频繁的表建立索引。
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
3. 尽量选择**区分度高**的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点建立**前缀索引**。
5. **尽量使用联合索引**，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表查询。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价越大，会影响增删改的效率。
7. 如果索引列不能存 *NULL* 值，请在创建表时使用 *NOT NULL* 约束它。当优化器知道每列是否包含 *NULL* 值时，它可以更好地确定哪个索引最有效地用于查询。













## 三、SQL优化







## 四、视图/存储过程/触发器







## 五、锁









## 六、InnoDB引擎







## 七、MySQL管理