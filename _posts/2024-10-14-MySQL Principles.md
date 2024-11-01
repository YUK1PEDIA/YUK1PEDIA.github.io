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

**覆盖索引**是指当一个索引包含了查询语句中所需要的所有列时，MySQL可以不必加载表中的实际数据行，而是直接从索引中获取查询结果。这意味着查询可以直接使用索引来找到并返回所需的数据，而不需要回到主键索引或者实际的数据行（这个过程叫做回表查询），这可以减少磁盘I/O操作，提高查询效率。

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

### 1.插入数据

- *insert*

  - 如果我们需要一次性往数据库表中插入多条记录，可以从以下三个方面进行优化。

  ```mysql
  insert into tb_test values(1,'tom');
  insert into tb_test values(2,'cat');
  insert into tb_test values(3,'jerry');
  .....
  ```

  - 优化方案一：批量插入数据

  ```mysql
  Insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
  ```

  - 优化方案二：手动控制事务

  ```mysql
  start transaction;
  insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
  insert into tb_test values(4,'Tom'),(5,'Cat'),(6,'Jerry');
  insert into tb_test values(7,'Tom'),(8,'Cat'),(9,'Jerry');
  commit;
  ```

  - 优化方案三：主键**顺序**插入，性能要高于**乱序**插入

  ```
  主键乱序插入 : 8 1 9 21 88 2 4 15 89 5 7 3
  主键顺序插入 : 1 2 3 4 5 7 8 9 15 21 88 89
  ```




