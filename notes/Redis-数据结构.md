[[Redis]]

- [ ] 跳表的深入
- [ ] 

<img src="Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250523220957219.png" alt="image-20250523220957219" style="zoom: 33%;" />

## 数据结构

### SDS

![image-20250520231840092](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520231840092.png)

特性：

- 二进制安全
- 不会发生缓冲区溢出
- 高效获取字符串长度
- 节省内存空间

#### flags 成员变量

- sdshdr16 类型的 len 和 alloc 的数据类型都是 uint16_t，表示字符数组长度和分配空间大小不能超过 2 的 16 次方。
- sdshdr32 则都是 uint32_t，表示表示字符数组长度和分配空间大小不能超过 2 的 32 次方。

### 链表

#### 链表节点

```c
typedef struct listNode {
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    void *value;
} listNode;
```

![image-20250520232302231](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520232302231.png)

#### 链表结构

```c
typedef struct list {
    listNode *head;//链表头节点
    listNode *tail;//链表尾节点
    void *(*dup)(void *ptr);//节点值复制函数
    void (*free)(void *ptr);//节点值释放函数
    int (*match)(void *ptr, void *key);//节点值比较函数
    unsigned long len;//链表节点数量
} list;
```

list 结构为链表提供了链表头指针 head、链表尾节点 tail、链表节点数量 len、以及可以自定义实现的 dup、free、match 函数。

![image-20250520232450940](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520232450940.png)

### ziplist

![image-20250523221643679](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250523221643679.png)

- ***zlbytes***，记录整个压缩列表占用的内存字节数；
- ***zltail***，记录压缩列表「尾部」节点距离起始地址由多少字节，也就是列表尾的偏移量；
- ***zllen***，记录压缩列表包含的节点数量；
- ***zlend***，标记压缩列表的结束点，固定值 0xFF（十进制 255）。
- ***prevlen***，记录了「前一个节点」的长度，目的是为了实现从后向前遍历；
- ***encoding***，记录了当前节点实际数据的「类型和长度」，类型主要有两种：字符串和整数。
- ***data***，记录了当前节点的实际数据，类型和长度都由 `encoding`  决定；

**问题：**连锁更新问题



### 哈希表

#### 哈希表结构

```c
typedef struct dictht {
    dictEntry **table;    //哈希表数组    
    unsigned long size;  //哈希表大小    
    unsigned long sizemask;//哈希表大小掩码，用于计算索引值    
    unsigned long used;//该哈希表已有的节点数量
} dictht;
```

![image-20250520232701716](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520232701716.png)

#### 哈希表节点

```c
typedef struct dictEntry {
    void *key; //键值对中的键
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;//键值对中的值   
    struct dictEntry *next;//指向下一个哈希表节点，形成链表
} dictEntry;
```

#### rehash

思考为什么要用哈希表？

![image-20250520232856149](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520232856149.png)

rehash 操作：

- 给「哈希表 2」分配空间，一般会比「哈希表 1」大 2 倍；
- 将「哈希表 1」的数据迁移到「哈希表 2」中；
- 迁移完成后，「哈希表 1」的空间会被释放，并把「哈希表 2」设置为「哈希表 1」，然后在「哈希表 2」新创建一个空白的哈希表，为下次 rehash 做准备。

![image-20250520232946540](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520232946540.png)

#### 渐进式 rehash

渐进式 rehash 步骤：

- 给「哈希表 2」分配空间；
- **在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1」中索引位置上的所有 key-value 迁移到「哈希表 2」上**；
- 随着处理客户端发起的哈希表操作请求数量越多，最终在某个时间点会把「哈希表 1」的所有 key-value 迁移到「哈希表 2」，从而完成 rehash 操作。

在进行渐进式 rehash 的过程中，会有两个哈希表，所以在渐进式 rehash 进行期间，哈希表元素的删除、查找、更新等操作都会在这两个哈希表进行。

例子：

查找一个 key 的值的话，先会在「哈希表 1」里面进行查找，如果没找到，就会继续到哈希表 2 里面进行找到。

在渐进式 rehash 进行期间，新增一个 key-value 时，会被保存到「哈希表 2」里面，而「哈希表 1」则不再进行任何添加操作，这样保证了「哈希表 1」的 key-value 数量只会减少，随着 rehash 操作的完成，最终「哈希表 1」就会变成空表。

### 整数集合

