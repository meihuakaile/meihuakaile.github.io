---
title: 'mysql导入中文数据乱码'
date: "2018/04/19"
tags: [mysql]
categories: ['mysql']
copyright: true
---
用 LOAD DATA INFILE 命令导入数据时中文是乱码。

解决：

（1）执行下面命令，看编码。
```
mysql> SHOW VARIABLES LIKE "%CHAR%";
输出：
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```
（2）把全部编码都改成utf8；我只改了character_set_database和character_set_server：
```
set character_set_database=utf8；
set character_set_server=utf8；
```
（3）再次执行（1）中命令：
```
mysql> SHOW VARIABLES LIKE "%CHAR%";
输出：
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```
这样再导入数据就不会有中文乱码了。

注：（2）中的命令。网上说可以先试用命令： set names utf8; 如果变量的输出仍然不是全部都是utf8，再使用上面的一个一个赋值的方法。

但是上面的办法只能用在一个终端里有效。我们可以通过修改数据库的编码永久解决这个问题：
修改数据库编码：
```
alter database you_data_base_name character set utf8;
```
（我后来又试了一下只需要改set character_set_database=utf8;的字符编码就可以了）
原因：
通过命令 status 可以数据库中的状态：
```
mysql> status;
--------------
mysql  Ver 14.14 Distrib 5.7.20, for Linux (x86_64) using  EditLine wrapper

Connection id:        27
Current database:    
Current user:        chenliclchen@localhost
SSL:            Not in use
Current pager:        stdout
Using outfile:        ''
Using delimiter:    ;
Server version:        5.7.20-0ubuntu0.16.04.1 (Ubuntu)
Protocol version:    10
Connection:        Localhost via UNIX socket
Server characterset:    latin1  
Db     characterset:    latin1  ####################################数据库的默认编码
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:        /var/run/mysqld/mysqld.sock
Uptime:            4 hours 38 min 21 sec
```
你在执行过上面的（2）（set character_set_database=utf8；）后再执行 status 可以看到 用“#”标出来的那行的‘latin1’变成了‘utf8’.

附：
– character_set_server：默认的内部操作字符集
– character_set_client：客户端来源数据使用的字符集
– character_set_connection：连接层字符集
– character_set_results：查询结果字符集
– character_set_database：当前选中数据库的默认字符集
– character_set_system：系统元数据(字段名等)字符集
– 还有以collation_开头的同上面对应的变量，用来描述字符序。

参考：https://www.2cto.com/database/201408/326102.html