- 大批量插入数据

  - 如果一次性需要插入大批量数据(比如: 几百万的记录)，使用 *insert* 语句插入性能较低，此时可以使用 *MySQL* 数据库提供的 *load* 指令进行插入。操作如下：

  ![1.png](https://s2.loli.net/2024/10/24/mMzY2pj4QSF6vgI.png)

  - 可以执行如下指令，将数据脚本文件中的数据加载到表结构中：

  ```mysql
  -- 客户端连接服务端时，加上参数 -–local-infile
  mysql –-local-infile -u root -p
  -- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
  set global local_infile = 1;
  -- 执行load指令将准备好的数据，加载到表结构中
  load data local infile '/root/sql1.log' into table tb_user fields
  terminated by ',' lines terminated by '\n' ;
  ```

  同样的，主键顺序插入性能高于乱序插入，一下面的插入过程为例：
  
  1. 创建表结构
  
  ```mysql
  CREATE TABLE `tb_user` (
  	`id` INT(11) NOT NULL AUTO_INCREMENT,
  	`username` VARCHAR(50) NOT NULL,
  	`password` VARCHAR(50) NOT NULL,
  	`name` VARCHAR(20) NOT NULL,
  	`birthday` DATE DEFAULT NULL,
  	`sex` CHAR(1) DEFAULT NULL,
  	PRIMARY KEY (`id`),
  	UNIQUE KEY `unique_user_username` (`username`)
  ) ENGINE=INNODB DEFAULT CHARSET=utf8 ;
  ```
  
  2. 设置参数
  
  ```mysql
  -- 客户端连接服务端时，加上参数 -–local-infile
  mysql –-local-infile -u root -p
  -- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
  set global local_infile = 1;
  ```
  
  3. *load* 加载数据
  
  ```mysql
  load data local infile '/root/load_user_100w_sort.sql' into table tb_user
  fields terminated by ',' lines terminated by '\n' ;
  ```
  
  ![1.png](https://s2.loli.net/2024/10/24/MwhOjYHyceW4EDA.png)



### 2.主键优化

在上面提到过这样的说法：**主键顺序插入的性能是要高于乱序插入的**，这是为什么呢？这部分来说明具体的原因，然后再分析一下主键又该如何设计。

1. 数据组织方式

   - 在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表(index organized table IOT)。![1.png](https://s2.loli.net/2024/10/24/vH3plaWItXTYCeS.png)

   行数据，都是存储在聚集索引的叶子节点上的。而我们之前也讲解过InnoDB的逻辑结构图：

   ![1.png](https://s2.loli.net/2024/10/24/XOEU8jgVaHdZWfF.png)

   在InnoDB引擎中，数据行是记录在逻辑结构 page 页中的，而每一个页的大小是固定的，默认16K。那也就意味着， 一个页中所存储的行也是有限的，如果插入的数据行row在该页存储不小，**将会存储到下一个页中**，页与页之间会通过指针连接。



2. 页分裂

   - 页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据(如果一行数据过大，会行溢出)，根据主键排列。

   - 主键顺序插入结果如下：

     1. 从磁盘中申请页， 主键顺序插入

     ![1.png](https://s2.loli.net/2024/10/24/ORQdcPTYyvaBqpC.png)

     2. 第一个页没有满，继续往第一页插入

     ![1.png](https://s2.loli.net/2024/10/24/6qgvFiMUk58AuHm.png)

     3. 当第一个也写满之后，再写入第二个页，页与页之间会通过指针连接

     ![1.png](https://s2.loli.net/2024/10/24/k1L9SgZXs3Bo7AQ.png)

     4. 当第二页写满了，再往第三页写入

     ![1.png](https://s2.loli.net/2024/10/24/LXkifd726Whmeyz.png)

   - 主键乱序插入效果

     1. 假如1#,2#页**都已经写满了**，存放了如图所示的数据

     ![1.png](https://s2.loli.net/2024/10/24/BMcxXqWNYC7olsp.png)

     2. 此时再插入id为50的记录，我们来看看会发生什么现象

        会再次开启一个页，写入新的页中吗？

        ![1.png](https://s2.loli.net/2024/10/24/7dfl9EeS1YmPsMQ.png)

        不会。因为，索引结构的叶子节点是有顺序的。按照顺序，应该存储在47之后。

        ![1.png](https://s2.loli.net/2024/10/24/D5LoGpHWx3Pewml.png)

        但是47所在的1#页，已经写满了，存储不了50对应的数据了。 那么此时会开辟一个新的页 3#。

        ![1.png](https://s2.loli.net/2024/10/24/exSMLFJXz5Rwnd8.png)

        但是并不会直接将50存入3#页，而是会将1#页后一半的数据，移动到3#页，然后在3#页，插入50。

        ![1.png](https://s2.loli.net/2024/10/24/GElhFmfygQx4WMK.png)

        移动数据并插入id为50的数据之后，这三个页之间的数据顺序是有问题的。 1#的下一个页，应该是3#， 3#的下一个页是2#。 所以，此时，需要重新设置链表指针。

        ![1.png](https://s2.loli.net/2024/10/24/YSUTyruRVHplQJ2.png)

        上述的这种现象，称之为 "页分裂"，是比较耗费性能的操作。



3. 页合并

   - 目前表中已有数据的索引结构(叶子节点)如下：

   ![1.png](https://s2.loli.net/2024/10/24/eqKjEMHRp1s8WNO.png)

   - 当我们对已有数据进行删除时，具体的效果：当删除一行记录时，实际上**记录并没有被物理删除**，只是记录被标记（flaged）为删除并且它的空间变得**允许被其他记录声明使用**。

   ![1.png](https://s2.loli.net/2024/10/24/ny79zwBHI4MAULJ.png)

   - 当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。

   ![1.png](https://s2.loli.net/2024/10/24/WKi8acprbLnhZRY.png)

   - 删除数据，并将页合并之后，再次插入新的数据21，则直接插入3#页

   ![1.png](https://s2.loli.net/2024/10/24/lQagt6GimqjLZrM.png)

   这个里面所发生的合并页的这个现象，就称之为 "页合并"。

   **注：MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。**



4. 索引设计原则
   - 满足业务需求的情况下，尽量降低主键的长度。
   - 插入数据时，尽量选择顺序插入，选择使用AUTO_INCREMENT自增主键。
   - 尽量不要使用UUID做主键或者是其他自然主键，如身份证号。
   - 业务操作时，避免对主键的修改。





### 3.order by 优化

MySQL的排序，有两种方式：

1. **Using filesort** : 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sortbuffer中完成排序操作，**所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序**。
2. **Using index** : 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

对于以上的两种排序方式，Using index的性能高，而Using filesort的性能低，**我们在优化排序操作时，尽量要优化为 Using index**。

**order by优化原则:**

1. 根据排序字段建立合适的索引，多字段排序时，也要遵循**最左前缀法则**。
2. 尽量使用覆盖索引。
3. 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。
4.  如果不可避免的出现filesort，大数据量排序时，可以**适当增大排序缓冲区大小sort_buffer_size(默认256k)**。



### 4.group by 优化

在分组操作中，我们需要通过以下两点进行优化，以提升性能：

1. 在分组操作时，可以通过索引来提高效率。
2. 分组操作时，索引的使用也是满足最左前缀法则的。





### 5.limit 优化

在数据量比较大时，如果进行limit分页查询，在查询时，越往后，分页查询效率越低。

![1.png](https://s2.loli.net/2024/10/24/UAaWofOV5DkqBip.png)

通过测试我们会看到，越往后，分页查询效率越低，这就是分页查询的问题所在。

因为，当在进行分页查询时，如果执行 `limit 2000000,10` ，此时需要MySQL排序前 2000010 条记录，**仅仅返回 2000000 - 2000010 的记录**，其他记录丢弃，查询排序的代价非常大 。

优化思路: 一般分页查询时，通过创建 **覆盖索引** 能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。

```mysql
explain select * from tb_sku t , (select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
```





### 6.count 优化

1. 概述

   - 在之前的测试中，我们发现，如果数据量很大，在执行count操作时，是非常耗时的。

   ```mysql
   select count(*) from tb_user ;
   ```

   - MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(*) 的时候会**直接返回这个数**，效率很高； 但是如果是带条件的count，MyISAM也慢。
   - InnoDB 引擎就麻烦了，它执行 count(*) 的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数。
   - 如果说要大幅度提升InnoDB表的count效率，主要的优化思路：**自己计数**(可以借助于redis这样的数据库进行,但是如果是带条件的count又比较麻烦了)。

2. count 用法

   - count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是NULL，累计值就加 1，否则不加，最后返回累计值。
   - 用法：count（*）、count（主键）、count（字段）、count（数字）

   ![1.png](https://s2.loli.net/2024/10/24/78sCbvuhl9QBS2W.png)

   - 按照效率排序的话，count(字段) < count(主键 id) < count(1) ≈ count(*****)，所以尽量使用 count(*****)。



### 7.update 优化

我们主要需要注意一下update语句执行时的注意事项。

```mysql
update course set name = 'javaEE' where id = 1 ;
```

当我们在执行删除的SQL语句时，会锁定id为1这一行的数据，然后事务提交之后，行锁释放。

但是当我们在执行如下SQL时。

```mysql
update course set name = 'SpringBoot' where name = 'PHP' ;
```

当我们开启多个事务，在执行上述的SQL时，我们发现**行锁升级为了表锁**。 导致该update语句的性能大大降低。

**因此，我们需要注意：InnoDB的行锁是针对索引加的锁，不是针对记录加的锁 ,并且该索引不能失效，否则会从行锁升级为表锁 。**







## 四、视图/存储过程/触发器

### 1.视图

1. 介绍：视图（View）是一种**虚拟存在的表**。视图中的数据并不在数据库中实际存在，行和列数据来自于定义视图的查询中使用的表，并且是在使用视图时动态生成的。

  - 通俗的讲，视图只保存了查询的SQL逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。

  - 视图的主要用途包括：
    - **简化复杂的查询**：通过创建视图，可以将复杂的SQL查询语句封装起来，使得后续使用更加简单。
    - **数据抽象和安全**：视图可以用来隐藏底层表的实际结构，仅暴露给用户需要的部分数据，从而实现**数据抽象**。同时，通过限制对某些列或行的访问，可以增强数据库的安全性。
    - **数据整合**：当数据分布在多个表中时，可以通过创建视图来整合这些数据，提供一个统一的访问接口。

  举个例子：假设有一个员工表 `employees` 和一个部门表 `departments`，可以创建一个视图来显示每个部门的员工姓名和部门名称，而不是每次都写复杂的JOIN查询。

  ```mysql
  CREATE VIEW department_employees AS
  SELECT e.name, d.department_name
  FROM employees e
  JOIN departments d ON e.department_id = d.department_id;
  ```

  之后，可以像查询普通表一样查询这个视图：

  ```mysql
  SELECT * FROM department_employees;
  ```

  这将返回所有员工的名字以及他们所在的部门名称。

  - 注意事项
    - 视图本身并不存储数据，它只是一个查询的外壳。因此，更新视图实际上是在更新底层的表。
    - 并不是所有的视图都支持更新操作，这取决于视图的定义和所使用的数据库系统。



2. 语法

   - 创建：

   ```mysql
   CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH [ CASCADED | LOCAL ] CHECK OPTION ]
   ```

   - 查询：

   ```mysql
   查看创建视图语句：SHOW CREATE VIEW 视图名称;
   查看视图数据：SELECT * FROM 视图名称 ...... ;
   ```

   - 修改：

   ```mysql
   方式一：CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句[ WITH [ CASCADED | LOCAL ] CHECK OPTION ]
   方式二：ALTER VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH [ CASCADED | LOCAL ] CHECK OPTION ]
   ```

   - 删除：

   ```mysql
   DROP VIEW [IF EXISTS] 视图名称 [,视图名称] ...
   ```

   - 示例如下：

   ```mysql
   -- 创建视图
   create or replace view stu_v_1 as select id,name from student where id <= 10;
   -- 查询视图
   show create view stu_v_1;
   select * from stu_v_1;
   select * from stu_v_1 where id < 3;
   -- 修改视图
   create or replace view stu_v_1 as select id,name,no from student where id <= 10;
   alter view stu_v_1 as select id,name from student where id <= 10;
   -- 删除视图
   drop view if exists stu_v_1;
   ```

   上述我们演示了，视图应该如何创建、查询、修改、删除，那么我们能不能通过视图来插入、更新数据呢？

   ```mysql
   create or replace view stu_v_1 as select id,name from student where id <= 10 ;
   select * from stu_v_1;
   insert into stu_v_1 values(6,'Tom');
   insert into stu_v_1 values(17,'Tom22');
   ```

   执行上述的SQL会发现，id为6和17的数据都是可以**成功插入**的。 但是我们执行查询，查询出来的数据，**却没有id为17的记录**。

   因为我们在创建视图的时候，指定的条件为 **id<=10**, id为17的数据，是不符合条件的，所以没有查询出来，但是这条数据确实是**已经成功的插入到了基表中**。 

   如果我们定义视图时，如果指定了条件，然后我们在**插入、修改、删除**数据时，是否可以做到必须满足条件才能操作，否则不能够操作呢？ 答案是可以的，这就需要借助于视图的**检查**选项了。

3. 检查选项

   当使用**WITH CHECK OPTION**子句创建视图时，MySQL会通过视图检查正在更改的每个行，例如 插入，更新，删除，以使其符合视图的定义。 **MySQL允许基于另一个视图创建视图**，它还会检查依赖视图中的规则以保持一致性。为了确定检查的范围，mysql提供了两个选项： CASCADED 和 LOCAL，默认值为 CASCADED 。

   - **CASCADED级联**

     - 比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为 cascaded，但是v1视图创建时未指定检查选项。 则在执行检查时，不仅会检查v2，还会级联检查v2的关联视图v1。

       ![1.png](https://s2.loli.net/2024/10/27/T4OQdIwufjyrgLE.png)

   - **LOCAL本地**

     - 比如，v2视图是基于v1视图的，如果在v2视图创建的时候指定了检查选项为 local ，但是v1视图创建时未指定检查选项。 则在执行检查时，只会检查v2，不会检查v2的关联视图v1。

       ![1.png](https://s2.loli.net/2024/10/27/MpdkxvgQXAtw7lj.png)

4. 视图的更新

   - 要使视图可更新，视图中的行与基础表中的行之间**必须存在一对一的关系**。如果视图包含以下任何一项，则该视图不可更新：

      - 聚合函数或窗口函数（SUM()、 MIN()、 MAX()、 COUNT()等）
      -  DISTINCT
      -  GROUP BY
      -  HAVING
      -  UNION 或者 UNION ALL

   - 示例演示

      ```mysql
      create view stu_v_count as select count(*) from student;
      ```

      上述的视图中，就只有一个单行单列的数据，如果我们对这个视图进行更新或插入的，将会报错。

5. 视图的作用

   - 简单：视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。
   - 安全：数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据。
   - 数据独立：视图可帮助用户屏蔽真实表结构变化带来的影响。

6. 案例

   - 为了保证数据库表的安全性，开发人员在操作tb_user表时，只能看到的用户的基本字段，屏蔽手机号和邮箱两个字段。

      ```mysql
      create view tb_user_view as select id,name,profession,age,gender,status,createtime from tb_user;
      select * from tb_user_view;
      ```

   - 查询每个学生所选修的课程（三张表联查），这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。

      ```mysql
      create view tb_stu_course_view as select s.name student_name, s.no student_no, c.name course_name from student s, student_course sc, course c where s.id = sc.studentid and sc.courseid = c.id;
      select * from tb_stu_course_view;
      ```






### 2.存储过程

1. 介绍：**存储过程是事先经过编译并存储在数据库中的一段 SQL 语句的集合**，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。存储过程思想上很简单，**就是数据库 SQL 语言层面的代码封装与重用**。

   ![1.png](https://s2.loli.net/2024/10/27/Vwv3Z97fXNCKD4F.png)

   特点：

   - 封装，复用：可以把某一业务SQL封装在存储过程中，需要用到的时候直接调用即可。
   - 可以接收参数，也可以返回数据：在存储过程中，可以传递参数，也可以接收返回值。
   - 减少网络交互，效率提升：如果涉及到多条SQL，每执行一次都是一次网络传输。 而如果封装在存储过程中，我们只需要网络交互一次可能就可以了。

2. 基本语法：

   - 创建：

     ```mysql
     CREATE PROCEDURE 存储过程名称 ([ 参数列表 ])
     BEGIN
     -- SQL语句
     END ;
     ```

   - 调用：

     ```mysql
     CALL 名称 ([ 参数 ]);
     ```

   - 查看：

     ```mysql
     SELECT * FROM INFORMATION_SCHEMA.ROUTINES WHERE ROUTINE_SCHEMA = 'xxx'; -- 查询指定数据库的存储过程及状态信息
     SHOW CREATE PROCEDURE 存储过程名称 ; -- 查询某个存储过程的定义
     ```

   - 删除

     ```mysql
     DROP PROCEDURE [ IF EXISTS ] 存储过程名称 ；
     ```

     注意：在命令行中，执行创建存储过程的SQL时，需要通过关键字 delimiter 指定SQL语句的结束符。因为命令行中SQL语句的结束符默认是分号 `;` ，如果在IDE中执行则不需要指定。

     ![1.png](https://s2.loli.net/2024/10/27/PBx4riEMjnXsmQJ.png)

   - 演示示例

     ```mysql
     -- 存储过程基本语法
     -- 创建
     create procedure p1()
     begin
     	select count(*) from student;
     end;
     -- 调用
     call p1();
     -- 查看
     select * from information_schema.ROUTINES where ROUTINE_SCHEMA = 'itcast';
     show create procedure p1;
     -- 删除
     drop procedure if exists p1;
     ```

3. 变量

   在MySQL中变量分为三种类型: 系统变量、用户定义变量、局部变量。

   1. **系统变量**：系统变量 是MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局变量（GLOBAL）、会话变量（SESSION）。

      - 查看系统变量

        ```mysql
        SHOW [ SESSION | GLOBAL ] VARIABLES ; -- 查看所有系统变量
        SHOW [ SESSION | GLOBAL ] VARIABLES LIKE '......'; -- 可以通过LIKE模糊匹配方式查找变量
        SELECT @@[SESSION | GLOBAL] 系统变量名; -- 查看指定变量的值
        ```

      - 设置系统变量

        ```mysql
        SET [ SESSION | GLOBAL ] 系统变量名 = 值 ;
        SET @@[SESSION | GLOBAL]系统变量名 = 值 ;
        ```

        注意：如果没有指定SESSION/GLOBAL，默认是SESSION，会话变量。mysql服务重新启动之后，**所设置的全局参数会失效**，要想不失效，可以在 /etc/my.cnf 中配置。 

        - 全局变量(GLOBAL): 全局变量针对于所有的会话。
        - 会话变量(SESSION): 会话变量针对于单个会话，在另外一个会话窗口就不生效了。

      - 演示示例

        ```mysql
        -- 查看系统变量
        show session variables ;
        show session variables like 'auto%';
        show global variables like 'auto%';
        select @@global.autocommit;
        select @@session.autocommit;
        -- 设置系统变量
        set session autocommit = 1;
        insert into course(id, name) VALUES (6, 'ES');
        set global autocommit = 0;
        select @@global.autocommit;
        ```

   2. **用户定义变量**：用户定义变量是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用 "@变量名" 使用就可以。其作用域为当前连接。

      - 赋值（赋值时，可以使用 = ，也可以使用 := ）

        ```mysql
        #方式一
        SET @var_name = expr [, @var_name = expr] ... ;
        SET @var_name := expr [, @var_name := expr] ... ;
        
        #方式二
        SELECT @var_name := expr [, @var_name := expr] ... ;
        SELECT 字段名 INTO @var_name FROM 表名;
        ```

      - 使用

        ```mysql
        SELECT @var_name ;
        ```

        注意：用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。

      - 演示示例

        ```mysql
        -- 赋值
        set @myname = 'itcast';
        set @myage := 10;
        set @mygender := '男',@myhobby := 'java';
        select @mycolor := 'red';
        select count(*) into @mycount from tb_user;
        -- 使用
        select @myname,@myage,@mygender,@myhobby;
        select @mycolor , @mycount;
        select @abc;
        ```

   3. **局部变量**：局部变量是根据需要定义的在局部生效的变量，访问之前，需要**DECLARE**声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的BEGIN ... END块。

      - 声明

        ```mysql
        DECLARE 变量名 变量类型 [DEFAULT ... ] ;
        ```

        变量类型就是数据库字段类型：INT、BIGINT、CHAR、VARCHAR、DATE、TIME等。DEFAULT是指定局部变量的默认值。

      - 赋值

        ```mysql
        SET 变量名 = 值 ;
        SET 变量名 := 值 ;
        SELECT 字段名 INTO 变量名 FROM 表名 ... ;
        ```

      - 演示示例

        ```mysql
        -- 声明局部变量 - declare
        -- 赋值
        create procedure p2()
        begin
        declare stu_count int default 0;
        # 将学生表的数据个数赋值给 stu_count 这个局部变量
        select count(*) into stu_count from student;
        select stu_count;
        end;
        call p2();
        ```

4. if

   - 介绍：if 用于做条件判断，具体的语法结构为：

      ```mysql
      IF 条件1 THEN
      .....
      ELSEIF 条件2 THEN -- 可选
      .....
      ELSE -- 可选
      .....
      END IF;
      ```

      在if条件判断的结构中，ELSE IF 结构可以有多个，也可以没有。 ELSE结构可以有，也可以没有。

   - 案例：根据定义的分数score变量，判定当前分数对应的分数等级

      - score >= 85分，等级为优秀。
      - score >= 60分 且 score < 85分，等级为及格。
      - score < 60分，等级为不及格。

      ```mysql
      create procedure p3()
      begin
      	declare score int default 58;
      	declare result varchar(10);
      	if score >= 85 then
      		set result := '优秀';
      	elseif score >= 60 then
      		set result := '及格';
      	else
      		set result := '不及格';
      	end if;
      	select result;
      end;
      call p3();
      ```

      上述的需求我们虽然已经实现了，但是也存在一些问题，比如：score 分数我们是在存储过程中定义死的，而且对于最终计算出来的分数等级，我们也仅仅是最终查询展示出来而已。

      那么我们能不能**把score分数动态的传递进来**，计算出来的分数等级是否可以作为返回值返回呢？答案是肯定的，我们可以通过接下来所讲解的 **参数** 来解决上述的问题。

5. 参数

   - 参数的类型主要分为以下三种：IN、OUT、INOUT。 具体的含义如下：

   | 类型  | 含义                                         | 备注 |
   | ----- | -------------------------------------------- | ---- |
   | IN    | 该类参数作为输入，也就是需要调用时传入值     | 默认 |
   | OUT   | 该类参数作为输出，也就是该参数可以作为返回值 |      |
   | INOUT | 既可以作为输入参数，也可以作为输出参数       |      |

   - 用法：

   ```mysql
   CREATE PROCEDURE 存储过程名称 ([ IN/OUT/INOUT 参数名 参数类型 ])
   BEGIN
   -- SQL语句
   END ;
   ```

   于是上面 **if** 中的案例可以改成下面形式：

   ```mysql
   create procedure p4(in score int, out result varchar(10))
   begin
   	if score >= 85 then
   		set result := '优秀';
   	elseif score >= 60 then
   		set result := '及格';
   	else
   		set result := '不及格';
   	end if;
   end;
   -- 定义用户变量 @result来接收返回的数据, 用户变量可以不用声明
   call p4(18, @result);
   select @result;
   ```

6. case

   - 语法

      - 语法1：

         ```mysql
         -- 含义：当case_value的值为 when_value1时，执行statement_list1，当值为 when_value2时，执行statement_list2， 否则就执行 statement_list
         CASE case_value
         	WHEN when_value1 THEN statement_list1
         	[ WHEN when_value2 THEN statement_list2] ...
         	[ ELSE statement_list ]
         END CASE;
         ```

      - 语法2：

         ```mysql
         -- 含义：当条件search_condition1成立时，执行statement_list1，当条件search_condition2成立时，执行statement_list2， 否则就执行 statement_list
         CASE
         	WHEN search_condition1 THEN statement_list1
         	[WHEN search_condition2 THEN statement_list2] ...
         	[ELSE statement_list]
         END CASE;
         ```

   - 案例：

      - 根据传入的月份，判定月份所属的季节（要求采用case结构）。

         - 1-3月份，为第一季度
         - 4-6月份，为第二季度
         - 7-9月份，为第三季度
         - 10-12月份，为第四季度

         ```mysql
         create procedure p6(in month int)
         begin
         	declare result varchar(10);
         	case
         		when month >= 1 and month <= 3 then
         			set result := '第一季度';
         		when month >= 4 and month <= 6 then
         			set result := '第二季度';
         		when month >= 7 and month <= 9 then
         			set result := '第三季度';
         		when month >= 10 and month <= 12 then
         			set result := '第四季度';
         		else
         			set result := '非法参数';
         	end case ;
         	select concat('您输入的月份为: ',month, ', 所属的季度为: ',result);
         end;
         call p6(16);
         ```

         **注意：如果判定条件有多个，多个条件之间，可以使用 and 或 or 进行连接。**

7. while

   - while 循环是有条件的循环控制语句。满足条件后，再执行循环体中的SQL语句。具体语法为：

     ```mysql
     -- 先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
     WHILE 条件 DO
     	SQL逻辑...
     END WHILE;
     ```

   - 案例

     ```mysql
     -- A. 定义局部变量, 记录累加之后的值;
     -- B. 每循环一次, 就会对n进行减1 , 如果n减到0, 则退出循环
     create procedure p7(in n int)
     begin
     	declare total int default 0;
     	while n>0 do
     		set total := total + n;
     		set n := n - 1;
     	end while;
     	select total;
     end;
     call p7(100);
     ```

8. repeat

   - repeat是有条件的循环控制语句, 当满足until声明的条件的时候，则退出循环 。具体语法为：

     ```mysql
     -- 先执行一次逻辑，然后判定UNTIL条件是否满足，如果满足，则退出。如果不满足，则继续下一次循环
     REPEAT
     	SQL逻辑...
     UNTIL 条件
     END REPEAT;
     ```

   - 案例：计算从1累加到n的值，n为传入的参数值。(使用repeat实现)

     ```mysql
     -- A. 定义局部变量, 记录累加之后的值;
     -- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环
     create procedure p8(in n int)
     begin
     	declare total int default 0;
     	repeat
     		set total := total + n;
     		set n := n - 1;
     	until n <= 0
     	end repeat;
     	select total;
     end;
     call p8(10);
     call p8(100);
     ```

9. loop

   - LOOP 实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。LOOP可以配合一下两个语句使用：

      - LEAVE ：配合循环使用，退出循环。
      - ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。

      ```mysql
      [begin_label:] LOOP
      	SQL逻辑...
      END LOOP [end_label];
      LEAVE label; -- 退出指定标记的循环体
      ITERATE label; -- 直接进入下一次循环
      ```

      上述语法中出现的 begin_label，end_label，label 指的都是我们所自定义的标记。

   - 案例一：计算从1累加到n的值，n为传入的参数值。

      ```mysql
      -- A. 定义局部变量, 记录累加之后的值;
      -- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环 ----> leave xx
      create procedure p9(in n int)
      begin
      	declare total int default 0;
      	sum:loop
      		if n<=0 then
      			leave sum;
      		end if;
      		set total := total + n;
      		set n := n - 1;
      	end loop sum;
      	select total;
      end;
      call p9(100);
      ```

   - 案例二：计算从1到n之间的偶数累加的值，n为传入的参数值。

      ```mysql
      -- A. 定义局部变量, 记录累加之后的值;
      -- B. 每循环一次, 就会对n进行-1 , 如果n减到0, 则退出循环 ----> leave xx
      -- C. 如果当次累加的数据是奇数, 则直接进入下一次循环. --------> iterate xx
      create procedure p10(in n int)
      begin
      	declare total int default 0;
      	sum:loop
      		if n<=0 then
      			leave sum;
      		end if;
      		if n%2 = 1 then
      			set n := n - 1;
      			iterate sum;
      		end if;
      		set total := total + n;
      		set n := n - 1;
      	end loop sum;
      	select total;
      end;
      call p10(100);
      ```

10. 游标

   - 游标（CURSOR）是用来**存储查询结果集**的数据类型 , 在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用包括游标的声明、OPEN、FETCH 和 CLOSE，其语法分别如下。

      - 声明游标

         ```mysql
         DECLARE 游标名称 CURSOR FOR 查询语句 ;
         ```

      - 打开游标

         ```mysql
         OPEN 游标名称 ;
         ```

      - 获取游标记录

         ```mysql
         FETCH 游标名称 INTO 变量 [, 变量 ] ;
         ```

      - 关闭游标

         ```mysql
         CLOSE 游标名称 ;
         ```

   - 案例：根据传入的参数uage，来查询用户表**tb_user**中，所有的用户年龄小于等于**uage**的用户姓名**（name）**和专业**（profession）**，并将用户的姓名和专业插入到所创建的一张新表**(id,name,profession)**中。

      ```mysql
      -- 逻辑:
      -- A. 声明游标, 存储查询结果集
      -- B. 准备: 创建表结构
      -- C. 开启游标
      -- D. 获取游标中的记录
      -- E. 插入数据到新表中
      -- F. 关闭游标
      create procedure p11(in uage int)
      begin
      	declare uname varchar(100);
      	declare upro varchar(100);
      	# 注意游标的声明要在普通变量声明之后，否则会报错
      	declare u_cursor cursor for select name,profession from tb_user where age <= uage;
      	drop table if exists tb_user_pro;
      	create table if not exists tb_user_pro(
      		id int primary key auto_increment,
      		name varchar(100),
      		profession varchar(100)
      	);	
      	open u_cursor;
      	while true do
      		fetch u_cursor into uname,upro;
      		insert into tb_user_pro values (null, uname, upro);
      	end while;
      	close u_cursor;
      end;
      call p11(30);
      ```
      
      上述的存储过程，最终我们在调用的过程中会报错，之所以报错是因为在上面的while循环中并没有退出条件。当游标的数据集获取完毕之后，再次获取数据就会报错（游标中已经没有数据可供获取），从而终止了程序的执行。
      
      ![1.png](https://s2.loli.net/2024/10/28/PyaFSIQpnL4bcgA.png)
      
      但是此时，tb_user_pro表结构及其数据都已经插入成功了，我们可以直接刷新表结构，检查表结构中的数据。
      
      ![1.png](https://s2.loli.net/2024/10/28/bzPZQFGjdmk8WiU.png)
      
      上述的功能虽然我们实现了，但是逻辑并不完善，而且程序执行完毕获取不到数据，数据库还报错。 接下来，我们就需要来完成这个存储过程，并且解决这个问题。
      
      要想解决这个问题，就需要通过MySQL中提供的 **条件处理程序 Handler** 来解决。

11. 条件处理程序

    - **条件处理程序（Handler）**可以用来定义在流程控制结构执行过程中**遇到问题时相应的处理步骤**。具体语法为：

       ```mysql
       DECLARE handler_action HANDLER FOR condition_value [, condition_value] ... statement ;
       
       handler_action 的取值：
       	CONTINUE: 继续执行当前程序
       	EXIT: 终止执行当前程序
       	
       condition_value 的取值：
       	SQLSTATE sqlstate_value: 状态码，如 02000（表示没有数据了）
       	SQLWARNING: 所有以01开头的SQLSTATE代码的简写
       	NOT FOUND: 所有以02开头的SQLSTATE代码的简写
       	SQLEXCEPTION: 所有没有被SQLWARNING 或 NOT FOUND捕获的SQLSTATE代码的简写
       ```

    - 案例：我们继续来完成在上一小节提出的这个需求，并解决其中的问题。根据传入的参数uage，来查询用户表tb_user中，所有的用户年龄小于等于uage的用户姓名（name）和专业（profession），并将用户的姓名和专业插入到所创建的一张新表(id,name,profession)中。

       - 通过SQLSTATE指定具体的状态码

         ```mysql
         -- 逻辑:
         -- A. 声明游标, 存储查询结果集
         -- B. 准备: 创建表结构
         -- C. 开启游标
         -- D. 获取游标中的记录
         -- E. 插入数据到新表中
         -- F. 关闭游标
         create procedure p11(in uage int)
         begin
         	declare uname varchar(100);
         	declare upro varchar(100);
         	declare u_cursor cursor for select name,profession from tb_user where age <= uage;
         	
         	-- 声明条件处理程序 ： 当SQL语句执行抛出的状态码为02000时，将关闭游标u_cursor，并退出
         	declare exit handler for SQLSTATE '02000' close u_cursor;
         	
         	drop table if exists tb_user_pro;
         	create table if not exists tb_user_pro(
         		id int primary key auto_increment,
         		name varchar(100),
         		profession varchar(100)
         	);
         	
         	open u_cursor;
         	
         	while true do
         		fetch u_cursor into uname,upro;
         		insert into tb_user_pro values (null, uname, upro);
         	end while;
         	
         	close u_cursor;
         	
         end;
         call p11(30);
         ```
    
       - 通过SQLSTATE的代码简写方式 NOT FOUND。02 开头的状态码，代码简写为 NOT FOUND
    
         ```mysql
         create procedure p12(in uage int)
         begin
         	declare uname varchar(100);
         	declare upro varchar(100);
         	declare u_cursor cursor for select name,profession from tb_user where age <= uage;
         	
         	-- 声明条件处理程序 ： 当SQL语句执行抛出的状态码为02开头时，将关闭游标u_cursor，并退出
         	declare exit handler for not found close u_cursor;
         	
         	drop table if exists tb_user_pro;
         	create table if not exists tb_user_pro(
         		id int primary key auto_increment,
         		name varchar(100),
         		profession varchar(100)
         	);
         	
         	open u_cursor;
         	
         	while true do
         		fetch u_cursor into uname,upro;
         		insert into tb_user_pro values (null, uname, upro);
         	end while;
         	
         	close u_cursor;
         end;
         call p12(30);
         ```




### 3.存储函数

- 存储函数是**有返回值的存储过程**，存储函数的参数只能是IN类型的。具体语法如下：

  ```mysql
  CREATE FUNCTION 存储函数名称 ([ 参数列表 ])
  RETURNS type [characteristic ...]
  BEGIN
  	-- SQL语句
  	RETURN ...;
  END ;
  ```

- characteristic说明：

  - DETERMINISTIC：相同的输入参数总是产生相同的结果

  - NO SQL ：不包含 SQL 语句

  - READS SQL DATA：包含读取数据的语句，但不包含写入数据的语句

- 案例：计算从1累加到n的值，n为传入的参数值。

  ```mysql
  create function fun1(n int)
  returns int deterministic
  begin
  	declare total int default 0;
  	while n>0 do
  		set total := total + n;
  		set n := n - 1;
  	end while;
  	return total;
  end;
  
  select fun1(50);
  ```

  

### 4.触发器

1. 介绍：触发器是与表有关的数据库对象，指在 insert/update/delete 之前(BEFORE)或之后(AFTER)，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性, 日志记录 , 数据校验等操作 。使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发。

   | 触发器类型      | NEW 和 OLD                                              |
   | --------------- | ------------------------------------------------------- |
   | INSERT 型触发器 | NEW 表示将要或者已经新增的数据                          |
   | UPDATE 型触发器 | OLD 表示修改之前的数据 , NEW 表示将要或已经修改后的数据 |
   | DELETE 型触发器 | OLD 表示将要或者已经删除的数据                          |

2. 语法

   - 创建

     ```mysql
     CREATE TRIGGER trigger_name
     BEFORE/AFTER INSERT/UPDATE/DELETE
     ON tbl_name FOR EACH ROW -- 行级触发器
     BEGIN
     	trigger_stmt ;
     END;
     ```

   - 查看

     ```mysql
     SHOW TRIGGERS ;
     ```

   - 删除

     ```mysql
     DROP TRIGGER [schema_name.]trigger_name ; -- 如果没有指定 schema_name，默认为当前数据库。
     ```

3. 案例：通过触发器记录 tb_user 表的数据变更日志，将变更日志插入到日志表user_logs中, 包含增加,修改 , 删除 ;

   - 表结构准备

     ```mysql
     -- 准备工作 : 日志表 user_logs
     create table user_logs(
     	id int(11) not null auto_increment,
     	operation varchar(20) not null comment '操作类型, insert/update/delete',
     	operate_time datetime not null comment '操作时间',
     	operate_id int(11) not null comment '操作的ID',
     	operate_params varchar(500) comment '操作参数',
     	primary key(`id`)
     )engine=innodb default charset=utf8;
     ```

   - 插入数据触发器

     ```mysql
     create trigger tb_user_insert_trigger
     	after insert on tb_user for each row
     begin
     	insert into user_logs(id, operation, operate_time, operate_id, operate_params)
     VALUES
     	(null, 'insert', now(), new.id, concat('插入的数据内容为:
     	id=',new.id,',name=',new.name, ', phone=', NEW.phone, ', email=', NEW.email, ',
     	profession=', NEW.profession));
     end;
     ```

     

   







## 五、锁









## 六、InnoDB引擎







## 七、MySQL管理