整数集合是  Set 对象的底层实现之一。当一个 Set 对象只包含整数值元素，并且元素数量不大时，就会使用整数集这个数据结构作为底层实现。

```c
typedef struct intset {    
    uint32_t encoding;//编码方式    
    uint32_t length;//集合包含的元素数量    
    int8_t contents[];//保存元素的数组
} intset;
```

可以看到，保存元素的容器是一个 contents 数组，虽然 contents 被声明为 int8_t 类型的数组，但是实际上 contents 数组并不保存任何 int8_t 类型的元素，contents 数组的真正类型取决于 intset 结构体里的 encoding 属性的值。比如：

- 如果 encoding 属性值为 INTSET_ENC_INT16，那么 contents 就是一个 int16_t 类型的数组，数组中每一个元素的类型都是 int16_t；
- 如果 encoding 属性值为 INTSET_ENC_INT32，那么 contents 就是一个 int32_t 类型的数组，数组中每一个元素的类型都是 int32_t；
- 如果 encoding 属性值为 INTSET_ENC_INT64，那么 contents 就是一个 int64_t 类型的数组，数组中每一个元素的类型都是 int64_t；

#### 整数集合的升级操作



1. 假设有一个整数集合里有 3 个类型为 int16_t 的元素。![image-20250520233417945](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520233417945.png)

**在原本空间的大小之上再扩容多 80 位（4x32-3x16=80），这样就能保存下 4 个类型为 int32_t 的元素**。![image-20250520233356218](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250520233356218.png)

### 跳表

Redis 只有 Zset 对象的底层实现用到了跳表，跳表的优势是能支持平均 O(logN) 复杂度的节点查找。

zset 结构体里有两个数据结构：一个是跳表，一个是哈希表。这样的好处是既能进行高效的范围查询，也能进行高效单点查询。

```c
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

Zset 对象在执行数据插入或是数据更新的过程中，会依次在**跳表**和**哈希表**中插入或更新相应的数据，从而保证了跳表和哈希表中记录的信息一致。

- Zset 对象能支持范围查询（如 ZRANGEBYSCORE 操作）：跳表
- 以常数复杂度获取元素权重（如 ZSCORE 操作）：哈希表进行索引

 struct zset 中的哈希表只是用于以常数复杂度获取元素权重，大部分操作都是跳表实现的。

#### 跳表结构设计

![image-20250523222153693](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250523222153693.png)
**跳表节点**

```c
typedef struct zskiplistNode {
    sds ele;//Zset 对象的元素值 
    double score;//元素权重值
    struct zskiplistNode *backward;//后向指针
    //节点的 level 数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;//跨度是用来记录两个节点之间的距离
    } level[];
} zskiplistNode;
```

跳表是一个带有层级关系的链表，而且每一层级可以包含多个节点，每一个节点通过指针连接起来，实现这一特性就是靠跳表节点结构体中的**zskiplistLevel 结构体类型的 level 数组**。

![img](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/3%E5%B1%82%E8%B7%B3%E8%A1%A8-%E8%B7%A8%E5%BA%A6.png)

**跨度实际上是为了计算这个节点在跳表中的排位**。

从头节点点到该结点的查询路径上，将沿途访问过的所有层的跨度累加起来，得到的结果就是目标节点在跳表中的排位。

**跳表结构体**

```c
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

### quicklist

quicklist 就是「双向链表 + 压缩列表」

#### quicklist 结构设计

```c
typedef struct quicklist {
    quicklistNode *head;//quicklist 的链表头
    quicklistNode *tail;//quicklist 的链表尾
    unsigned long count;//所有压缩列表中的总元素个数
    unsigned long len;//quicklistNodes 的个数       
    ...
} quicklist;
```

#### quicklistNode结构定义

```c
typedef struct quicklistNode {
    struct quicklistNode *prev;     //前一个 quicklistNode
    struct quicklistNode *next;     //后一个 quicklistNode
    unsigned char *zl;  					  //quicklistNode 指向的压缩列表
    unsigned int sz;  //压缩列表的的字节大小              
    unsigned int count : 16;        //ziplist 中的元素个数 
    ....
} quicklistNode;
```

![image-20250523223321841](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250523223321841.png)

### listpack

![image-20250523223440099](Redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.assets/image-20250523223440099.png)

- encoding，定义该元素的编码类型，会对不同长度的整数和字符串进行编码；
- data，实际存放的数据；
- len，encoding+data 的总长度；
