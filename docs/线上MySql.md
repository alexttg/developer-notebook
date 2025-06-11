##解析MySql binlog

如何拿到线上binlog日志 解析出来拿到自己想要的信息，这里使用的是mysqlbinlog 解析 

###具体操作命令

```
./mysqlbinlog /Users/hk/Downloads/mysql-bin.000011  --base64-output=decode-rows -v --skip-gtids=true |grep -C 100 -i "brands" > /Users/hk/Downloads/brands.log
```

###参数说明


`./mysqlbinlog`:

用于解析MySQL二进制日志的命令，用于将二进制日志转换为可读的纯文本格式。

`/Users/hk/Downloads/mysql-bin.000011`：

解析的二进制日志文件的路径和文件名。

`--base64-output=decode-rows`：

以可读的形式输出二进制日志记录，具体来说是以decode-rows格式输出，这通常用于解析二进制日志中的INSERT、UPDATE和DELETE语句。

`-v`：

以详细模式输出解析过的二进制日志，输出信息包括每个日志事件的详细信息，如时间戳、数据库名、表名、事件类型等。

`--skip-gtids=true`：

跳过GTID相关的信息，GTID是一个全局唯一标识符，用于跟踪在不同MySQL实例之间的复制操作。在这个参数被设置为true时，mysqlbinlog将忽略GTID信息。

`| grep -C 100 -i "brands"`：

将mysqlbinlog命令输出的文本通过管道（|）传递给grep命令。grep是一个用于搜索文本的命令。这个特定的grep命令将会：匹配包含"brands"字符串的行，并输出其前后100行的内容（用于显示上下文），并将搜索结果输出到屏幕上。

> /Users/hk/Downloads/brands.log: 这个参数是将搜索结果输出到brands.log文件中。输出重定向运算符>将命令的标准输出（即控制台上的输出）重定向到指定的文件中。

###解析后日志
```
#230425 20:40:03 server id 2727657857  end_log_pos 381609609 CRC32 0x5ed1b475 	Delete_rows: table id 13129 flags: STMT_END_F
### DELETE FROM `soomey`.`brands`
### WHERE
###   @1=1490
###   @2='播'
###   @3='VIP'
###   @4='2022-10-24 11:38:29'
###   @5=b'0'
###   @6=346831
### DELETE FROM `soomey`.`brands`
### WHERE
###   @1=1490
###   @2='broadcast/播'
###   @3='TMALL'
###   @4='2022-10-24 11:38:29'
###   @5=b'0'
###   @6=346832
### DELETE FROM `soomey`.`brands`
### WHERE
###   @1=1490
###   @2='播（broadcast）'
###   @3='JD'
###   @4='2022-10-24 11:38:29'
###   @5=b'0'
###   @6=346833
### DELETE FROM `soomey`.`brands`
### WHERE
###   @1=1490
###   @2='播 broadcast'
###   @3='DOUYIN'
###   @4='2022-10-24 11:38:29'
###   @5=b'0'
```

