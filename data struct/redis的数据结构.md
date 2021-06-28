# redis 底层数据结构

## 1. string
SDS （Simple Dynamic String，简单动态字符串）是 Redis 底层所使用的字符串表示.

```c
typedef char *sds;

/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */  //实际存储的string大小
    uint8_t alloc; /* excluding the header and null terminator */  //buf大小
    unsigned char flags; /* 3 lsb of type, 5 unused bits */ //第三位类型，高未使用
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```


```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```
len 记录当前字节数组的长度（不包括\0），使得获取字符串长度的时间复杂度由O(N)变为了O(1)
alloc 记录了当前字节数组总共分配的内存大小（不包括\0）
flags 记录了当前字节数组的属性、用来标识到底是sdshdr8还是sdshdr16等
buf 保存了字符串真正的值以及末尾的一个\0




alloc:可以高效地执行追加操作（append），go的这里会新申请一块内存，在拼接；redis的特殊在于，他有一定的策略去申请内存，buf大小
append策略：<=1024*1024=1M 2倍；>1M +1M

## 问题1：
__attribute__ ((__packed__))不进行内存对齐
```c
printf("%ld\n", sizeof(struct sdshdr8));  // 3
printf("%ld\n", sizeof(struct sdshdr16)); // 5
printf("%ld\n", sizeof(struct sdshdr32)); // 9
printf("%ld\n", sizeof(struct sdshdr64)); // 17
```
redis 通过自己在malloc等c语言内存分配函数上封装了一层zmalloc
```c
if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \    // 确保内存对齐！
```

## 2. list
### ziplist

