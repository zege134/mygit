### 1、redis4.0.1安装（三个节点）

```shell
[root@redis-master ~]# cd /usr/local/src/
[root@redis-master src]# vim install_redis.sh
#!/usr/bin/env bash
# It's Used to be install redis.
# Created on 2018/04/08 11:18.
# @author: wangshibo.
# Version: 1.0
   
function install_redis () {
#################################################################################################
        cd /usr/local/src
        if [ ! -f " redis-4.0.1.tar.gz" ]; then
           wget http://download.redis.io/releases/redis-4.0.1.tar.gz
        fi
        cd /usr/local/src
        tar -zxvf /usr/local/src/redis-4.0.1.tar.gz
        cd redis-4.0.1
        make PREFIX=/usr/local/redis install
        mkdir -p /usr/local/redis/{etc,var}
        rsync -avz redis.conf  /usr/local/redis/etc/
        sed -i 's@pidfile.*@pidfile /var/run/redis-server.pid@' /usr/local/redis/etc/redis.conf
        sed -i "s@logfile.*@logfile /usr/local/redis/var/redis.log@" /usr/local/redis/etc/redis.conf
        sed -i "s@^dir.*@dir /usr/local/redis/var@" /usr/local/redis/etc/redis.conf
        sed -i 's/daemonize no/daemonize yes/g' /usr/local/redis/etc/redis.conf
        sed -i 's/^# bind 127.0.0.1/bind 0.0.0.0/g' /usr/local/redis/etc/redis.conf
 #################################################################################################
}
   
install_redis
[root@redis-master src]# chmod 755 install_redis.sh
[root@redis-master src]# sh -x install_redis.sh
```

###  **2、redis启停脚本（三个节点上都要操作）** 

```shell
[root@redis-master src]# vim /etc/init.d/redis-server
#!/bin/bash
#
# redis - this script starts and stops the redis-server daemon
#
# chkconfig:   - 85 15
# description:  Redis is a persistent key-value database
# processname: redis-server
# config:      /usr/local/redis/etc/redis.conf
# config:      /etc/sysconfig/redis
# pidfile:     /usr/local/redis/var/redis-server.pid
  
# Source function library.
. /etc/rc.d/init.d/functions
  
# Source networking configuration.
. /etc/sysconfig/network
  
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
  
redis="/usr/local/redis/bin/redis-server"
prog=$(basename $redis)
  
REDIS_CONF_FILE="/usr/local/redis/etc/redis.conf"
  
[ -f /etc/sysconfig/redis ] && . /etc/sysconfig/redis
  
lockfile=/var/lock/subsys/redis-server
  
start() {
    [ -x $redis ] || exit 5
    [ -f $REDIS_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $redis $REDIS_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
  
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
  
restart() {
    stop
    start
}
  
reload() {
    echo -n $"Reloading $prog: "
    killproc $redis -HUP
    RETVAL=$?
    echo
}
  
force_reload() {
    restart
}
  
rh_status() {
    status $prog
}
  
rh_status_q() {
    rh_status >/dev/null 2>&1
}
  
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
        exit 2
esac
 
执行权限
[root@redis-master src]# chmod 755 /etc/init.d/redis-server
```

###  **3、redis-sentinel启停脚本示例（三个节点上都要操作）** 

