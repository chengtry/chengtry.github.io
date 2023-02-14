# Redis基础

## Redis的安装：

### 安装redis

```shell
#下载redis的安装包并用Xftp上传到服务器，用tar命令解压安装包
1. tar -xzvf redis-6.0.6.tar.gz
#进入redis安装目录使用make命令
2. cd redis-6.0.6
   make
```

### 安装过程中出现的问题

```shell
1.#使用make命令发生错误，查明原因发现未安装gcc，使用以下命令
yum install cpp
yum install binutils
yum install glibc
yum install glibc-kernheaders
yum install glibc-common
yum install glibc-devel
yum install gcc
yum install make
yum -y install gcc-c++

2.#再次使用make命令还是报错，查明原因发现gcc版本太低（只针对redis6.0版本及以上）
#redis6.0版本以上需要将gcc升级到5.3以上，而CentOS 7默认安装的gcc版本是4.8.5
gcc -v #查看gcc版本号
#升级gcc到 5.3及以上版本
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
#再次使用make命令（可以先重新安装redis-->先删除解压好的redis-->用tar -xzvf 命令重新解压）

3.#redis启动出现三个警告
第一个警告：The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.

第二个警告：overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to/etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.

第三个警告：you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix thisissue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain thesetting after a reboot. Redis must be restarted after THP is disabled.

#解决方案
第一个警告
将net.core.somaxconn = 1024添加到/etc/sysctl.conf中，然后执行sysctl -p生效配置。

第二个警告
将vm.overcommit_memory = 1添加到/etc/sysctl.conf中，然后执行sysctl -p生效配置。

第三个警告
将echo never > /sys/kernel/mm/transparent_hugepage/enabled添加到/etc/rc.local中，然后执行source /etc/rc.local生效配置。

重新启动redis正常
```

## Redis中文乱码：

### 解决方案：启动时加上--raw

```java
redis-cli -p 6379 --raw
```

## 查看Redis安装目录：

### 用whereis和which命令找不到安装目录可以使用以下方法：

```java
第一步：ps -ef|grep redis       得到进程号信息
第二步：ls -l /proc/进程号/cwd   得到安装目录信息
```

![image-20210310095730403](F:\images\Typora\image-20210310095730403.png)

## 查看Redis版本号：

### 1. redis-server --version

### 2. redis-cli -v

![image-20210310101418280](F:\images\Typora\image-20210310101418280.png)

### 3. 启动redis后，用info命令

![image-20210310101443498](F:\images\Typora\image-20210310101443498.png)

## Redis启动、停止、重启、设置密码、进入

### Redis启动:

```java
启动redis服务
redis-server redis.conf
```

### Redis关闭：

```java
redis-cli -h 127.0.0.1 -p 6379 shutdown
redis-cli shutdown
```

### Redis重启：

```java
使用关闭+启动组合命令：redis-cli shutdown 、 redis-server redis.conf
```

### Redis设置密码：

```java
1. 修改配置文件，永久生效(修改后需要重启redis才能生效)
    进入redis.conf配置文件，添加 requirepass redis123 (密码：redis123)

2. 临时添加密码(重启redis密码失效)
    config set requirepass redis123 
```

### Redis进入

```java
使用密码进入并解决中文乱码 
redis-cli -a redis123 --raw
```

## 为什么 Redis 是单线程还这么快？

```
Redis 所有的数据都是存放在内存中的，单线程的操作没有上下文的切换，从而达到很高的效率。
```

# 五种基本类型

## String类型

### 常见命令：

exists key：判断键是否存在

del key：删除键值对

move key db：将键值对移动到指定数据库

expire key second：设置键值对的过期时间

type key：查看value的数据类型

### TTL命令：

Redis的key，通过TTL命令返回key的过期时间，一般来说有3种：

1.当前key没有设置过期时间，所以会返回-1.

2.当前key有设置过期时间，而且key已经过期，所以会返回-2.

3.当前key有设置过期时间，且key还没有过期，故会返回key的正常剩余时间.

### 设置和取消过期时间

```shell
#设置 name 的过期时间为6000秒
expire name 6000

#取消 name 的过期时间
persist name

#查看 name 过期时间
ttl name
```

### rename 和 renamex 命令：

`RENAME key newkey` 修改 key 的名称

`RENAMENX key newkey` 仅当 newkey 不存在时，将 key 改名为 newkey 

### 命令代码：

```java
127.0.0.1:6379> keys * # 查看当前数据库所有key
(empty list or set)
127.0.0.1:6379> set name qinjiang # set key
OK
127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> keys *
1) "age"
2) "name"
127.0.0.1:6379> move age 1 # 将键值对移动到指定数据库
(integer) 1
127.0.0.1:6379> EXISTS age # 判断键是否存在
(integer) 0 # 不存在
127.0.0.1:6379> EXISTS name
(integer) 1 # 存在
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> keys *
1) "age"
127.0.0.1:6379[1]> del age # 删除键值对
(integer) 1 # 删除个数


127.0.0.1:6379> set age 20
OK
127.0.0.1:6379> EXPIRE age 15 # 设置键值对的过期时间

(integer) 1 # 设置成功 开始计数
127.0.0.1:6379> ttl age # 查看key的过期剩余时间
(integer) 13
127.0.0.1:6379> ttl age
(integer) 11
127.0.0.1:6379> ttl age
(integer) 9
127.0.0.1:6379> ttl age
(integer) -2 # -2 表示key过期，-1表示key未设置过期时间

127.0.0.1:6379> get age # 过期的key 会被自动delete
(nil)
127.0.0.1:6379> keys *
1) "name"

127.0.0.1:6379> type name # 查看value的数据类型
string
```

