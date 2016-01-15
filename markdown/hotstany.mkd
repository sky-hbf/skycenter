#postgres 流复制文档（V1）


##一：在主库建立流复制用户
- 建用户命令（例）：su - postgres -c 'creat user repl superuser login password 'repltest';'

- 添加 pg_hba.conf（例）：host    replication     repltest            192.168.1.238/32             trust

## 二：主库上面修改 postgresql.conf,修改完重启数据库，以下为主要参数
- checkpoint_segments = 30
- checkpoint_timeout = 10min
- wal_level = hot_standby
- hot_standby = on
- max_wal_senders = 3
- wal_keep_segments = 16 

## 三：data目录迁移
-  步骤1）在主库上面执行：su - postgres "psql -c 'select pg_start_backup('h1')'";

-  步骤2）拷贝主数据库的data目录到从的机器上面的对应位置

-  步骤3）在从的机器上，进入对应的data目录：rm -f pg_xlog/\*;rm -f pg_log/\* ; rm -f postmaster.pid

-  步骤4）在主的机器上面执行：su - postgres "psql -c 'select pg_stop_backup'";


## 四：创建触发器
> 名称：recovery.conf,放在从机的data目录里面，格式如下（例）

- standby_mode = 'on'
- trigger_file = '/web/data5432/test2.recovery'
- primary_conninfo = 'host=192.168.0.216 port=5432 user=repltest password=123 keepalives_idle=60'




## 五：启动从库确认
-  主库上面有进程类似：wal sender process
-  从库上面有进程类似：wal receiver process

#postgres ,WAL的归档

## 主的postgresql.conf，打开如下配置
-  archive_mode = on
-  archive_command = 'test ! -f /web/walbak/5432wal/%f && cp %p /web/walbak/5432wal/%f'  

## 执行data目录迁移

-  按上面(三)执行

## 写crontab
- */5 * * * * /bin/cp /web/data5432/pg_xlog/* /web/walbak/5432wal/ 
##创建触发器（恢复时使用）

-  standby_mode = on
-  restore_command = 'cp /web/walbak/5432wal/%f  %p'

