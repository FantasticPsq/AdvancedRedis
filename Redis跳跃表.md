### Redis 跳跃表 ###
1. 重点内容：
```text
1. 跳跃表是有序集合的底层实现之一
2，Redis的跳跃表实现由zskiplist和zskiplistNode两个结构组成，其中zskiplist
用于保存跳跃表的信息（比如头节点，尾节点，长度等),而zskiplistNode用于表示
跳跃表的节点
3. 每个跳跃表节点的层数是1到32之间的随机数，产生随机数用的是次幂定律（越大的数
出现的概率越小）。
4. 同一个跳跃表中，多个节点的分值可以相同，但是每个节点的成员对象必须是唯一的。
5. 跳跃表中的节点按照分值从小到大进行排列，当分支相同时，节点按照成员对象的大小
进行排序。
```

2. zskiplist:
```c
typedef struct zskiplist {
    // 表头节点和表尾节点
    struct zskiplist *header,*tail;
    // 表中节点的数量
    unsigned long length;
    // 表中层数量最大的节点的层数
    int level;
} zskiplist;
```
header和tail指针分别指向跳跃表表头节点和表尾节点，通过这两个指针，程序定位表头节点
和表尾节点的复杂度为O(1)。
通过使用length属性记录跳跃表中节点的数量，level表示表中层数最多的那个节点的层数，注意
并不包括头节点的层数。
![](https://hunter-image.oss-cn-beijing.aliyuncs.com/redis/skiplist/Redis%E8%B7%B3%E8%B7%83%E8%A1%A8.png)
3. zskiplistNode
```c
typedef struct zskiplistNode {
    struct zskiplistLevel {
        //前进指针
        struct zskiplistNode *forward;
        // 跨度:
        // 两个节点之间的跨度越大，他们相距的越远
        // 指向NULL的所有前进指针的跨度都为0
        unsigned int span;
    } level[];
    // 后退指针
    struct zskiplistNode *backward;
    // 分值
    double score;
    //成员对象：节点的成员对象是一个指针，他指向一个字符串对象，而字符串对象则保存一个SDS值
    robj *obj;
} zskiplistNode;
```
![](https://hunter-image.oss-cn-beijing.aliyuncs.com/redis/skiplist/zskiplistNode.png)