## List类型

### 命令代码：

```java
---------------------------LPUSH---RPUSH---LRANGE--------------------------------

127.0.0.1:6379> LPUSH mylist k1 # LPUSH mylist=>{1}
(integer) 1
127.0.0.1:6379> LPUSH mylist k2 # LPUSH mylist=>{2,1}
(integer) 2
127.0.0.1:6379> RPUSH mylist k3 # RPUSH mylist=>{2,1,3}
(integer) 3
127.0.0.1:6379> get mylist # 普通的get是无法获取list值的
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> LRANGE mylist 0 4 # LRANGE 获取起止位置范围内的元素
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> LRANGE mylist 0 2
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379> LRANGE mylist 0 1
1) "k2"
2) "k1"
127.0.0.1:6379> LRANGE mylist 0 -1 # 获取全部元素
1) "k2"
2) "k1"
3) "k3"

---------------------------LPUSHX---RPUSHX-----------------------------------

127.0.0.1:6379> LPUSHX list v1 # list不存在 LPUSHX失败
(integer) 0
127.0.0.1:6379> LPUSHX list v1 v2  
(integer) 0
127.0.0.1:6379> LPUSHX mylist k4 k5 # 向mylist中 左边 PUSH k4 k5
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1
1) "k5"
2) "k4"
3) "k2"
4) "k1"
5) "k3"

---------------------------LINSERT--LLEN--LINDEX--LSET----------------------------

127.0.0.1:6379> LINSERT mylist after k2 ins_key1 # 在k2元素后 插入ins_key1
(integer) 6
127.0.0.1:6379> LRANGE mylist 0 -1
1) "k5"
2) "k4"
3) "k2"
4) "ins_key1"
5) "k1"
6) "k3"
127.0.0.1:6379> LLEN mylist # 查看mylist的长度
(integer) 6
127.0.0.1:6379> LINDEX mylist 3 # 获取下标为3的元素
"ins_key1"
127.0.0.1:6379> LINDEX mylist 0
"k5"
127.0.0.1:6379> LSET mylist 3 k6 # 将下标3的元素 set值为k6
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "k5"
2) "k4"
3) "k2"
4) "k6"
5) "k1"
6) "k3"

---------------------------LPOP--RPOP--------------------------

127.0.0.1:6379> LPOP mylist # 左侧(头部)弹出
"k5"
127.0.0.1:6379> RPOP mylist # 右侧(尾部)弹出
"k3"

---------------------------RPOPLPUSH--------------------------

127.0.0.1:6379> LRANGE mylist 0 -1
1) "k4"
2) "k2"
3) "k6"
4) "k1"
127.0.0.1:6379> RPOPLPUSH mylist newlist # 将mylist的最后一个值(k1)弹出，加入到newlist的头部
"k1"
127.0.0.1:6379> LRANGE newlist 0 -1
1) "k1"
127.0.0.1:6379> LRANGE mylist 0 -1
1) "k4"
2) "k2"
3) "k6"

---------------------------LTRIM--------------------------

127.0.0.1:6379> LTRIM mylist 0 1 # 截取mylist中的 0~1部分
OK
127.0.0.1:6379> LRANGE mylist 0 -1
1) "k4"
2) "k2"

# 初始 mylist: k2,k2,k2,k2,k2,k2,k4,k2,k2,k2,k2
---------------------------LREM--------------------------

127.0.0.1:6379> LREM mylist 3 k2 # 从头部开始搜索 至多删除3个 k2
(integer) 3
# 删除后：mylist: k2,k2,k2,k4,k2,k2,k2,k2

127.0.0.1:6379> LREM mylist -2 k2 #从尾部开始搜索 至多删除2个 k2
(integer) 2
# 删除后：mylist: k2,k2,k2,k4,k2,k2


---------------------------BLPOP--BRPOP--------------------------

mylist: k2,k2,k2,k4,k2,k2
newlist: k1

127.0.0.1:6379> BLPOP newlist mylist 30 # 从newlist中弹出第一个值，mylist作为候选
1) "newlist" # 弹出
2) "k1"
127.0.0.1:6379> BLPOP newlist mylist 30
1) "mylist" # 由于newlist空了 从mylist中弹出
2) "k2"
127.0.0.1:6379> BLPOP newlist 30
(30.10s) # 超时了

127.0.0.1:6379> BLPOP newlist 30 # 我们连接另一个客户端向newlist中push了test, 阻塞被解决。
1) "newlist"
2) "test"
(12.54s)
```

## Set类型

### 命令代码：

