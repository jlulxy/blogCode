---
title: mysql存储引擎
date: 2019-12-20 18:50:31
tags: [mysql,存储引擎]
---
### mysql的存储引擎
在文件系统中，mysql将每个数据库（也可以称之为schema）保存为数据目录下的一个子目录。创建表时mysql会在数据库字幕侠创建一个和表同名字的.frm文件保存表的定义。因为mysql使用文件系统的目录和文件来超纯数据库和表的定义，不同的存储引擎保存数据和索引的方式是不同的，单表的则是在mysql服务层统一处理的。

#### InnoDB存储引擎
InnoDB是mysql 默认使用的事务型引擎，也是最重要使用最广泛的存储引擎。被设计用来处理大量短期事务，短期事务大部分情况是正常提交的，很好会被回滚。

##### InooDb概览
InnoDb把在4.1以后可以把每个表的数据结构和索引放在一个单独的文件中（但是我实际操作发现默认是放在同一个文件中）。

**划重点**

***索引数据和表数据分开存储这种理解在InnoDB是错误的，实际上InnoDB的表数据保存在主键索引的B-Tree的叶子节点；***

InnDb才用MVCC的方式支持高并发，实现了四个隔离界别.默认级别（REPEATABLE READ），通过间隙所策略防止幻读的出现，间隙所使得Innodb不仅仅锁定查询设计的行，还得会对缩影中的间隙进行锁定，防止幻影行的插入。
InnoDB数据表是基于聚簇索引建立的。聚簇索引对逐渐的查询有很高的性能，不过二级索引必须包涵主键列，如果主键列比较大索引也会比较大。
作为事务型存储引擎，InnoDB通过一些机制和工具支持热备份（获得一致性视图不需要通知对所有表的写入）

#### myISAM存储引擎
5.1以前的mysql默认是MyIsam引擎。支持大量的特性，包括全文索引，压缩，空就按函数，单不支持事务和行级锁，崩溃后无法安全恢复。

##### MyIsam概述
MyIsam也是存储在两个文件中：数据文件和索引文件，以.MYD和.MYI为扩展名字。
1. 加锁和并发：myIsam只能加表锁，是支持读写锁，值得强调的是在其读的时候是可以向表中插入新的记录（并发插入）
2. 修复：支持手动修复
3. 索引特性：  myIsam表，及时是BLOB和TEXT等长字段也可以基于前500哥字符创建索引。MyIsam支持圈粉索引
4. 延迟更新索引键，清理缓冲区该表的时候才会吧索引块写入到磁盘，必然会照成搞崩溃后的重建，可以再全局设置和单表设置设置延迟更新索引键的特性。
5. MyIsam 压缩表：，对不在加入的新数据的吧IAO进行压缩。
6. 性能L在某些场景性能很好（读大量高于写的时候）