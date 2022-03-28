在PostgreSQL(以下简写为PG)中，内存缓冲区有多种应用时机和场景，比如日志的缓冲区、堆表的缓冲区等。同是堆表，在页面的替换上也有不同的策略。以下简要介绍这些内存页的换入换出算法

# LRU

LRU(Least Recently Used)即最近最少使用，是一种典型的缓存淘汰算法。该算法在Leetcode上还有一道题：[146. LRU缓存机制（中等）](https://leetcode-cn.com/problems/lru-cache/)

LRU的本质是说：将内存中最近使用过的数据认为是「有用的」，将很久都没有用过（即最近最少使用，中文理解起来比较绕）的数据认为是「没用的」。如果内存满了，或者达到限额了，就优先删除那些「没用的」的数据

## PG中的使用

### VFD

VFD(Virtual File Description，虚拟文件描述符)中就使用了LRU机制管理缓存池。`Vfd`在PG中的定位处于：



详见`Vfd`结构体：

```c
typedef struct vfd {
    int fd;        /* 该vfd所对应的物理文件描述符，若文件未打开，则为VFD_CLOSED */
    unsigned short fdstate; /* 该vfd的标志位，第0位表示文件是否为dirty(若为dirty，则需要在文件关闭前落盘)，第1位表示文件是否为临时文件(若为临时文件，则需要在关闭后删除) */
    ResourceOwner resowner; /* owner, for automatic cleanup */
    File nextFree;          /* 指向下一个空闲的vfd，为vfd数组的下标 */
    File lruMoreRecently;   /* 指向比该vfd最近更不常用的vfd */
    File lruLessRecently;   /* 指向比该vfd最近更常用的vfd */
    off_t fileSize; /* 文件大小 */
    char *fileName; /* 该vfd对应文件的名称，如果是空闲的vfd，则为null */
    /* NB: fileName is malloc'd, and must be free'd when closing the VFD */
    int fileFlags;   /* 该文件打开时的标志，包括只读、只写、读写等 */
    mode_t fileMode; /* 文件创建时所指定的模式 */
} Vfd;
```



## Demo