```java
---------------SADD--SCARD--SMEMBERS--SISMEMBER--------------------

127.0.0.1:6379> SADD myset m1 m2 m3 m4 # 向myset中增加成员 m1~m4
(integer) 4
127.0.0.1:6379> SCARD myset # 获取集合的成员数目
(integer) 4
127.0.0.1:6379> smembers myset # 获取集合中所有成员
1) "m4"
2) "m3"
3) "m2"
4) "m1"
127.0.0.1:6379> SISMEMBER myset m5 # 查询m5是否是myset的成员
(integer) 0 # 不是，返回0
127.0.0.1:6379> SISMEMBER myset m2
(integer) 1 # 是，返回1
127.0.0.1:6379> SISMEMBER myset m3
(integer) 1

---------------------SRANDMEMBER--SPOP----------------------------------

127.0.0.1:6379> SRANDMEMBER myset 3 # 随机返回3个成员
1) "m2"
2) "m3"
3) "m4"
127.0.0.1:6379> SRANDMEMBER myset # 随机返回1个成员
"m3"
127.0.0.1:6379> SPOP myset 2 # 随机移除并返回2个成员
1) "m1"
2) "m4"
# 将set还原到{m1,m2,m3,m4}

---------------------SMOVE--SREM----------------------------------------

127.0.0.1:6379> SMOVE myset newset m3 # 将myset中m3成员移动到newset集合
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "m4"
2) "m2"
3) "m1"
127.0.0.1:6379> SMEMBERS newset
1) "m3"
127.0.0.1:6379> SREM newset m3 # 从newset中移除m3元素
(integer) 1
127.0.0.1:6379> SMEMBERS newset
(empty list or set)

# 下面开始是多集合操作,多集合操作中若只有一个参数默认和自身进行运算
# setx=>{m1,m2,m4,m6}, sety=>{m2,m5,m6}, setz=>{m1,m3,m6}

-----------------------------SDIFF------------------------------------

127.0.0.1:6379> SDIFF setx sety setz # 等价于setx-sety-setz
1) "m4"
127.0.0.1:6379> SDIFF setx sety # setx - sety
1) "m4"
2) "m1"
127.0.0.1:6379> SDIFF sety setx # sety - setx
1) "m5"


-------------------------SINTER---------------------------------------
# 共同关注（交集）

127.0.0.1:6379> SINTER setx sety setz # 求 setx、sety、setx的交集
1) "m6"
127.0.0.1:6379> SINTER setx sety # 求setx sety的交集
1) "m2"
2) "m6"

-------------------------SUNION---------------------------------------

127.0.0.1:6379> SUNION setx sety setz # setx sety setz的并集
1) "m4"
2) "m6"
3) "m3"
4) "m2"
5) "m1"
6) "m5"
127.0.0.1:6379> SUNION setx sety # setx sety 并集
1) "m4"
2) "m6"
3) "m2"
4) "m1"
5) "m5"
```

## Hash类型

### 命令代码：

```java
------------------------HSET--HMSET--HSETNX----------------
127.0.0.1:6379> HSET studentx name sakura # 将studentx哈希表作为一个对象，设置name为sakura
(integer) 1
127.0.0.1:6379> HSET studentx name gyc # 重复设置field进行覆盖，并返回0
(integer) 0
127.0.0.1:6379> HSET studentx age 20 # 设置studentx的age为20
(integer) 1
127.0.0.1:6379> HMSET studentx sex 1 tel 15623667886 # 设置sex为1，tel为15623667886
OK
127.0.0.1:6379> HSETNX studentx name gyc # HSETNX 设置已存在的field
(integer) 0 # 失败
127.0.0.1:6379> HSETNX studentx email 12345@qq.com
(integer) 1 # 成功

----------------------HEXISTS--------------------------------
127.0.0.1:6379> HEXISTS studentx name # name字段在studentx中是否存在
(integer) 1 # 存在
127.0.0.1:6379> HEXISTS studentx addr
(integer) 0 # 不存在

-------------------HGET--HMGET--HGETALL-----------
127.0.0.1:6379> HGET studentx name # 获取studentx中name字段的value
"gyc"
127.0.0.1:6379> HMGET studentx name age tel # 获取studentx中name、age、tel字段的value
1) "gyc"
2) "20"
3) "15623667886"
127.0.0.1:6379> HGETALL studentx # 获取studentx中所有的field及其value
 1) "name"
 2) "gyc"
 3) "age"
 4) "20"
 5) "sex"
 6) "1"
 7) "tel"
 8) "15623667886"
 9) "email"
10) "12345@qq.com"


--------------------HKEYS--HLEN--HVALS--------------
127.0.0.1:6379> HKEYS studentx # 查看studentx中所有的field
1) "name"
2) "age"
3) "sex"
4) "tel"
5) "email"
127.0.0.1:6379> HLEN studentx # 查看studentx中的字段数量
(integer) 5
127.0.0.1:6379> HVALS studentx # 查看studentx中所有的value
1) "gyc"
2) "20"
3) "1"
4) "15623667886"
5) "12345@qq.com"

-------------------------HDEL--------------------------
127.0.0.1:6379> HDEL studentx sex tel # 删除studentx 中的sex、tel字段
(integer) 2
127.0.0.1:6379> HKEYS studentx
1) "name"
2) "age"
3) "email"

-------------HINCRBY--HINCRBYFLOAT------------------------
127.0.0.1:6379> HINCRBY studentx age 1 # studentx的age字段数值+1
(integer) 21
127.0.0.1:6379> HINCRBY studentx name 1 # 非整数字型字段不可用
(error) ERR hash value is not an integer
127.0.0.1:6379> HINCRBYFLOAT studentx weight 0.6 # weight字段增加0.6
"90.8"
```

## Zset类型

### 命令代码:

