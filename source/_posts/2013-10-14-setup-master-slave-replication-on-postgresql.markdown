---
layout: post
title: "setup master slave replication on postgresql"
date: 2013-10-14 13:04
comments: true
categories: 
- postgresql
---

####1. 常用的命令

*   `pg_ctl init -D DIR -o "--no-locale"`

    初始化数据目录. `-o`表示要传递给initdb的参数. `--no-locale`将encoding设置为SQL_ASCII,Collation和Ctype设置为C,可以避免各种编码的问题.

    
*   `pg_ctl start -D <DATA DIR> -l <LOG FILE>`

    启动pg

*   `pg_ctl stop  -D <DATA DIR>`

    停止pg

*   `pg_dump <DBNAME> [-t <TBNAME>]...`

    导出数据库/表


<!-- more -->

####2. 配置master slave replication

##### 注意

*   需要先有一个可用的master server。不需要有一个可用的slave server
*   配置过程中需要将master的数据目录rsync到slave机器上
*   最后在slave机器上启动pg服务

##### master端

1.   修改postgresl.conf配置文件中的如下配置项：

        wal_level=hot_standby

        max_wal_senders=5

        wal_keep_segments=32

2.   修改pg_hba.conf,增加:

        host replication    <USER> <NETWORK/MASK>   trust

3.   重启master   

4.   执行

        psql -c "select pg_start_backup('level',true)"

        rsync -ac DATADIR SLAVEHOST:SLAVEDATADIR --exclude postmaster.pid

        psql -c "select pg_stop_backup()"

##### slave端

1.  修改postgresql.conf

        hot_standby=on

2.  新增recovery.conf配置文件

        cp <PGROOT>/share/recovery.conf.sample <DATADIR>/recovery.conf

3.  修改recovery.conf

        standby_mode='on'

        primary_conninfo='host=<HOST of MASTER> port=<PORT of MASTER> user=<USER>'

4.  启动


####3. 验证

#####  方式一

*   在master上执行

        psql -c "select pg_current_xlog_location()"

*   在slave上执行

        psql -c "select pg_last_xlog_receive_location()"

        psql -c "select pg_last_xlog_replay_location()"

通过对比可以看到replicate的进度

#####  方式二

*   在master上执行`ps -ef|grep sender`
*   在slave上执行`ps -ef|grep receiver`

同样，通过对比可以看到replicate的进度


####4. 参考

http://www.postgresql.org/docs/9.1/static/app-initdb.html
http://www.postgresql.org/docs/9.1/static/multibyte.html#MULTIBYTE-CHARSET-SUPPORTED
http://wiki.postgresql.org/wiki/Streaming_Replication