```shell
[root@redis-master src]# vim /etc/init.d/redis-sentinel
#!/bin/bash
#
# redis-sentinel - this script starts and stops the redis-server sentinel daemon
#
# chkconfig:   - 85 15
# description:  Redis sentinel
# processname: redis-server
# config:      /usr/local/redis/etc/sentinel.conf
# config:      /etc/sysconfig/redis
# pidfile:     /usr/local/redis/var/redis-sentinel.pid
  
# Source function library.
. /etc/rc.d/init.d/functions
  
# Source networking configuration.
. /etc/sysconfig/network
  
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
  
redis="/usr/local/redis/bin/redis-sentinel"
prog=$(basename $redis)
  
REDIS_CONF_FILE="/usr/local/redis/etc/sentinel.conf"
  
[ -f /etc/sysconfig/redis ] && . /etc/sysconfig/redis
  
lockfile=/var/lock/subsys/redis-sentinel
  
start() {
    [ -x $redis ] || exit 5
    [ -f $REDIS_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $redis $REDIS_CONF_FILE --sentinel
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
  
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
  
restart() {
    stop
    start
}
  
reload() {
    echo -n $"Reloading $prog: "
    killproc $redis -HUP
    RETVAL=$?
    echo
}
  
force_reload() {
    restart
}
  
rh_status() {
    status $prog
}
  
rh_status_q() {
    rh_status >/dev/null 2>&1
}
  
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload}"
        exit 2
esac
 
执行权限：
[root@redis-master src]# chmod 755 /etc/init.d/redis-sentinel
```

###  **4、配置redis.conf** 

```shell
a）编辑redis-master主节点的redis.conf文件
[root@redis-master src]# mkdir -p /usr/local/redis/data/redis
[root@redis-master src]# cp /usr/local/redis/etc/redis.conf /usr/local/redis/etc/redis.conf.bak
[root@redis-master src]# vim /usr/local/redis/etc/redis.conf
bind 0.0.0.0
daemonize yes
pidfile "/usr/local/redis/var/redis-server.pid"
port 6379
tcp-backlog 128
timeout 0
tcp-keepalive 0
loglevel notice
logfile "/usr/local/redis/var/redis-server.log"
databases 16
save 900 1  
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir "/usr/local/redis/data/redis"
#masterauth "20180408"                        #master设置密码保护，即slave连接master时的密码
#requirepass "20180408"                       #设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes                                #打开aof持久化
appendfilename "appendonly.aof"
appendfsync everysec                          # 每秒一次aof写
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
 
注意：
上面配置中masterauth和requirepass表示设置密码保护，如果设置了密码，则连接redis后需要执行"auth 20180408"密码后才能操作其他命令。这里我不设置密码。
 
b）编辑redis-slave01和redis-slave02两个从节点的redis.conf文件
[root@redis-slave01 src]# mkdir -p /usr/local/redis/data/redis
[root@redis-slave01 src]# cp /usr/local/redis/etc/redis.conf /usr/local/redis/etc/redis.conf.bak
[root@redis-slave01 src]# vim /usr/local/redis/etc/redis.conf
bind 0.0.0.0
daemonize yes
pidfile "/usr/local/redis/var/redis-server.pid"
port 6379
tcp-backlog 128
timeout 0
tcp-keepalive 0
loglevel notice
logfile "/usr/local/redis/var/redis-server.log"
databases 16
save 900 1  
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir "/usr/local/redis/data/redis"
#masterauth "20180408"               
#requirepass "20180408"      
slaveof 192.168.10.202 6379                  #相对主redis配置，多添加了此行       
slave-serve-stale-data yes
slave-read-only yes                          #从节点只读，不能写入
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes                           
appendfilename "appendonly.aof"
appendfsync everysec                        
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

###  5、配置sentinel.conf（这个默认没有，需要自建）。三个节点的配置一样。

```shell
[root@redis-master src]# mkdir -p /usr/local/redis/data/sentinel
[root@redis-master src]# vim /usr/local/redis/etc/sentinel.conf
port 26379
pidfile "/usr/local/redis/var/redis-sentinel.pid"
dir "/usr/local/redis/data/sentinel"
daemonize yes
protected-mode no
logfile "/usr/local/redis/var/redis-sentinel.log"
sentinel monitor redisMaster 192.168.10.202 6379 2 
sentinel down-after-milliseconds redisMaster 10000 
sentinel parallel-syncs redisMaster 1
sentinel failover-timeout redisMaster 60000  
```

###  **6、启动redis和sentinel（三个节点都要操作）** 

```shell
设置系统变量
[root@redis-slave02 src]# vim /etc/profile
.......
export PATH=$PATH:/usr/local/redis/bin
[root@redis-slave02 src]# source /etc/profile
 