```java
-------------------ZADD--ZCARD--ZCOUNT--------------
127.0.0.1:6379> ZADD myzset 1 m1 2 m2 3 m3 # 向有序集合myzset中添加成员m1 score=1 以及成员m2 score=2..
(integer) 2
127.0.0.1:6379> ZCARD myzset # 获取有序集合的成员数
(integer) 2
127.0.0.1:6379> ZCOUNT myzset 0 1 # 获取score在 [0,1]区间的成员数量
(integer) 1
127.0.0.1:6379> ZCOUNT myzset 0 2
(integer) 2

----------------ZINCRBY--ZSCORE--------------------------
127.0.0.1:6379> ZINCRBY myzset 5 m2 # 将成员m2的score +5
"7"
127.0.0.1:6379> ZSCORE myzset m1 # 获取成员m1的score
"1"
127.0.0.1:6379> ZSCORE myzset m2
"7"

--------------ZRANK--ZRANGE-----------------------------------
127.0.0.1:6379> ZRANK myzset m1 # 获取成员m1的索引，索引按照score排序，score相同索引值按字典顺序顺序增加
(integer) 0
127.0.0.1:6379> ZRANK myzset m2
(integer) 2
127.0.0.1:6379> ZRANGE myzset 0 1 # 获取索引在 0~1的成员
1) "m1"
2) "m3"
127.0.0.1:6379> ZRANGE myzset 0 -1 # 获取全部成员
1) "m1"
2) "m3"
3) "m2"

#testset=>{abc,add,amaze,apple,back,java,redis} score均为0
------------------ZRANGEBYLEX---------------------------------
127.0.0.1:6379> ZRANGEBYLEX testset - + # 返回所有成员
1) "abc"
2) "add"
3) "amaze"
4) "apple"
5) "back"
6) "java"
7) "redis"
127.0.0.1:6379> ZRANGEBYLEX testset - + LIMIT 0 3 # 分页 按索引显示查询结果的 0,1,2条记录
1) "abc"
2) "add"
3) "amaze"
127.0.0.1:6379> ZRANGEBYLEX testset - + LIMIT 3 3 # 显示 3,4,5条记录
1) "apple"
2) "back"
3) "java"
127.0.0.1:6379> ZRANGEBYLEX testset (- [apple # 显示 (-,apple] 区间内的成员
1) "abc"
2) "add"
3) "amaze"
4) "apple"
127.0.0.1:6379> ZRANGEBYLEX testset [apple [java # 显示 [apple,java]字典区间的成员
1) "apple"
2) "back"
3) "java"

-----------------------ZRANGEBYSCORE---------------------
127.0.0.1:6379> ZRANGEBYSCORE myzset 1 10 # 返回score在 [1,10]之间的的成员
1) "m1"
2) "m3"
3) "m2"
127.0.0.1:6379> ZRANGEBYSCORE myzset 1 5
1) "m1"
2) "m3"

--------------------ZLEXCOUNT-----------------------------
127.0.0.1:6379> ZLEXCOUNT testset - +
(integer) 7
127.0.0.1:6379> ZLEXCOUNT testset [apple [java
(integer) 3

------------------ZREM--ZREMRANGEBYLEX--ZREMRANGBYRANK--ZREMRANGEBYSCORE--------------------------------
127.0.0.1:6379> ZREM testset abc # 移除成员abc
(integer) 1
127.0.0.1:6379> ZREMRANGEBYLEX testset [apple [java # 移除字典区间[apple,java]中的所有成员
(integer) 3
127.0.0.1:6379> ZREMRANGEBYRANK testset 0 1 # 移除排名0~1的所有成员
(integer) 2
127.0.0.1:6379> ZREMRANGEBYSCORE myzset 0 3 # 移除score在 [0,3]的成员
(integer) 2


# testset=> {abc,add,apple,amaze,back,java,redis} score均为0
# myzset=> {(m1,1),(m2,2),(m3,3),(m4,4),(m7,7),(m9,9)}
----------------ZREVRANGE--ZREVRANGEBYSCORE--ZREVRANGEBYLEX-----------
127.0.0.1:6379> ZREVRANGE myzset 0 3 # 按score递减排序，然后按索引，返回结果的 0~3
1) "m9"
2) "m7"
3) "m4"
4) "m3"
127.0.0.1:6379> ZREVRANGE myzset 2 4 # 返回排序结果的 索引的2~4
1) "m4"
2) "m3"
3) "m2"
127.0.0.1:6379> ZREVRANGEBYSCORE myzset 6 2 # 按score递减顺序 返回集合中分数在[2,6]之间的成员
1) "m4"
2) "m3"
3) "m2"
127.0.0.1:6379> ZREVRANGEBYLEX testset [java (add # 按字典倒序 返回集合中(add,java]字典区间的成员
1) "java"
2) "back"
3) "apple"
4) "amaze"

-------------------------ZREVRANK------------------------------
127.0.0.1:6379> ZREVRANK myzset m7 # 按score递减顺序，返回成员m7索引
(integer) 1
127.0.0.1:6379> ZREVRANK myzset m2
(integer) 4


# mathscore=>{(xm,90),(xh,95),(xg,87)} 小明、小红、小刚的数学成绩
# enscore=>{(xm,70),(xh,93),(xg,90)} 小明、小红、小刚的英语成绩
-------------------ZINTERSTORE--ZUNIONSTORE-----------------------------------
127.0.0.1:6379> ZINTERSTORE sumscore 2 mathscore enscore # 将mathscore enscore进行合并 结果存放到sumscore
(integer) 3
127.0.0.1:6379> ZRANGE sumscore 0 -1 withscores # 合并后的score是之前集合中所有score的和
1) "xm"
2) "160"
3) "xg"
4) "177"
5) "xh"
6) "188"

127.0.0.1:6379> ZUNIONSTORE lowestscore 2 mathscore enscore AGGREGATE MIN # 取两个集合的成员score最小值作为结果的
(integer) 3
127.0.0.1:6379> ZRANGE lowestscore 0 -1 withscores
1) "xm"
2) "70"
3) "xg"
4) "87"
5) "xh"
6) "93"
```

