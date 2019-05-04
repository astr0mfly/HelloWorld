---
date: "2019-04-25T09:48:37-07:00"
draft: true
title: "Mysql-Query_lock"
tags: [mysql]
series: [DataBase]
categories: [“编码人生”]
toc: true
---

SHOW PROCESSLIST;
SHOW OPEN TABLES WHERE In_use > 0;
 
#运行线程
SHOW  GLOBAL  STATUS LIKE 'Threads_running%';
#链接线程
SHOW  GLOBAL  STATUS LIKE 'Threads_connected%';
#查询数
SHOW  GLOBAL  STATUS LIKE 'Queries%';
SHOW  GLOBAL  STATUS LIKE 'Question%';
#提交数
SHOW GLOBAL STATUS LIKE 'Com_commit';
#回滚数
SHOW GLOBAL STATUS LIKE 'Com_rollback';
SHOW STATUS LIKE 'Binlog_cache%';
SHOW INNODB STATUS;
#日志功能参数查询
SHOW VARIABLES LIKE '%pro%';
SHOW PROFILES;
#开启语句执行时间日志功能
SET profiling=0;
#显示数据日志ID
SHOW PROFILES;
SHOW PROFILE ALL FOR QUERY 839810;
 
#查看当前MySQL容量状态
SELECT   
  table_schema AS 'Db Name',  
  ROUND( SUM( data_length + index_length ) / 1024 / 1024, 3 ) AS 'Db Size (MB)',  
  ROUND( SUM( data_free ) / 1024 / 1024, 3 ) AS 'Free Space (MB)'  
FROM information_schema.tables  
GROUP BY table_schema ; 
 
mysql状态查看 QPS/TPS/缓存命中率查看
转载：http://blog.163.com/ji_1006/blog/static/106123412013101421530142/
运行中的mysql状态查看
 
对正在运行的mysql进行监控，其中一个方式就是查看mysql运行状态。 
 
(1)QPS(每秒Query量) 
QPS = Questions(or Queries) / seconds 
mysql > show  global  status like 'Question%'; 
 
(2)TPS(每秒事务量) 
TPS = (Com_commit + Com_rollback) / seconds 
mysql > show global status like 'Com_commit'; 
mysql > show global status like 'Com_rollback'; 
 
(3)key Buffer 命中率 
mysql>show  global   status  like   'key%'; 
key_buffer_read_hits = (1-key_reads / key_read_requests) * 100% 
key_buffer_write_hits = (1-key_writes / key_write_requests) * 100% 
 
(4)InnoDB Buffer命中率 
mysql> show status like 'innodb_buffer_pool_read%'; 
innodb_buffer_read_hits = (1 - innodb_buffer_pool_reads / innodb_buffer_pool_read_requests) * 100% 
 
(5)Query Cache命中率 
mysql> show status like 'Qcache%'; 
Query_cache_hits = (Qcahce_hits / (Qcache_hits + Qcache_inserts )) * 100%; 
 
(6)Table Cache状态量 
mysql> show global  status like 'open%'; 
比较 open_tables  与 opend_tables 值 
 
(7)Thread Cache 命中率 
mysql> show global status like 'Thread%'; 
mysql> show global status like 'Connections'; 
Thread_cache_hits = (1 - Threads_created / connections ) * 100% 
 
(8)锁定状态 
mysql> show global  status like '%lock%'; 
Table_locks_waited/Table_locks_immediate=0.3%  如果这个比值比较大的话，说明表锁造成的阻塞比较严重 
Innodb_row_lock_waits innodb行锁，太大可能是间隙锁造成的 
 
(9)复制延时量 
mysql > show slave status 
查看延时时间 
 
(10) Tmp Table 状况(临时表状况) 
mysql > show status like 'Create_tmp%'; 
Created_tmp_disk_tables/Created_tmp_tables比值最好不要超过10%，如果Created_tmp_tables值比较大， 
可能是排序句子过多或者是连接句子不够优化 
 
(11) Binlog Cache 使用状况 
mysql > show status like 'Binlog_cache%'; 
如果Binlog_cache_disk_use值不为0 ，可能需要调大 binlog_cache_size大小 
 
(12) Innodb_log_waits 量 
mysql > show status like 'innodb_log_waits'; 
Innodb_log_waits值不等于0的话，表明 innodb log  buffer 因为空间不足而等待 
 
比如命令： 
>#show global status; 
虽然可以使用： 
>#show global status like %...%; 
来过滤，但是对应长长的list，每一项都代表什么意思，还是有必要弄清楚。
## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T09:48:37-07:00|
