## Redis理论知识

基于键值对(k(string类型)-v(数据结构))的数据库

Redis是以C语言实现的

### 数据结构

#### 字符串

为什么不用C语言的字符串直接实现？

① C语言以/0结尾 -> 获取长度遍历:时间复杂度O(n)

② 没有很好的扩容机制

③ 特殊字符无法处理（结尾\0）

**Redis数据结构**

sdshdr(simple dynamic string)

```	c	
struct sdshdr{ //redis3.0后的结构
    unsigned int len; //动态的字符串有效长度，剩余长度3.0之前用free记录未使用长度
    unsigned int alloc;//已经给sds分配的长度
    unsigned char flags;//类型
    char buf[];//存储字符串的结构
}
```

redis将sds分成8、16、32、64四种，flags用于记录sdshdr的种类，用以节省sds.alloc的开销。

**总结** 类似于C语言的字符串封装

① 字符串长度获取时间复杂度O(1)

② 减少字符串扩容时引起的数据搬运次数

③ 存储更加复杂的二进制数据

#### 链表

链表底层由双向链表实现

节点实现

```C
typedef struct listNode{
    struct listNode* prev;
    struct listNode* next;
    void * value;
}listNode;
```

链表实现

```C
typedef struct list{
    listNode * head;//链表头
    listNode * tail;//链表尾
    unsigned long len;//链表长度
    void *(*dup) (void *ptr);//节点值复制函数
    void *(*free)(void *ptr);//节点值释放函数
    void (*match)(void *ptr, void *key);//节点值对比函数
}
```

| 常见API<br />（redis文档） | 功能             |
| -------------------------- | ---------------- |
| lpush                      | 左加             |
| rpush                      | 右加             |
| lpop                       | 左出             |
| rpop                       | 右出             |
| llen                       | 获取链表长度     |
| lrange                     | 按索引范围获取值 |

#### 哈希表

键值对，获取值的时间复杂度O(1)；C语言没有哈希实现，Redis自身通过拉链法实现

```c
dictht;//单纯表示一个哈希表
dictEntry;//代表哈希表的一项，键值对
dict;//Redis给外层调用的，包含两个dictht
```

```c
typedef struct dictht{
    dictEntry **table;//哈希表数组（哈希表项集合）
    unsigned long size;//哈希表大小，一般是2的幂次
    unsigned long sizemask;//哈希表掩码，保证访问不会溢出，例如访问时与0011相与，保留低两位的值
    unsigned long used;//哈希表已使用大小
}dictht;
```

