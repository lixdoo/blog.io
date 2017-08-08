---
layout: post
title: MySQL热备份
date: 2017-8-01
categories: blog
tags: [数据库]
description: 热备份
---


##主从备份 基本原理

利用MySQL自身提供的主从备份机制，基于MySQL内部复制功能。主库中数据库操作存在日志里，从库读取主库日志，执行即成功备份。


##主要步骤
配置环境：Windows
主库地址：192.168.0.210  从库地址：192.168.0.95
#1、设置从库账户
首先登陆主库数据库，在主库MySQL控制台中输入
mysql>GRANT PERLICATION SLAVE ON *.* TO 'backup'@'192.168.0.95' INDETIFIED BY '123456';
mysql>flush privileges;

授权账户完成，可以在从服务器（192.168.0.95）上验证成功：mysql -h192.168.0.210 -ubackup  -p123456;

#2、修改配置文件
在Mysql安装目录下找到配置文件my.ini
主库（192.168.0.210）服务器：
#back
server-id=1
log-bin=backuplog
sync_binlog=1
binlog_format=row
auto_increment_increment = 2
auto_increment_offset = 1
max_binlog_size=512m
expire_logs_days=1
binlog_do_db=jeecms
binlog_ignore_db=mysql
binlog_ignore_db=information_schema

从库（192.168.0.95）服务器：
#back
server-id=2
replicate-do-db=jeecms   （jeecms为同步数据库名）



##注my.ini参数说明（转载[http://www.cnblogs.com/Mr-kevin/p/5590542.html]）

1. server-id
ID值唯一的标识了复制群集中的主从服务器，因此它们必须各不相同。master_id必须为1到2^32－1之间的一个正整数值，slave_id值必须为2到2^32－1之间的一个正整数值。

2. log-bin
表示打开binlog，打开该选项才可以通过I/O写到Slave的relay-log，也是可以进行replication的前提。

3. binlog-do-db
表示需要记录二进制日志的数据库。如果有多个数据可以用逗号分隔，或者使用多个binlog-do-dg选项。

4. binlog-ingore-db
表示不需要记录二进制日志的数据库，如果有多个数据库可用逗号分隔，或者使用多binlog-ignore-db选项。

5. replicate-do-db
表示需要同步的数据库，如果有多个数据可用逗号分隔，或者使用多个replicate-do-db选项。

6. replicate-ignore-db
表示不需要同步的数据库，如果有多个数据库可用逗号分隔，或者使用多个replicate-ignore-db选项。

7. log-slave-updates
配置从库上的更新操作是否写入二进制文件，如果这台从库，还要做其他从库的主库，那么就需要打这个参数，以便从库的从库能够进行日志同步。

8. slave-skip-errors
在复制过程，由于各种原因导致binglo中的sql出错，默认情况下，从库会停止复制，要用户介入。可以设置slave-skip-errors来定义错误号，如果复制过程中遇到的错误是定义的错误号，便可以路过。如果从库是用来做备份，设置这个参数会存在数据不一致，不要使用。如果是分担主库的查询压力,可以考虑。

9. sync_binlog=1 Or N
Sync_binlog的默认值是0,这种模式下,MySQL不会同步到磁盘中去。这样的话，Mysql依赖操作系统来刷新二进制日志 binary log，就像操作系统刷新其他文件的机制一样。因此如果操作系统或机器（不仅仅是 Mysql服务器）崩溃，有可能binlog中最后的语句丢失了。要想防止这种情况，可以使用 sync_binlog全局变量，使binlog在每Ｎ次binlog 写入后与硬盘同步。当sync_binlog 变量设置为１是最安全的，因为在crash崩溃的情况下，你的二进制日志binary log只有可能丢失最多一个语句或者一个事务。但是，这也是最慢的一种方式（除非磁盘有使用带蓄电池后备电源的缓存cache,使得同步到磁盘的操作非常快）。即使sync_binlog设置为1,出现崩溃时,也有可能表内容和binlog内容之间存在不一致性。如果使用 InnoDB表,Mysql服务器处理COMMIT语句,它将整个事务写入binlog并将事务提交到InnoDB中。如果在两次操作之间出现崩溃重启时。事务被InnoDB回滚，但仍然存在binlog中。可以用-innodb-safe-binlog选项来增加InnoDB表内容和binlog之间的一致性。
（注释：在Mysql 5.1版本中不需要-innodb-safe-binlog;由于引入了XA事务支持，该选项作废了），该选项可以提供更大程度的安全，使每个事务的binlog(sync_binlog=1)和（默认情况为真） InnoDB日志与硬盘同步，该选项的效果是崩溃后重启时，在滚回事务后，Mysql服务器从binlog剪切回滚的InnoDB 事务。这样可以确保binlog反馈InnoDB表的确切数据等，并使从服务器保持与主服务器保持同步（不接收回滚的语句）。

10. auto_increment_offset和auto_increment_increment Auto_increment_increment和auto_increment_offset用于主－主服务器（master-to-master）复制，并可以用来控制AUTO_INCREMENT列的操作。两个变量均可以设置为全局或局部变量，并且假定每个值都可以为1到65,535之间的整数值。将其中一个变量设置为0会使该变量为1。这两个变量影响AUTO_INCREMENT列的方式：auto_increment_increment控制列中的值的增量值，auto_increment_offset确定AUTO_INCREMENT列值的起点。如果auto_increment_offset的值于auto_increment_increment的值，则auto_increment_offset的值被忽略。
例如：表内已有一些数据，就会用现在已有的最大自增值做为初始值。

11. max_binlog_size
单个二进制日志文件最大的大小，超过此大小会自动创建新的二进制日志文件

12. expire_logs_days
二进制日志的过期天数，过期的日志将会被自动删除，减少日志占用硬盘的容量

13. binlog_format
二进制日志格式，三种模式：statement语句模式,row行模式，mixed混合模式。
当设置隔离级别为read-commited必须设置二进制日志格式为row，现在mysql官方认为statement这个已经不再适合继续使用；
但mixed类型在默认的事务隔离级别下，可能会导致主从数据不一致；

#3、备份数据库
为了防止出现数据不一致问题，备份数据时要上锁数据库：
mysql>flush tables with read lock;

备份完成后，查询主库状态并记录：
mysql>show master status\G；
记录主库日志偏移量；

解除锁 mysql>unlock tables;

切换到从库上指定同步位置：
mysql>stop slave;
mysql>change master to master_host='192.168.0.210',master_user='backup',master_password='123456',master_log_file='backuplog.000001',master_log_pos=111;
mysql>start slave;

#4、验证
查看从库状态：
mysql>show slave status\G;
Slave_IO_Running、Slave_SQL_Running两项值均为Yes，即表示设置从服务器成功。


###参考资料
- [参考一](http://blog.csdn.net/sleepbird/article/details/1745261)
- [参考二](http://blog.csdn.net/binyao02123202/article/details/19323399)
- [参数三](http://www.cnblogs.com/Mr-kevin/p/5590542.html)
