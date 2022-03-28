## Redis理论知识

基于键值对(k(string类型)-v(数据结构))的数据库

Redis是以C语言实现的

### 数据结构

> #### 字符串

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

> #### 链表

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

> #### 哈希表

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



> 集合

> 有序集合



### 持久化

### 缓存

### 集群