**负载因子**/**空闲度**：指used与size之比，越低查找速度越快

① 不要太空：浪费空间；	② 不要太挤，影响速率；

```C
typedef struct dictEntry{//键值对
    void *key;//key
    union{
        void *val;//value
        uint64_t u64;
        int64_t s64;
        double d;
    }v;
    struct dictEntry next;//拉链法，指向下一个节点
}dictEntry;
```

```c
typedef struct dict{//外界访问接口
    dictType *type;
    void *private;
    dictht ht[2];
    int rehashidx;
}dict;
```

**rehash**: 当负载因子不在合理范围内时根据ht[0] 老表重新计算哈希值和索引值赋予ht[1] 新表，当赋值完成后释放ht[0]所占用空间，将ht[0]指向ht[1]，ht[1]指null；

**rehash执行条件**：

① redis未执行后台备份 负载因子>1

② redis执行后台备份 负载因子 >5

**rehashidx**一般是-1，表示未进行rehash，当置为正时表示rehash过程的进度；



#### 集合

> 普通集合

哈希表的封装与实现，key的唯一性->集合来使用

> 整数集合

redis特有的数据结构，集合中只有整数

```c
typedef struct intset{
    uint32_t encoding;//编码方式，uint16_t，uint32_t，uint64_t
    uint32_t length;//集合长度
    int8_t contents[];//元素数组
}
```

通过二分法查找数组中的数 OlogN -> 节省空间

由于需要保持空间有序，修改数据需要申请空间或释放空间



#### 有序集合

基于跳表实现

##### 跳表

基于单链表的数据结构，元素有序，查找任意元素复杂度O(N)

参考二叉树，新建索引

跳表节点定义

```C
typedef struct zskiplistNode{
    sds ele;//元素，在热词情境中，变成一段文字
    
    double score;//权重值，热词情境下就是热度
    
    struct zskiplistNode *backward;//指向后面的指针
    
    struct zskiplistlevel{
        struct zskiplistNode *forward;
        unsigned long span;
    }level[];
    //节点的level数组 x.level[i].span
    //表示节点x在第i层到其下一个节点需要跳过的节点数，两个相邻的节点span=1
}zskiplistNode;
```

跳表定义

```C
typedef struct zskiplist{
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
}zskiplist;
```



![image-20220410204054987](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410204054987.png)

跳表存在索引的问题（修改数据时索引需要重建）-> 概率索引插入

ZSet

redis实现微博热搜TopK

键存储事件、值存储热度，跳表实现O(NlogN)



### 持久化

把内存的数据搬运到硬盘

redis是内存型数据库，持久化保证数据断电后不消失

#### RDB（Redis Data Base）

全量备份，把当前redis内存中的数据生成快照（RDB文件）保存在硬盘中

##### RDB结构

![image-20220410205245390](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410205245390.png)

![image-20220410205644288](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410205644288.png)

##### 触发条件

>  手动触发

save 命令： 主线程执行rdbSave，服务器进程阻塞，不能处理其他请求

bgsave（background save）: fork子进程执行rdbSave，主线程处理其他请求

> 自动触发

配置文件写入 save m n 在m秒内发生n次变化自动执行bgsave



#### AOF (Append Only File)

增量备份，记录之后所有对Redis数据进行修改的操作命令

AOF的重写与恢复

实现方式：创建一个新的AOF文件，替代原有的冗余AOF文件（之根据当前redis的数据状态生成）

##### 触发方式

> 手动触发

bgrewriteaof命令

> 自动触发

配置文件 appendonly yes

※ Always 同步写回，每次执行写命令落入磁盘

※ Everysec 每秒写回，命令写入内存缓存区，redis会把该缓存区命令写到磁盘AOF文件中

※ No 操作命令写到redis缓存区，落盘交给操作系统

默认 EverySec

![image-20220410210919455](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410210919455.png)



### 缓存

#### 缓存淘汰

原因：缓存的空间是固定大小的

##### 先进先出 FIFO

##### 最近最少使用 LRU（Redis默认）

双向链表 + 哈希表 形式存储

双向链表用于组织数据 将最新被访问到的节点放在表头

哈希表用于存储键key 和 值链表节点

##### 最不经常使用 LFU

链表节点多一个字段：被访问次数



#### 过期删除

缓存一般有有效期

##### 主动删除

设置删除时间间隔，指定时间后主动删除：

有可能会撞到redis（缓存）繁忙的时候

##### 惰性删除

程序取值时查看数据是否已经过期，过期时删除：

容易造成某些数据长期霸占内存

##### 定期删除

每隔一段时间，主动删除，不主动删除时，惰性删除；



#### 缓存一致

####  Cache Aside

提升并发量，将数据从数据库载入缓存，保持数据一致性

核心：允许在一定小范围内，数据非一致，但最终是一致的

缓存是辅助数据手段，缓存的数据更新了，并非更新缓存数据，而是删除

##### model1

先删再改

![image-20220410212837908](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410212837908.png)

可能存在的问题：

写请求慢于读请求，使得用户读到缓存的旧数据

##### model1 - improve

延时双删方法

![image-20220410213210075](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410213210075.png)



##### model2

先改再删

![image-20220410215549637](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410215549637.png)

此模型的数据发生概率不高



#### Read/Write Through

核心在于把缓存作为主要的数据读取方式（避免缓存击穿）

![image-20220410215926161](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410215926161.png)

左： Through模式	右：CA 模式

#### Write Behind

![image-20220410215952435](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410215952435.png)

#### 缓存穿透

查询一个数据库不存在的数据，缓存和数据库都被穿透

**解决方法**

##### ① 拦截非法查询请求

如 控制器层拦截 return error

##### ② 缓存空对象

查询了刚被删除的数据，在缓存中置为 value = null，防止打到数据库上

##### ③ 布隆过滤器

![image-20220410221223499](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410221223499.png)

输入查询数据，经过hash计算，判断对应位置是否都为1 -> 才是有效请求



#### 缓存击穿

查询某个数据的值，缓存中没有，但是数据库有，请求连到数据库

一般缓存都有过期时间，缓存击穿很常见

**问题：如果缓存击穿的是热点数据？**

redis是普通关系型数据库的10-100倍，redis勉强处理->Mysql宕机

**解决方法：**

① Mysql: 减少击穿后的直接流量，加锁，保护数据库 

② Redis: 热点数据不过期；

​				热点数据后台启动异步线程，重新把数据填回缓存；



#### 缓存雪崩

一大批被缓存的数据同时失效，数据请求全部打到数据库

**解决方法**

Mysql: 强行减少并发量 -> 加锁

Redis:

① 热点数据永不过期 or 重新回传；

② 分析失效时间，尽量让失效时间点分散；

③ 缓存预热 -> ***上线前*** 分析当天情况，将热点数据加载到缓存系统；



### 集群

好几台机器上redis的协同操作，多机情况

#### 主从复制

![image-20220410222022403](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410222022403.png)

从库会滞后主库一定时间

配置文件中需要修改 主从模式（master/slave）和 对应的 ip：port

主从复制过程：

① 主库向从库握手通信

② 从库返回主库PSYNC命令，开启数据同步，发送主库id（网断了，可能能会换机器）和复制的进度偏移量offset

③ 主库Reply从库，确认全量复制 or 断线后重复制



##### 全量复制

初步复制后的同步，从库毫无数据

**主库**：执行BGSAVE 生成RDB+开辟缓冲区，新的数据命令也会冲入缓冲

**从库**：从库通过RDB文件恢复数据

**命令传播阶段**：主库修改时，主库会把数据命令发给从库



##### 断线后重复制

重连后需要依赖 服务器运行ID（唯一确定主库身份）和复制偏移量，复制积压缓冲区（先进先出队列，存储了最近主节点的数据修改命令）

当ID不同时，执行全量复制



#### 哨兵

不提供数据服务的redis服务器->监控所有的主从库，主库掉线了，从从库中选出新主库

通过心跳确认库存活

哨兵组监控主库，其中一个丢失心跳，其他哨兵尝试连接，多数未能连接上时，将从库中的一台选为主库（故障转移）-> 发送slaveof no one，同时向其他slave发送新主库的IP端口

r故障转移过程中，主库重新上线会变成从库



#### cluster（集群）

redis提供的分布式数据库解决方案

-- 自动将数据切分给多个节点存储

-- 即便这些节点中一部分宕机也能继续执行数据操作

##### 分区策略

键值对，采用虚拟槽，所有键通过CRC16校验函数，对16384取模，决定数据分配到哪个槽位

每个redis的cluster节点负责一部分槽（slot）数据的存储

节点可以拓展主从复制模式

##### 查询策略

每个节点都会存储整个集群的节点信息（**元信息**）

##### 数据传播

gossip协议，每个节点都把自己的数据信息通过这个协议散布

周期性执行，所有被感染的节点选择k个邻接节点散步信息

##### 扩容和缩容

数据太多了 or 太少了 -> 数据迁移

![image-20220410224533504](C:\Users\凌宇\AppData\Roaming\Typora\typora-user-images\image-20220410224533504.png)