# 三种特殊类型

## Geospatial(地理位置)

### 命令代码：

```java
----------------georadius---------------------
127.0.0.1:6379> GEORADIUS china:city 120 30 500 km withcoord withdist # 查询经纬度(120,30)坐标500km半径内的成员
1) 1) "hangzhou"
   2) "29.4151"
   3) 1) "120.20000249147415"
      2) "30.199999888333501"
2) 1) "shanghai"
   2) "205.3611"
   3) 1) "121.40000134706497"
      2) "31.400000253193539"

------------geohash---------------------------
127.0.0.1:6379> geohash china:city yichang shanghai # 获取成员经纬坐标的geohash表示
1) "wmrjwbr5250"
2) "wtw6ds0y300"
```

## Hyperloglog(基数统计)

### 命令代码：

```java
----------PFADD--PFCOUNT---------------------
127.0.0.1:6379> PFADD myelemx a b c d e f g h i j k # 添加元素
(integer) 1
127.0.0.1:6379> type myelemx # hyperloglog底层使用String
string
127.0.0.1:6379> PFCOUNT myelemx # 估算myelemx的基数
(integer) 11
127.0.0.1:6379> PFADD myelemy i j k z m c b v p q s
(integer) 1
127.0.0.1:6379> PFCOUNT myelemy
(integer) 11

----------------PFMERGE-----------------------
127.0.0.1:6379> PFMERGE myelemz myelemx myelemy # 合并myelemx和myelemy 成为myelemz
OK
127.0.0.1:6379> PFCOUNT myelemz # 估算基数
(integer) 17
```

## BitMap(位图)

### 命令代码：

```java
------------setbit--getbit--------------
127.0.0.1:6379> setbit sign 0 1 # 设置sign的第0位为 1 
(integer) 0
127.0.0.1:6379> setbit sign 2 1 # 设置sign的第2位为 1  不设置默认 是0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> type sign
string

127.0.0.1:6379> getbit sign 2 # 获取第2位的数值
(integer) 1
127.0.0.1:6379> getbit sign 3
(integer) 1
127.0.0.1:6379> getbit sign 4 # 未设置默认是0
(integer) 0

-----------bitcount----------------------------
127.0.0.1:6379> BITCOUNT sign # 统计sign中为1的位数
(integer) 4
```

# Redis事务

## 事务：

```java
Redis的单条命令是保证原子性的，但是redis事务不能保证原子性

Redis事务本质：一组命令的集合。

----------------- 队列 set set set 执行 -------------------

事务中每条命令都会被序列化，执行过程中按顺序执行，不允许其他命令进行干扰。

一次性
顺序性
排他性
Redis事务没有隔离级别的概念
Redis单条命令是保证原子性的，但是事务不保证原子性！
```

## Redis事务过程：

```java
开启事务（multi）
命令入队
执行事务（exec）

所以事务中的命令在加入时都没有被执行，直到提交时才会开始执行(Exec)一次性完成。

127.0.0.1:6379> multi # 开启事务
OK
127.0.0.1:6379> set k1 v1 # 命令入队
QUEUED
127.0.0.1:6379> set k2 v2 # ..
QUEUED
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> keys *
QUEUED
127.0.0.1:6379> exec # 事务执行
1) OK
2) OK
3) "v1"
4) OK
5) 1) "k3"
   2) "k2"
   3) "k1"
```

## 取消事务(DISCARD)：

```java
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> DISCARD # 放弃事务
OK
127.0.0.1:6379> EXEC 
(error) ERR EXEC without MULTI # 当前未开启事务
127.0.0.1:6379> get k1 # 被放弃事务中命令并未执行
(nil)
```

## 事务错误

### 1.代码语法错误（编译时异常）所有的命令都不执行

```java
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> error k1 # 这是一条语法错误命令
(error) ERR unknown command `error`, with args beginning with: `k1`, # 会报错但是不影响后续命令入队 
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors. # 执行报错
127.0.0.1:6379> get k1 
(nil) # 其他命令并没有被执行
```

### 2.代码逻辑错误 (运行时异常) **其他命令可以正常执行 ** >>> 所以不保证事务原子性

```java
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> INCR k1 # 这条命令逻辑错误（对字符串进行增量）
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) (error) ERR value is not an integer or out of range # 运行时报错
4) "v2" # 其他命令正常执行

# 虽然中间有一条命令报错了，但是后面的指令依旧正常执行成功了。
# 所以说Redis单条指令保证原子性，但是Redis事务不能保证原子性。
```

# 监控

## 乐观锁：

使用watch key监控指定数据，相当于乐观锁加锁。

### 1. 正常执行

```java
127.0.0.1:6379> set money 100 # 设置余额:100
OK
127.0.0.1:6379> set use 0 # 支出使用:0
OK
127.0.0.1:6379> watch money # 监视money (上锁)
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY use 20
QUEUED
127.0.0.1:6379> exec # 监视值没有被中途修改，事务正常执行
1) (integer) 80
2) (integer) 20
```

### 2.多线程测试