启动redis和sentinel
[root@redis-master src]# /etc/init.d/redis-server start
Starting redis-server:                                     [  OK  ]
[root@redis-master src]# /etc/init.d/redis-sentinel start
Starting redis-sentinel:                                   [  OK  ]
```

### 7、基本命令

```shell
1）查看三个节点的redis的主从关系
[root@redis-master src]# redis-cli -h 192.168.10.202 -p 6379 INFO|grep role
role:master
[root@redis-master src]# redis-cli -h 192.168.10.203 -p 6379 INFO|grep role
role:slave
[root@redis-master src]# redis-cli -h 192.168.10.205 -p 6379 INFO|grep role
role:slave
  
从上面信息可以看出，192.168.10.202是master，192.168.10.203和192.168.10.205是slave
  
2）查看Master节点信息：
[root@redis-master src]# redis-cli -h 192.168.10.202 -p 6379 info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.10.203,port=6379,state=online,offset=61480,lag=0
slave1:ip=192.168.10.205,port=6379,state=online,offset=61480,lag=0
master_replid:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:61626
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:61626
  
从上面信息看出，此时192.168.10.202的角色为master，有两个slave（203和205）被连接成功.
  
此时打开master的sentinel.conf，在末尾可看到如下自动写入的内容：
[root@redis-master src]# cat /usr/local/redis/etc/sentinel.conf
port 26379
pidfile "/usr/local/redis/var/redis-sentinel.pid"
dir "/usr/local/redis/data/sentinel"
daemonize yes
protected-mode no
logfile "/usr/local/redis/var/redis-sentinel.log"
sentinel myid c165761901b5ea3cd2d622bbf13f4c99eb73c1bc
sentinel monitor redisMaster 192.168.10.202 6379 2
sentinel down-after-milliseconds redisMaster 10000
sentinel failover-timeout redisMaster 60000
# Generated by CONFIG REWRITE
sentinel config-epoch redisMaster 0
sentinel leader-epoch redisMaster 0
sentinel known-slave redisMaster 192.168.10.203 6379
sentinel known-slave redisMaster 192.168.10.205 6379
sentinel known-sentinel redisMaster 192.168.10.205 26379 cc25d5f0e37803e888732d63deae3761c9f91e1d
sentinel known-sentinel redisMaster 192.168.10.203 26379 e1505ffc65f787871febfde2f27b762f70cddd71
sentinel current-epoch 0
  
[root@redis-master ~]# redis-cli -h 192.168.10.205 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=redisMaster,status=ok,address=192.168.10.202:6379,slaves=2,sentinels=3
 
3）查看Slave节点信息：
  
先查看salve01节点信息
[root@redis-slave01 src]# redis-cli -h 192.168.10.203 -p 6379 info Replication
# Replication
role:slave
master_host:192.168.10.202
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:96744
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:96744
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:96744
  
此时192.168.10.203的角色为slave，它们所属的master为220。
此时打开slave的sentinel.conf，在末尾可看到如下自动写入的内容：
[root@redis-slave01 src]# cat /usr/local/redis/etc/sentinel.conf
port 26379
pidfile "/usr/local/redis/var/redis-sentinel.pid"
dir "/usr/local/redis/data/sentinel"
daemonize yes
protected-mode no
logfile "/usr/local/redis/var/redis-sentinel.log"
sentinel myid e1505ffc65f787871febfde2f27b762f70cddd71
sentinel monitor redisMaster 192.168.10.202 6379 2
sentinel down-after-milliseconds redisMaster 10000
sentinel failover-timeout redisMaster 60000
# Generated by CONFIG REWRITE
sentinel config-epoch redisMaster 0
sentinel leader-epoch redisMaster 0
sentinel known-slave redisMaster 192.168.10.203 6379
sentinel known-slave redisMaster 192.168.10.205 6379
sentinel known-sentinel redisMaster 192.168.10.205 26379 cc25d5f0e37803e888732d63deae3761c9f91e1d
sentinel known-sentinel redisMaster 192.168.10.202 26379 c165761901b5ea3cd2d622bbf13f4c99eb73c1bc
sentinel current-epoch 0
 
