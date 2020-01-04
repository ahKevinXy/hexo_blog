---
title: Mysql技术内幕与innodb存储引擎
date: 2019-12-28 00:07:42
tags: [Book,Mysql,InnoDb]
categories: MySQL
---



> 目录

* [第一章 MySQL体系结构和存储引擎](#MySQL体系结构和存储引擎)

* [第二章 InnoDB存储引擎之间的比较](#存储引擎之间的比较)

* [第三章 文件](#文件)

* [第四章 表](#表)

* [第五章 索引与算法](#索引与算法)

* [第六章 锁](#锁)

* [第七章 事务](#事务)

* [第八章 备份与恢复](#备份与恢复)

* [第九章 性能调优](#性能调优)

* [第十章 InnoDb存储引擎源代码的编译](#源代码编译)


## 第一章 MySQL体系结构和存储引擎  <span id='MySQL体系结构和存储引擎'></span>

***

###  定义数据库和实例

* 数据库
    物理操作系统文件或其他形式文件类型的集合

* 数据库实例
    由数据库后台进程/线程以及一个共享内存区组成

* 数据库加载默认配置路径 

    mysql --help | grep my.cnf

    /etc/my.cnf /etc/mysql/my.cnf /usr/lcoal/mysql/etc/my.cnf ~/.my.cnf

###  MySQL体系结构

* MySQL组成部分
1. 连接池组件
2. 管理服务与工具组件
3. sql接口组件
4. 查询分析器组件
4. 优化器组件
5. 缓冲(Cache) 组件
6. 插件式存储引擎
7. 物理文件


###  MySQL 表存储引擎

#### InnoDb 存储引擎


InnoDB 存储引擎支持事务,主要面向在线事务处理(OLTP)方面的应用.其特点是行锁设计,支持外键,并支持类似Oracle 的非锁定读,默认情况下读取操作不会产生锁



InnoDb 存储引擎将数据放在一个逻辑的表空间中,InnoDb使用多版本并发控制(MVCC)来获得高并发性,并且实现了SQL标准的4种隔离,默认为REPETABL级别.同时使用了一种 * next-key locking 的策略来避免幻读(phantom) * .此外InnoDb还提供了插入缓冲,二次写,自适应哈希索引,预读等高性能高可用的功能.
对于表中数据的存储,InnoDB存储引擎采用了聚集(Clustered)的方式,这种方式类似Oracle的索引聚集表.每张表的存储都按照主键的顺序存放,如果没有显式在表定义时指定主键,InnoDB存储引擎会为每一行生成一个6字节的ROWID,并以此为主键

#### MyISAM 存储引擎


MyISAM 存储引擎是MySQL官方提供的存储引擎,其特点是不支持事务,支持表锁和全文索,对于OLAP(Online Analytical Processing,在线分析处理) 操作速度快.MyISAM存储的另一个特点是缓冲池只缓存索引文件

MyISAM 存储引擎表由MYD和MYI组成,MYD用来存放数据文件,MYI用来存放索引文件.可以使用myisampack工具来进一步压缩数据文件(压缩后只读)

#### NDB 存储引擎
NDB 存储引擎是一个集群存储引擎,类似Oracle 的RAC集群,结构是share nothing 的集群架构,因此能提供更高的可用性.NDB的特点是数据全部放在内存中,因此主键查找的速度非常快,并且调过添加NDB数据存储节点可以线性地提高数据库虚拟,是高可用,高性能的集群系统 * NDN存储引擎的连接操作join是在MySQL数据库层完成的而不是存储引擎层完成的 *

#### Memory 存储引擎
Memory 存储引擎是将表中数据存放在内存中,如果数据库重启或反生奔溃,表中的数据将丢失.Memory存储引擎默认使用哈希索引,而不是B+tree索引.虽然Memory存储引擎速度快,只支持表锁,并发性差,并且不支持Text和Blob列类型.存储变长字段是按照定长字段的方式进行的,浪费内存

#### Archive 存储引擎
Archive 存储引擎只支持INSERT和SELECT操作.Archive存储引擎使用zlib(待备注)算法将数据行(row)进行压缩后存储,Archive存储引擎适合存储归档数据,如日志信息.

#### Federated 存储引擎
Federated 存储引擎表并不存放数据,它只是指向一台远程MySQL数据库服务器上

#### Maria 存储引擎
Maria 存储引擎是新开发的引擎,设计主要是用来取代原有的MyISAM存储引擎,从而成为MySQL的默认存储引擎.特点:支持缓存数据和索引文件,应用了行级锁,提供了MVCC功能,支持事务和非事务安全的选项,以及更好的BLOB字符类型的处理性能

#### 其它存储引擎
除了上述的七种存储引擎,还有Merge,csv,Sphinx和infobright

#### 各个存储引擎的比较

### 连接MySQL

连接MySQL操作是一个连接进程和MySQL数据库实例进行通信.本质上是进程通信,常用的进程通信方式有:管道,命名管道,命名名字,TCP/IP套接字,UNIX域套接字


## 第二章 InnoDB存储引擎 <span id="存储引擎之间的比较"> </span>

 ***


InnoDB 是事务安全的MySQL存储引擎.设计采用了类似Oracle数据库的架构.通常来说,InnoDB存储引擎是OLTP应用中核心表的首选存储引擎

### InnoDB存储引擎概述
InnoDB存储引擎由Innobase Oy公司开发

### InnoDB 存储引擎的版本

InnoDB存储引擎被包含于所有MySQL数据库的二进制发行版本

### InnoDB体系架构
InnoDB存储引擎有多个内存块,负责的工作: 维护所有进程/线程需要访问的多个内部数据结构,缓存磁盘上的数据,方便快速地读取,同时在对磁盘文件的数据修改之前在这里缓存. 重做日志缓存

后台线程的主要作用是负责刷新新内存池内的数据,保证缓冲池中的内存缓冲是最近的数据.此外将已修改的数据文件刷新到磁盘文件,同时保证在数据库发生异常的情况下InnoDB能恢复到正常运行状态

#### 后台线程
InnoDB存储引擎是多线程的模型,因此其后台有多个不同的后台线程,负责处理不同的任务

1. Master Thread
Master Thread 是非常核心的后台线程,主要负责将缓冲池中的数据异步刷新到磁盘,保证数据的一致性,包括脏页的刷新,合并插入缓冲去,UNDO页的回收
2. IO Thread
在InnoDB存储引擎中大量使用了AIO来处理IO请求,这样可以极大提高数据的性能.而IO Thread 的工作主要是负责这些IO请求的回调处理.
3. Purge Thread
事务被提交后,其所使用的undolog可能不再需要,因此需要PurgeThread来回收已经使用并分配的undo页,purge操作可以独立到单独的线程中进行,以此来减轻Maser Thread的工作,从而提高CPU的使用率以及提升存储引擎的性能
4. Page Cleaner Thread
Page Cleaner Thread 作用是将之前的版本中脏页的刷新操作都放入到单独的线程中来完成,其目的是为了减轻原Master Thread的工作及对于用户查询线程的阻塞

#### 内存
1. 缓冲池
InnoDB 存储引擎是基于磁盘存储,并将其中的记录按照页的方法进行管理.缓冲池简单的来说就是一块内存区域,通过内存的速度来弥补磁盘速度较慢对数据库性能的影响. 
InnoDB存储引擎而言,缓冲池可以通过innodb_buffer_pool_size来设置
缓冲池中缓存的数据页类型有:索引页,数据页,undo页,插入缓冲(insert buffer),自适应哈希索引(adaptive hash index),InnoDB存储的锁信息(lock info),数据字典信息(data dictionary)
2. LRU List ,Free List Flush List
InnoDB存储引擎中是通过LRU(Lastest Recent Used,最近最少使用)算法来进行管理的,将频繁使用的页在LRU列表前端,少使用的在LRU列表尾端,当缓冲池不能存放新读取到的页时,将首先释放LRU列表尾部的页

3. 重做日志缓冲

4. 额外的内存池


### Checkpoint 技术

缓冲池的作用是为了协调CPU速度与磁盘速度的鸿沟.
Checkpoint(检查点) 技术目的是解决以下几个问题
* 缩短数据库的恢复时间
* 缓冲池不够用时,将脏页刷新到磁盘
* 重做日志不可用时,刷新脏页

触发checkpoint
1. sharp checkpoint
2. Fuzzy checkpoint

sharp checkpoint 发生在数据库关闭时将所有的脏页数据都刷新到磁盘,这是默认方式,即参数innodb_fast_shutdown=1

Fuzzy checkpoint 发生情况:
1. Master Thread checkpoint
2. FLUSH_LRU_LIST checkpoint
3. Async/Sync Flush checkpoint
4. Dirty Page too much checkpoint

### Master Thread 工作方式
***
Innodb存储引擎的主要工作都是在一个单独的后台线程Master Thread中完成

#### InnoDB 1.0.x 版本之前的Master Thread
Master Thread 具有最高的线程优先级别.其内部由多个循环(loop)组成:住循环(loop),后台循环(backgroup loop),刷新循环(flush loop),暂停循环(suspend loop).

#### InnoDB 1.2.x 版本之前的Master Thread


#### InnoDB 1.2.x 版本的Master Thread

### InnoDB 关键特性
InnoDB存储引擎的关键特性:
* 插入缓冲
* 两次写
* 自适应哈希索引
* 异步IO
* 刷新邻接页

#### 插入缓冲
1. Insert Buffer

Insert  Buffer 可能是InnoDb存储引擎关键性中最令人激动与兴奋的一个功能.

2. Change Buffer
InnoDB存储引擎可以对DML操作都进行缓冲

3. Insert Buffer 的内部实现
InnoDB Buffer的数据结构是一种B+树.


4. Merge Insert Buffer

##### 两次写

如果说Insert Buffer 带给InnoDB 存储引擎的是性能的提升,那么Doublewrite 带给 InnoDB存储引擎的是数据页的可靠性


#### 自适应哈希索引

哈希(hash) 是一种非常快的查找方法,在一般情况下这种查找的时间复杂度为O(1),即一般仅需要一次查找就能定位数据.而B+数的查找取决于B+树的高度,在生产环境,B+树的高度一般为3~4层,故需要3~4次的查询

#### 异步 IO
为了提高磁盘操作系统性能,当前的数据库系统采用异步IO(Asynchronous IO)的方式来处理磁盘操作

#### 刷新邻接页

InnoDB存储引擎提供了Flush Neighbor Page的特性.其工作原理: 当刷新一个脏页时,InnoDB存储引擎会检测该页所在区(extent)的所有页,如果是脏页,那么一起进行刷新

### 启动,关闭与恢复

InnoDB是MySQL数据库的存储引擎之一,因此InnoDB存储引擎的启动与关闭,更准确的是指在MySQL实例的启动过程中对InnoDB存储引擎的处理过程.
在关闭时,参数Innodb_fast_shutdown 影响表存储引擎Innodb的行为


## 文件