```java
线程1：

127.0.0.1:6379> watch money # money上锁
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> DECRBY money 20
QUEUED
127.0.0.1:6379> INCRBY use 20
QUEUED
127.0.0.1:6379>     # 此时事务并没有执行


模拟线程插队，线程2：
127.0.0.1:6379> INCRBY money 500 # 修改了线程一中监视的money
(integer) 600

回到线程1，执行事务：
127.0.0.1:6379> EXEC # 执行之前，另一个线程修改了我们的值，这个时候就会导致事务执行失败
(nil) # 没有结果，说明事务执行失败
127.0.0.1:6379> get money # 线程2 修改生效
"600"
127.0.0.1:6379> get use # 线程1事务执行失败，数值没有被修改
"0"


解锁获取最新值，然后再加锁进行事务。
unwatch进行解锁。
注意：每次提交执行exec后都会自动释放锁，不管是否成功
```

# 缓存穿透、缓存击穿、缓存雪崩

## 缓存穿透

### 概念：

在默认情况下，用户请求数据时，会先在缓存(Redis)中查找，若没找到即缓存未命中，再在数据库中进行查找，数量少可能问题不大，可是一旦大量的请求数据（例如秒杀场景）缓存都没有命中的话，就会全部转移到数据库上，造成数据库极大的压力，就有可能导致数据库崩溃。网络安全中也有人恶意使用这种手段进行攻击被称为洪水攻击。

### 解决方案：

#### 1.布隆过滤器

**布隆过滤器解决方法：**

将数据库中所有的查询条件，放入布隆过滤器中，当一个查询请求过来时，先经过布隆过滤器进行查，如果判断请求查询值存在，则继续查；如果判断请求查询不存在，直接丢弃。

**布隆过滤器原理**

原理就是一个对一个key进行k个hash算法获取k个值，在比特数组中将这k个值散列后设定为1，然后查的时候如果特定的这几个位置都为1，那么布隆过滤器判断该key存在。布隆过滤器可能会误判，如果它说不存在那肯定不存在，如果它说存在，那数据有可能实际不存在；Redis的bitmap只支持2^32大小，对应到内存也就是512MB，误判率万分之一，可以放下2亿左右的数据，性能高，空间占用率及小，省去了大量无效的数据库连接。

因此我们可以通过布隆过滤器，将Redis缓存穿透控制在一个可容范围内。布隆过滤器可以判断某个数据一定不存在，但是无法判断一定存在。

#### 2.缓存空对象

## 缓存击穿

### 概念：

相较于缓存穿透，缓存击穿的目的性更强，一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。这就是缓存被击穿，只是针对其中某个key的缓存不可用而导致击穿，但是其他的key依然可以使用缓存响应。 比如热搜排行上，一个热点新闻被同时大量访问就可能导致缓存击穿。

### 解决方案：

#### 1.设置热点数据永不过期

这样就不会出现热点数据过期的情况，但是当Redis内存空间满的时候也会清理部分数据，而且此种方案会占用空间，一旦热点数据多了起来，就会占用部分空间。

#### xxxxxxxxxx # 重新加载systemctl daemon-reload # 设置开机自动启动systemctl enable keepalived.service # 取消开机自动启动systemctl disable keepalived.service # 启动systemctl start keepalived.service # 停止systemctl stop keepalived.service bash

在访问key之前，采用SETNX（set if not exists）来设置另一个短期key来锁住当前key的访问，访问结束再删除该短期key。保证同时刻只有一个线程访问。这样对锁的要求就十分高。

## 缓存雪崩

### 概念：

大量的key设置了相同的过期时间，导致在缓存在同一时刻全部失效，造成瞬时DB请求量大、压力骤增，引起雪崩。

### 解决方案：

#### 1.redis高可用

这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群

#### 2.限流降级

这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。

#### 3.数据预热

数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

# Redis主从复制

## 概念

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点（Master/Leader）,后者称为从节点（Slave/Follower）， 数据的复制是单向的！只能由主节点复制到从节点（主节点以写为主、从节点以读为主）。

默认情况下，每台Redis服务器都是主节点，一个主节点可以有0个或者多个从节点，但每个从节点只能由一个主节点。

## 开启多个redis服务（一主二从）

### 1.去阿里云添加安全组规则，开放6379/6380/6381端口号，开放linux防火墙端口号

```shell
# 开放linux防火墙（6379/6380/6381）端口号：
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=6380/tcp
firewall-cmd --permanent --add-port=6381/tcp
```

### 2.复制redis.conf 到 redis-6380.conf，redis-6381.conf

```shell
# 首先进入redis.conf的目录下，使用以下两个命令可以找到redis.conf的目录 
  ps -aux|grep redis
  ls -l /proc/进程号/cwd

# 然后复制redis.conf，命令如下
  cp redis.conf redis-6380.conf
  cp redis.conf redis-6381.conf
```

### 3.修改 redis-6380.conf，redis-6381.conf信息（主要三个信息）

```java
分别是 pid，port端口号和slaveof 127.0.0.1 6379
用vim命令进入 redis-6380.conf配置文件后，命令模式下输入/pid查找pid并修改，slaveof xxx需要手动填写   

pidfile /var/run/redis-6380.pid
port 6380
slaveof 127.0.0.1 6379
```

### 4.启动redis服务（三个端口号）

```java
启动6379服务：redis-server redis.conf          redis-cli -p 6379
启动6380服务：redis-server redis-6380.conf     redis-cli -p 6380
启动6381服务：redis-server redis-6381.conf     redis-cli -p 6381
```

### 5.查看信息

```java
info replication
```

### 6.遇到master_link_status为down情况

