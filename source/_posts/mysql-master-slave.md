---
title: mysql主从备份
date: 2019-04-03 17:19:55
tags:
---
# master的配置与启动
## 配置容器
* mkdir -p /etc/docker-data/mysql1/conf.d
* vi /etc/docker-data/mysql1/conf.d/mysql.cnf
```
[mysqld]
pid-file    = /var/run/mysqld/mysqld.pid
socket    = /var/run/mysqld/mysqld.sock
datadir    = /var/lib/mysql

symbolic-links=0

character-set-server = utf8
#skip-networking
innodb_print_all_deadlocks = 1
max_connections = 2000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M

read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 28M
key_buffer_size = 4M

thread_cache_size = 8

query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M

ft_min_word_len = 4

log-bin = mysql-bin
server-id = 1
binlog_format = mixed

performance_schema = 0
explicit_defaults_for_timestamp

#lower_case_table_names = 1

interactive_timeout = 28800
wait_timeout = 28800

# Recommended in standard MySQL setup

sql_mode=NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER,STRICT_TRANS_TABLES

[mysqldump]
quick
max_allowed_packet = 16M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
```
## 启动容器
docker run --name mysql1 -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d -v /etc/docker-data/mysql1/conf.d:/etc/mysql/conf.d -v /logs/docker-data/mysql1:/logs -v /var/lib/docker-data/mysql1:/var/lib/mysql mysql:5.6
## 进入容器开启slave远程访问
```
mysql> GRANT REPLICATION SLAVE ON *.* to 'reader'@'%' identified by 'readerpwd';

Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```
# slave的配置与启动
## 配置容器
* mkdir -p /etc/docker-data/mysql2/conf.d
* vi /etc/docker-data/mysql2/conf.d/mysql.cnf
```
[mysqld]
pid-file    = /var/run/mysqld/mysqld.pid
socket    = /var/run/mysqld/mysqld.sock
datadir    = /var/lib/mysql

symbolic-links=0

character-set-server = utf8
#skip-networking
innodb_print_all_deadlocks = 1
max_connections = 2000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M

read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 28M
key_buffer_size = 4M

thread_cache_size = 8

query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M

ft_min_word_len = 4

log-bin = mysql-bin
server-id = 2
binlog_format = mixed

performance_schema = 0
explicit_defaults_for_timestamp

#lower_case_table_names = 1

interactive_timeout = 28800
wait_timeout = 28800

# Recommended in standard MySQL setup

sql_mode=NO_ENGINE_SUBSTITUTION,NO_AUTO_CREATE_USER,STRICT_TRANS_TABLES

[mysqldump]
quick
max_allowed_packet = 16M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
```
## 启动容器
* docker run --name mysql2 -e MYSQL_ROOT_PASSWORD=123456 -p 3307:3306 -d -v /etc/docker-data/mysql2/conf.d:/etc/mysql/conf.d -v /logs/docker-data/mysql2:/logs -v /var/lib/docker-data/mysql2:/var/lib/mysql mysql:5.6

# 主从复制配置
* 进入master
```
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000007 |      675 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
row in set (0.00 sec)
```
* 进入slave
```
mysql> change master to master_host='172.16.114.118',master_user='reader',master_password='readerpwd',master_log_file='mysql-bin.000007',master_log_pos=675;
Query OK, 0 rows affected, 2 warnings (0.03 sec)

mysql> start slave;
Query OK, 0 rows affected (0.03 sec)



mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
               Master_Host: 192.168.1.9
                  Master_User: reader
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 501
               Relay_Log_File: 0b763a8d1ddd-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000007
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```
    * 其中172.16.114.118是docker宿主机ifconfig查得的结果
    * 要保证master和slave初始数据状态完全一致

# 主从复制测试
## 进入master
```
mysql> create database slavetest;
Query OK, 1 row affected (0.00 sec)
```
## 进入slave
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| slavetest          |
| sys                |
+--------------------+
rows in set (0.00 sec)
```