---
title: mysql服务搭建- mysql学习笔记
date:  2019-12-16 20:26:01
tags: mysql 
---

[参考文章链接](https://juejin.im/post/5cee01c0f265da1bca51bdaf)
## mysql服务搭建

### 前置准备
卸载旧mysql &搭建环境创建用户组

```bash
查看rpm包

rpm -qa|grep mysql 若有可用rpm -e卸载

查找mysql残留包，有则删除，没有则忽略

find / -name mysql

安装相关依赖：
yum -y install make gcc-c++ cmake bison-devel ncurses-devel numactl libaio

最好要创建用户和用户组，用来权限管理

groupadd mysql useradd -s /sbin/nologin -g mysql -M mysql
```

### 下载二进制方式下载安装
```bash
查看rpm包

rpm -qa|grep mysql 若有可用rpm -e卸载

查找mysql残留包，有则删除，没有则忽略

find / -name mysql

安装相关依赖：
yum -y install make gcc-c++ cmake bison-devel ncurses-devel numactl libaio

最好要创建用户和用户组，用来权限管理

groupadd mysql useradd -s /sbin/nologin -g mysql -M mysql
```

### 下载解压mysql 配置文件夹权限
```bash
cd /usr/local/
//wget下载或者本地下载后上传
wget https://downloads.mysql.com/archives/get/file/mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
//解压安装包
tar -zxvf mysql-5.7.23-linux-glibc2.12-x86_64.tar.gz
//解压后为了方便后面操作可把解压后文件名修改为mysql
mv mysql-5.7.23-linux-glibc2.12-x86_64 mysql
//更改文件夹所属
chown -R mysql.mysql /usr/local/mysql/

```

###  创建mysql相关目录
```bash
mkdir -p /data/mysql/{data,logs,tmp}
//更改文件夹所属
chown -R mysql.mysql /data/mysql/
注意后续data logs tmp会在配置中记性配置
```

### 创建mysql配置文件my.cof
```bash
我操作的时候发现 /etc/my.conf 已经存在配置如下信息
vi /etc/my.cnf
# 简单模板如下：
[client]
port            = 3306
socket          = /data/mysql/tmp/mysql.sock

[mysqld]
user = mysql
basedir = /usr/local/mysql        
datadir = /data/mysql/data  
port = 3306               

socket = /data/mysql/tmp/mysql.sock
pid-file  = /data/mysql/tmp/mysqld.pid
tmpdir = /data/mysql/tmp    
skip_name_resolve = 1
symbolic-links=0
max_connections = 2000
group_concat_max_len = 1024000
sql_mode = NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
lower_case_table_names = 1
log_timestamps=SYSTEM
character-set-server = utf8
interactive_timeout = 1800  
wait_timeout = 1800
max_allowed_packet = 32M
binlog_cache_size = 4M
sort_buffer_size = 2M
read_buffer_size = 4M
join_buffer_size = 4M
tmp_table_size = 96M
max_heap_table_size = 96M
max_length_for_sort_data = 8096

#logs
server-id = 1003306
log-error = /data/mysql/logs/error.log
slow_query_log = 1
slow_query_log_file = /data/mysql/logs/slow.log
long_query_time = 3
log-bin = /data/mysql/logs/binlog
binlog_format = row
expire_logs_days = 15
log_bin_trust_function_creators = 1
relay-log = /data/mysql/logs/relay-bin
relay-log-recovery = 1  
relay_log_purge = 1  

#innodb  
innodb_file_per_table = 1
innodb_log_buffer_size = 16M
innodb_log_file_size = 256M
innodb_log_files_in_group = 2
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000
innodb_flush_neighbors = 0
innodb_flush_method = O_DIRECT
innodb_autoinc_lock_mode = 2
innodb_read_io_threads = 8
innodb_write_io_threads = 8
innodb_buffer_pool_size = 2G

```

### 配置mysql.server
```bash
/etc/init.d存放服务的启停脚本
cd /usr/local/mysql/support-files
cp mysql.server /etc/init.d/mysql
vi /etc/init.d/mysql   
# 修改目录位置
basedir=/usr/local/mysql
datadir=/data/mysql/data

# 注册开机启动服务
chkconfig --add mysql
chkconfig --list

```

### 启动mysql服务
```bash
echo "PATH=$PATH:/usr/local/mysql/bin  " >> /etc/profile  
source /etc/profile

echo "PATH=$PATH:/usr/local/mysql/bin  " >> /etc/profile  
source /etc/profile

# 启动mysql服务
service mysql start
# 使用初始密码登录mysql服务 并修改密码
mysql -uroot -p
alter user 'root'@'localhost' identified by 'root';
flush privileges;


```