```java
进入redis-6380.conf，redis-6381.conf配置文件中，修改如下两个配置
1.修改replicaof <masterip> <masterport>为（主机IP和端口号）
  replicaof 127.0.0.1 6379

2.修改masterauth <master-password>为
  masterauth redis123 (主机密码和从机密码设置相同)
```

# Redis哨兵模式

## 复制sentinel.conf到自己目录下

### 安装好的redis源码默认含有redis.conf文件和sentinel.conf文件

```java
将/opt/redis-6.0.6下的两个配置文件复制到自己的目录下（ycconfig）

cp /opt/redis-6.0.6/redis.conf /usr/local/bin/ycconfig/
cp /opt/redis-6.0.6/sentinel.conf /usr/local/bin/ycconfig/
```

## 复制sentinel.conf到sentinel-26380.conf，sentinel-23681.conf

```java
cp sentinel.conf sentinel-26380.conf
cp sentinel.conf sentinel-26381.conf
```

## 修改sentinel.conf配置文件（主要修改5处，其他redis默认的一般不修改）

```java
port 26379  
哨兵sentinel实例运行的端口，默认26379

daemonize yes(默认是no)  
redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

pidfile /var/run/redis-sentinel.pid
当redis以守护进程方式运行时，redis默认会把pid写入/var/run/redis-sentinel.pid文件，可以通过pidfile指定

logfile "/usr/local/bin/ycconfig/sentinel-26379.log"
指定日志文件名

sentinel auth-pass mymaster redis123
设置主的密码

sentinel monitor mymaster 127.0.0.1 6379 2 （redis默认配置）
mymaster名字可以自定义，ip地址为master节点的地址 后边的 2 代表的是，如果有俩个哨兵判断这个主节点挂了那这个主节点就挂了，通常设置为哨兵个数一半加一。
只有当半数以上的slave节点认为master节点宕机了,才会进行选举master节点,否则不会进行选举

sentinel down-after-milliseconds mymaster 30000 (redis默认配置)
哨兵连接主节点多长时间没有响应就代表挂了。后边 30000 是毫秒，也就是 30 秒。

sentinel parallel-syncs mymaster 1 （redis默认配置）
这个配置项是指在故障转移时，最多有多少个从节点对新的主节点进行同步,这个值越小完成故障转移的时间就越长，这个值越大就意味着越多的从节点因为同步数据而不可用,可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。

sentinel failover-timeout mymaster 180000  （redis默认配置）
1. 同一个sentinel对同一个master两次failover之间的间隔时间。
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
3.当想要取消一个正在进行的failover所需要的时间。
4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了。
```

## 启动redis哨兵服务

```shell
redis-sentinel sentienl.conf        (启动26379端口哨兵，两个步骤)
    redis-cli -p 26379
redis-sentinel sentienl-26380.conf  (启动26380端口哨兵，两个步骤)
    redis-cli -p 26380
redis-sentinel sentienl-26381.conf  (启动26381端口哨兵，两个步骤)
    redis-cli -p 26381
```

## 查看哨兵信息

```shell
info sentinel

127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6379,slaves=2,sentinels=1
```

## 测试哨兵模式

```shell
[root@yc ~]# ps -ef|grep redis  （找到主机进程号）
root      7390     1  0 Mar17 ?        00:04:03 redis-server 127.0.0.1:6380
root      7647     1  0 Mar17 ?        00:04:28 redis-server 127.0.0.1:6379
root      7837     1  0 Mar17 ?        00:04:03 redis-server 127.0.0.1:6381
root     14885     1  0 11:06 ?        00:00:03 redis-sentinel *:26379 [sentinel]
root     15311     1  0 11:14 ?        00:00:02 redis-sentinel *:26380 [sentinel]
root     15323  9975  0 11:14 pts/0    00:00:00 redis-cli -p 26380
root     15436     1  0 11:16 ?        00:00:01 redis-sentinel *:26381 [sentinel]
root     15449  9997  0 11:16 pts/2    00:00:00 redis-cli -p 26381
root     15874  9069  0 11:25 pts/1    00:00:00 redis-cli -p 6379
root     15921 15892  0 11:26 pts/3    00:00:00 grep --color=auto redis

[root@yc ~]# kill -9 7647  （杀死主机进程）

127.0.0.1:6381> info replication  （查看6381端口信息，发现角色从slave变为master）
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=417612,lag=0
master_replid:d3597e4133704dfe6b60c8103b5fc7174d480a4b
master_replid2:1447d768216f7530d3f885ae67f6529cfc593d02
master_repl_offset:417745
second_repl_offset:409820
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:4840
repl_backlog_histlen:412906


127.0.0.1:26380> info sentinel （查看哨兵信息，发现127.0.0.1:6381成为新的master）
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=127.0.0.1:6381,slaves=2,sentinels=3
```

## 遇到的问题

### 启动sentinel.conf报错

```java
No such master with specified name.
原因是配置的顺序，也就是我们监听的时候，是需要先配置监听master，给master取一个名字叫mymaster，才能配置这个认证节点的密码。但是默认配置是密码在前面，监听配置在后面，这样就会报这个错，调整一下即可。

sentinel monitor mymaster 127.0.0.1 6379 2
sentinel auth-pass mymaster redis123
顺序这样就能正常启动
```

### 主机杀死重连后，master_link_status:down

```java
这是因为从机设置了密码，当主机重连后会自动变为slave，在redis.conf配置文件中需要配置添加
masterauth redis123

这样再次重启后master_link_status:up 显示正常
```