[root@redis-slave01 ~]# redis-cli -h 192.168.10.203 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=redisMaster,status=ok,address=192.168.10.202:6379,slaves=2,sentinels=3
 
同样查看slave02节点的信息
[root@redis-slave02 src]# redis-cli -h 192.168.10.205 -p 6379 info Replication
# Replication
role:slave
master_host:192.168.10.202
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:99678
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:99678
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:170
repl_backlog_histlen:99509
  
[root@redis-slave02 src]# cat /usr/local/redis/etc/sentinel.conf
port 26379
pidfile "/usr/local/redis/var/redis-sentinel.pid"
dir "/usr/local/redis/data/sentinel"
daemonize yes
protected-mode no
logfile "/usr/local/redis/var/redis-sentinel.log"
sentinel myid cc25d5f0e37803e888732d63deae3761c9f91e1d
sentinel monitor redisMaster 192.168.10.202 6379 2
sentinel down-after-milliseconds redisMaster 10000
sentinel failover-timeout redisMaster 60000
# Generated by CONFIG REWRITE
sentinel config-epoch redisMaster 0
sentinel leader-epoch redisMaster 0
sentinel known-slave redisMaster 192.168.10.205 6379
sentinel known-slave redisMaster 192.168.10.203 6379
sentinel known-sentinel redisMaster 192.168.10.203 26379 e1505ffc65f787871febfde2f27b762f70cddd71
sentinel known-sentinel redisMaster 192.168.10.202 26379 c165761901b5ea3cd2d622bbf13f4c99eb73c1bc
sentinel current-epoch 0
 
[root@redis-slave02 ~]# redis-cli -h 192.168.10.205 -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=redisMaster,status=ok,address=192.168.10.202:6379,slaves=2,sentinels=3
```

###  **8、模拟故障（通过sentinel实现主从切换，sentinel也要部署多台，即集群模式，防止单台sentinel挂掉情况）** 

```shell
1）关掉任意一个slave节点（比如关闭掉slave01节点），所有节点的sentinel都可以检测到，出现如下示例信息：
[root@redis-master src]# redis-cli  -h 192.168.10.203 -p 6379
192.168.10.203:6379> get name
"kevin;"
192.168.10.203:6379> set name grace;
(error) READONLY You can't write against a read only slave.
192.168.10.203:6379> shutdown                             
not connected>
 
说明：shutdown命令表示关闭redis
 
从上可看出203被sentinel检测到已处于关闭状态，此时再来查看剩余节点的主从信息，它们的角色不会发生变化，只是master上的connected_slaves变为了1。
[root@redis-master src]# redis-cli  -h 192.168.10.202 -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.10.205,port=6379,state=online,offset=219376,lag=1
master_replid:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:219376
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:219376
 
查看sentinel日志（任意节点上查看），发现203节点已经进入"+sdown"状态
[root@redis-master src]# tail -f /usr/local/redis/var/redis-sentinel.log
2315:X 08 May 18:49:51.429 * Increased maximum number of open files to 10032 (it was originally set to 1024).
2315:X 08 May 18:49:51.431 * Running mode=sentinel, port=26379.
2315:X 08 May 18:49:51.431 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2315:X 08 May 18:49:51.463 # Sentinel ID is c165761901b5ea3cd2d622bbf13f4c99eb73c1bc
2315:X 08 May 18:49:51.463 # +monitor master redisMaster 192.168.10.202 6379 quorum 2
2315:X 08 May 18:50:11.544 * +slave slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:11.574 * +slave slave 192.168.10.205:6379 192.168.10.205 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:15.088 * +sentinel sentinel e1505ffc65f787871febfde2f27b762f70cddd71 192.168.10.203 26379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:16.075 * +sentinel sentinel cc25d5f0e37803e888732d63deae3761c9f91e1d 192.168.10.205 26379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:06:07.669 # +sdown slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
 
