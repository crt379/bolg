---
title: Debian安装Redis
date: 2024-05-04 21:12:41
tags:
- Redis
- Linux
category: Linux
---

## 更新系统

```shell
apt update
apt upgrade
```

## 安装redis

```shell
apt install redis-server
```

## 配置redis

```shell
vim /etc/redis/redis.conf 
```

修改
```conf
bind 0.0.0.0 -::1
requirepass 123456 # 123456 为密码
```

## 重启

```shell
systemctl restart redis
```

## 验证

```shell
root@server-4:~# redis-cli -h 192.168.31.207
192.168.31.207:6379> set a hhhh
(error) NOAUTH Authentication required.
192.168.31.207:6379> AUTH xxxx
OK
192.168.31.207:6379> set a hhhh
OK
192.168.31.207:6379> get a
"hhhh"
192.168.31.207:6379> del a
(integer) 1
192.168.31.207:6379> get a
(nil)
```


## 主从配置

### 主节点配置

```shell
vim /etc/redis/redis.conf 
```

修改
```conf
masterauth 123456  # master 设置密码
appendonly yes  # 开启appendonly 模式后,redis 将每一次写操作请求都追加到appendonly.aof 文件中
```

重启
```shell
systemctl restart redis
```

### 从节点

```shell
vim /etc/redis/redis.conf 
```

修改
```conf
replicaof 192.168.31.207 6379
masterauth 123456
appendonly yes
```

重启
```shell
systemctl restart redis
```

### 验证

- 主
```log
root@server-3:/home/crt# redis-cli
127.0.0.1:6379> auth 123456
OK
127.0.0.1:6379> 
127.0.0.1:6379> set test test
OK
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.31.210,port=6379,state=online,offset=196,lag=0
master_failover_state:no-failover
master_replid:af9a80775bdef4a73d0af5ba5659dab3001851e2
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:196
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:196
127.0.0.1:6379> 
127.0.0.1:6379> del test
(integer) 1
127.0.0.1:6379> 
```

- 从
```
root@server-4:~# redis-cli
127.0.0.1:6379> key test
(error) ERR unknown command 'key', with args beginning with: 'test' 
127.0.0.1:6379> auth 123456
(error) ERR AUTH <password> called without any password configured for the default user. Are you sure your configuration is correct?
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> key test
(error) ERR unknown command 'key', with args beginning with: 'test' 
127.0.0.1:6379> key
(error) ERR unknown command 'key', with args beginning with: 
127.0.0.1:6379> key *
(error) ERR unknown command 'key', with args beginning with: '*' 
127.0.0.1:6379> get test
"test"
127.0.0.1:6379> 
127.0.0.1:6379> get *
(nil)
127.0.0.1:6379> 
```