### sentinel监视redis，master节点挂了不自动选举的问题，Next failover delay: I will not start a failover before Sat Mar 19 00:39:23 2022

```bash
#修改redis.conf、redis-6380.conf、redis-6381.conf 中bind配置后 重启自动选举生效 
1、bind 0.0.0.0

2、protected-mode no（这个我本来设置的就是no）

#修改了#sentinel leader-epoch mymaster后重启（不知道有没有影响）
#sentinel leader-epoch mymaster 44 这个我修改成2 重启后自动修改成44了，之前是184
```

# SpirngBoot集成Redis

## 1.添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 2.yml配置

```yaml
spring:
  redis:
    host: 8.140.1.39   #linux对应主机号IP
    port: 6381
    password: redis123
    database: 0   #默认使用redis第0个数据库
    lettuce:
      shutdown-timeout: 0
      pool:
        max-active: 8  # 最大活跃连接
        max-wait: -1   # 最大阻塞时间
        max-idle: 8    # 最大空闲连接
        min-idle: 0    # 最小空闲连接
    connect-timeout: 10000
```

## xxxxxxxxxx 1.进入windos下的.ssh目录（用git bash操作）cd C:\Users\yangcheng\.sshssh-keygenid_rsa_remote # 这个是生成私钥的名称，自己定义。不写的话默认id_rsa# 生成完私钥后还会生成一个公钥，公钥名为id_rsa_remote.pub​2.打开C:\Users\yangcheng\.ssh中的config文件，添加如下内容Host 8.140.1.39  HostName 8.140.1.39  User rootPreferredAuthentications publickey​# 本地文件，注意这个地方是放私钥的路径IdentityFile C:\Users\yangcheng\.ssh\id_rsa_remote​3.将公钥拷贝到服务器上（也就是id_rsa_remote.pub文件）4.在服务器上也使用 ssh-keygen 命令，进入/root/.ssh目录5. mv /root/id_rsa_remote.pub /root/.ssh/6.cat ~/.ssh/id_rsa_remote.pub >> ~/.ssh/authorized_keys# 需要注意的是：.ssh/目录的权限需要为700！authorized_keys设置为600chmod 700 /root/.sshchmod 600 /root/.ssh/authorized_keys# 如果发现修改不了权限,lsattr查询chattr -ia /root/.ssh/authorized_keys7.重启ssh服务systemctl restart sshd8.vscode就可以免密登录这台服务器# ll -a 可以显示隐藏文件（例如：.ssh文件）shell

```java
package com.yc.redis;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.geo.Point;
import org.springframework.data.redis.connection.RedisGeoCommands;
import org.springframework.data.redis.core.RedisTemplate;

import java.util.HashMap;

@SpringBootTest
class RedisSpringbootApplicationTests {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @Test
    void contextLoads() {
        System.out.println(redisTemplate.opsForValue().get("name"));
        System.out.println(redisTemplate.opsForList().range("list", 0, -1));
        System.out.println(redisTemplate.hasKey("name") + "  " + redisTemplate.hasKey("list"));
        System.out.println(redisTemplate.opsForValue().get("who"));
        System.out.println(redisTemplate.keys("*"));
        System.out.println(redisTemplate.opsForList().index("list", 3));

        System.out.println("===============================================");
        HashMap hashMap = new HashMap();
        hashMap.put("name", "yc");
        hashMap.put("age", "22");
        hashMap.put("sex", "男");
        redisTemplate.opsForHash().putIfAbsent("hash", "name", "yc");
        System.out.println(redisTemplate.opsForHash().get("hash", "name"));
        redisTemplate.opsForHash().putAll("hashmap", hashMap);
    }

    @Test
    void geospatial() {
        redisTemplate.opsForGeo().add("china:city", new Point(116.512885, 39.847469), "北京");
        redisTemplate.opsForGeo().add("china:city", new Point(121.486801, 31.24257), "上海");
        redisTemplate.opsForGeo().add("china:city", new Point(114.064552, 22.548457), "深圳");
        redisTemplate.opsForGeo().add("china:city", new Point(121.621631, 38.918954), "大连");
        redisTemplate.opsForGeo().add("china:city", new Point(120.215512, 30.253083), "杭州");
        redisTemplate.opsForGeo().add("china:city", new Point(100.626621, 36.292102), "海南");

        System.out.println(redisTemplate.opsForGeo().position("china:city", "北京", "上海", "深圳", "大连"));
        System.out.println("===========================================================");
        System.out.println("北京与上海之间的距离：" + redisTemplate.opsForGeo().distance("china:city", "北京", "上海", RedisGeoCommands.DistanceUnit.KILOMETERS));
        System.out.println("北京的地理坐标：" + redisTemplate.opsForGeo().position("china:city", "北京"));
    }

    @Test
    void hyperLogLog() {
        System.out.println(redisTemplate.opsForHyperLogLog().size("day"));
    }

}
```

## 4.遇到的问题

### 4.1 启动项目显示Unable to connect to 8.140.1.39:6381

```shell
# protected-mode yes 改为 protected-mode no (即该配置项表示是否开启保护模式，默认是开启，开启后Redis只会本地进行访问，拒绝外部访问)。
# 注释掉bind 127.0.0.1 即 #bind 127.0.0.1 (ps: 不注释掉，表示指定 redis 只接收来自于该 IP 地址的请求，注释掉后，则表示将处理所有请求)。

#开启防火墙（我的错误最后发现是6381防火墙没开）
# 开放6381端口
firewall-cmd --permanent --add-port=6381/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
```