然后重启上面被关闭的slave节点（即192.168.10.203），所有节点的sentinel都可以检测到，可看出221又被sentinel检测到已处于可用状态，此时再来查看节点的主从信息，
它们的角色仍然不会发生变化，master上的connected_slaves又变为了2
[root@redis-slave01 src]# /etc/init.d/redis-server restart
Stopping redis-server:                                     [FAILED]
Starting redis-server:                                     [  OK  ]
[root@redis-slave01 src]# /etc/init.d/redis-server restart
Stopping redis-server:                                     [  OK  ]
Starting redis-server:                                     [  OK  ]
 
[root@redis-master src]# redis-cli  -h 192.168.10.202 -p 6379 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.10.205,port=6379,state=online,offset=268216,lag=0
slave1:ip=192.168.10.203,port=6379,state=online,offset=268070,lag=1
master_replid:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:268216
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:268216
 
查看sentinel日志（任意节点上查看），发现203节点已经进入"-sdown"状态
[root@redis-master src]# tail -f /usr/local/redis/var/redis-sentinel.log
2315:X 08 May 18:49:51.429 * Increased maximum number of open files to 10032 (it was originally set to 1024).
2315:X 08 May 18:49:51.431 * Running mode=sentinel, port=26379.
2315:X 08 May 18:49:51.431 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
2315:X 08 May 18:49:51.463 # Sentinel ID is c165761901b5ea3cd2d622bbf13f4c99eb73c1bc
2315:X 08 May 18:49:51.463 # +monitor master redisMaster 192.168.10.202 6379 quorum 2
2315:X 08 May 18:50:11.544 * +slave slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:11.574 * +slave slave 192.168.10.205:6379 192.168.10.205 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:15.088 * +sentinel sentinel e1505ffc65f787871febfde2f27b762f70cddd71 192.168.10.203 26379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 18:50:16.075 * +sentinel sentinel cc25d5f0e37803e888732d63deae3761c9f91e1d 192.168.10.205 26379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:06:07.669 # +sdown slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:10:14.965 * +reboot slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:10:15.020 # -sdown slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
 
=======================================================================================================
2）关掉master节点（即192.168.10.202），待所有节点的sentinel都检测到后（稍等一会，2-3秒钟时间），再来查看两个Slave节点的主从信息，发现其中一个节点的角色通过选举后会成为
master节点了！
[root@redis-master src]# redis-cli  -h 192.168.10.202 -p 6379
192.168.10.202:6379> shutdown
not connected>
 
查看sentinel日志（任意节点上查看），发现202节点已经进入"+sdown"状态
[root@redis-master src]# tail -f /usr/local/redis/var/redis-sentinel.log
2315:X 08 May 19:17:03.722 # +failover-state-reconf-slaves master redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:03.760 * +slave-reconf-sent slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:04.015 * +slave-reconf-inprog slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:04.459 # -odown master redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:05.047 * +slave-reconf-done slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:05.131 # +failover-end master redisMaster 192.168.10.202 6379
2315:X 08 May 19:17:05.131 # +switch-master redisMaster 192.168.10.202 6379 192.168.10.205 6379
2315:X 08 May 19:17:05.131 * +slave slave 192.168.10.203:6379 192.168.10.203 6379 @ redisMaster 192.168.10.205 6379
2315:X 08 May 19:17:05.131 * +slave slave 192.168.10.202:6379 192.168.10.202 6379 @ redisMaster 192.168.10.205 6379
2315:X 08 May 19:17:15.170 # +sdown slave 192.168.10.202:6379 192.168.10.202 6379 @ redisMaster 192.168.10.205 6379
 
[root@redis-slave01 src]# redis-cli -h 192.168.10.203 -p 6379 INFO|grep role
role:slave
[root@redis-slave01 src]# redis-cli -h 192.168.10.205 -p 6379 INFO|grep role     
role:master
 
[root@redis-slave01 src]# redis-cli  -h 192.168.10.205 -p 6379 info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.10.203,port=6379,state=online,offset=348860,lag=1
master_replid:a51f87958cea0b1e8fe2c83542ca6cebace53bf7
master_replid2:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_repl_offset:349152
second_repl_offset:342824
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:170
repl_backlog_histlen:348983
 
[root@redis-slave01 src]# redis-cli  -h 192.168.10.203 -p 6379 info replication
# Replication
role:slave
master_host:192.168.10.205
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:347546
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:a51f87958cea0b1e8fe2c83542ca6cebace53bf7
master_replid2:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_repl_offset:347546
second_repl_offset:342824
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:283207
repl_backlog_histlen:64340
 
由上可知，当master节点（即192.168.10.202）的redis关闭后，slave02节点（即192.168.10.205）变成了新的master节点，而slave01（即192.168.10.203）成为
了slave02的从节点！
 
192.168.10.205节点此时被选举为master，此时打开的205节点的redis.conf文件，slaveof配置项已被自动删除了。
而203从节点的redis.conf文件中slaveof配置项的值被自动修改为192.168.10.205 6379。
[root@redis-slave02 src]# cat /usr/local/redis/etc/redis.conf|grep slaveof
[root@redis-slave01 src]# cat /usr/local/redis/etc/redis.conf|grep slaveof
slaveof 192.168.10.205 6379
 
在这个新master上（即192.168.10.205）执行诸如set这样的写入操作将被成功执行
[root@redis-slave01 src]# redis-cli  -h 192.168.10.205 -p 6379
192.168.10.205:6379> set name beijing;
OK
 
[root@redis-master src]# redis-cli  -h 192.168.10.203 -p 6379
192.168.10.203:6379> get name
"beijing;"
192.168.10.203:6379> set name tianjin;
(error) READONLY You can't write against a read only slave.
192.168.10.203:6379>
 
重启192.168.10.202节点的redis，待所有节点的sentinel都检测到后，再来查看所有节点的主从信息，此时192.168.10.205节点的master角色不会被重新抢占，
而192.168.10.202节点的角色会从原来的master变为了slave。
[root@redis-master src]# /etc/init.d/redis-server restart
Stopping redis-server:                                     [FAILED]
Starting redis-server:                                     [  OK  ]
[root@redis-master src]# /etc/init.d/redis-server restart
Stopping redis-server:                                     [  OK  ]
Starting redis-server:                                     [  OK  ]
 
[root@redis-master src]# redis-cli -h 192.168.10.202 -p 6379 INFO|grep role
role:slave
[root@redis-master src]# redis-cli -h 192.168.10.203 -p 6379 INFO|grep role
role:slave
[root@redis-master src]# redis-cli -h 192.168.10.205 -p 6379 INFO|grep role
role:master
[root@redis-master src]# redis-cli  -h 192.168.10.205 -p 6379 info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.10.203,port=6379,state=online,offset=545410,lag=1
slave1:ip=192.168.10.202,port=6379,state=online,offset=545410,lag=1
master_replid:a51f87958cea0b1e8fe2c83542ca6cebace53bf7
master_replid2:96a1fd63d0ad9e7903851a82382e32d690667bcc
master_repl_offset:545556
second_repl_offset:342824
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:170
repl_backlog_histlen:545387
 
[root@redis-master src]# redis-cli  -h 192.168.10.202 -p 6379
192.168.10.202:6379> get name
"beijing;"
 
此时登录192.168.10.202节点的redis，执行"get name"得到的值为beijing，而不是原来的kevin，因为192.168.10.202节点的redis重启后会自动从新的master中同步
数据。此时打开192.168.10.202节点的redis.conf文件，会在末尾找到如下信息：
[root@redis-master src]# cat /usr/local/redis/etc/redis.conf|grep slaveof
slaveof 192.168.10.205 6379
 
到此，已经验证出了redis sentinel可以自行实现主从的故障切换了！
```

