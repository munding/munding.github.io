---
title: "Redis源码之简单动态字符串"
date: 2021-11-05
tags: ["源码阅读",""]
categories: ["Redis",""]
description: ""
summary: ""
draft: false
---

Redis中字符串的实现并没有完全使用C字符串，而是重新定义了简单动态字符串SDS（Simple Dynamic String）用来表示字符串。

sds.h/sdshdr

```c
struct sdshdr {
    unsigned int len; //记录buf数组中已使字节的数量
    unsigned int free; // 记录buf数组中未使用字节的数量
    char buf[]; //字节数组，用于保存字符串
};
```

buf数组长度不一定就是字符串长度+1（"\0"），还有free空间，数组内未使用的字节通过free属性记录。

相比于C字符串，SDS有以下优势：

### 兼容部分C字符串函数

sds.c/sdsnew

```c
 * mystring = sdsnewlen("abc",3);
 *
 * You can print the string with printf() as there is an implicit \0 at the
 * end of the string. However the string is binary safe and can contain
 * \0 characters in the middle, as the length is stored in the sds header. */
sds sdsnewlen(const void *init, size_t initlen) {
    struct sdshdr *sh;

    if (init) {
        sh = zmalloc(sizeof(struct sdshdr)+initlen+1);
    } else {
        sh = zcalloc(sizeof(struct sdshdr)+initlen+1);
    }
    if (sh == NULL) return NULL;
    sh->len = initlen;
    sh->free = 0;
    if (initlen && init)
        memcpy(sh->buf, init, initlen);
    sh->buf[initlen] = '\0';
    return (char*)sh->buf;
}

/* Create a new sds string starting from a null termined C string. */
sds sdsnew(const char *init) {
    size_t initlen = (init == NULL) ? 0 : strlen(init);
    return sdsnewlen(init, initlen);
```

从SDS的创建逻辑中可以看出

1. SDS遵循C字符串以空字符（"\0"）结尾。
2. SDS中的len属性（sds.h/sdslen）同C字符串函数strlen返回结果相同，即不计算尾部空字符。

这样SDS就可以直接重用一部分C字符串函数库里面的函数（如打印，显示类函数，<stdio.h>/printf），而字符串的修改操作，则使用SDS自定义优化后的函数。

### 常数复杂度获取字符串长度

C获取一个C字符串的长度，程序必须遍历整个字符串，直到遇到代表字符串结尾的空字符串位置，这个操作的复杂度为O(N)。

但是对于SDS来说，获取字符串长度只需要访问SDS中的len属性。复杂度仅为O(1)，确保了获取字符串长度这样的高频操作不会成为Redis性能瓶颈。

对于SDS的修改操作，SDS会实时维护len属性，如sds.c/sdscat（追加C字符串到SDS字符串）

```c
/* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
 * end of the specified sds string 's'.
 *
 * After the call, the passed sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdscatlen(sds s, const void *t, size_t len) {
    struct sdshdr *sh;
    size_t curlen = sdslen(s);

    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    sh = (void*) (s-(sizeof(struct sdshdr)));
    memcpy(s+curlen, t, len);
    sh->len = curlen+len;
    sh->free = sh->free-len;
    s[curlen+len] = '\0';
    return s;
}

/* Append the specified sds 't' to the existing sds 's'.
 *
 * After the call, the modified sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdscatsds(sds s, const sds t) {
    return sdscatlen(s, t, sdslen(t));
}
```

### 杜绝缓冲区溢出

C字符串由于不记录自身长度，对其进行修改操作容易造成缓冲区溢出（buffer overflow）。如C字符串拼接函数<stdio.h>/strcat，内存中相邻的字符串s1和s2，对s1字符串做拼接操作时，如果没有提前为s1分配足够的空间，则s2保存的内容会被意外修改。



SDS内部维护了一个free字段，当SDS API需要对其进行修改时，API会调用`sdsMakeRoomForDS`函数检测当前SDS的free空间是否满足要求，满足直接进行修改；不满足`sdsMakeRoomForDS`则会将SDS的空间扩展至执行修改所需的大小，避免缓冲区溢出的情况（如上方sds.c/sdscat）。

### 减少修改字符串长度时所需内存重分配次数

Redis作为数据库，经常被用于速度要求严苛，数据被频繁修改的场景。SDS实现了**空间预分配**和**惰性空间释放**两种优化策略。

sds.c/sdsMakeRoomFor

```c
/* Enlarge the free space at the end of the sds string so that the caller
 * is sure that after calling this function can overwrite up to addlen
 * bytes after the end of the string, plus one more byte for nul term.
 *
 * Note: this does not change the *length* of the sds string as returned
 * by sdslen(), but only the free buffer space we have. */
sds sdsMakeRoomFor(sds s, size_t addlen) {
    struct sdshdr *sh, *newsh;
    size_t free = sdsavail(s);
    size_t len, newlen;

    if (free >= addlen) return s;
    len = sdslen(s);
    sh = (void*) (s-(sizeof(struct sdshdr)));
    newlen = (len+addlen);
    if (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC;
    newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);
    if (newsh == NULL) return NULL;

    newsh->free = newlen - len;
    return newsh->buf;
}
```

#### 空间预分配

可以发现，在SDS API进行字符串新增逻辑中会给SDS重新分配free空间。

1. 如果SDS的长度（len属性）小于`SDS_MAX_PREALLOC（1024KB=1M）`，则会分配和len属性同样大小的未使用空间给buf，这时SDS的len属性和free属性值相同。
2. 如果SDS长度大于或者等于`SDS_MAX_PREALLOC（1024KB=1M）`，则会直接给free属性分配`SDS_MAX_PREALLOC（1024KB=1M）`的大小。

通过空间预分配策略，Redis可以减少连续执行字符串增长操作所需的内存重分配次数.

#### 惰性空间释放

sds.c/sdstrim

```c
/* Remove the part of the string from left and from right composed just of
 * contiguous characters found in 'cset', that is a null terminted C string.
 *
 * After the call, the modified sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call.
 *
 * Example:
 *
 * s = sdsnew("AA...AA.a.aa.aHelloWorld     :::");
 * s = sdstrim(s,"A. :");
 * printf("%s\n", s);
 *
 * Output will be just "Hello World".
 */
sds sdstrim(sds s, const char *cset) {
    struct sdshdr *sh = (void*) (s-(sizeof(struct sdshdr)));
    char *start, *end, *sp, *ep;
    size_t len;

    sp = start = s;
    ep = end = s+sdslen(s)-1;
    while(sp <= end && strchr(cset, *sp)) sp++;
    while(ep > start && strchr(cset, *ep)) ep--;
    len = (sp > ep) ? 0 : ((ep-sp)+1);
    if (sh->buf != sp) memmove(sh->buf, sp, len);
    sh->buf[len] = '\0';
    sh->free = sh->free+(sh->len-len);
    sh->len = len;
    return s;
}
```

可以发现，在进行字符串剪切操作时，多出的buf空间并不会直接释放，而是存储在free字段中。

同时为了避免内存泄露，SDS也提供了sds.c/sdsRemoveFreeSpace释放free空间操作，在redis.c/clientsCronResizeQueryBuffer中可以看到，当querybuf大于1024字节，会进行释放操作。

```c
/* Reallocate the sds string so that it has no free space at the end. The
 * contained string remains not altered, but next concatenation operations
 * will require a reallocation.
 *
 * After the call, the passed sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdsRemoveFreeSpace(sds s) {
    struct sdshdr *sh;

    sh = (void*) (s-(sizeof(struct sdshdr)));
    sh = zrealloc(sh, sizeof(struct sdshdr)+sh->len+1);
    sh->free = 0;
    return sh->buf;
}

/* The client query buffer is an sds.c string that can end with a lot of
 * free space not used, this function reclaims space if needed.
 *
 * The function always returns 0 as it never terminates the client. */
int clientsCronResizeQueryBuffer(redisClient *c) {
    size_t querybuf_size = sdsAllocSize(c->querybuf);
    time_t idletime = server.unixtime - c->lastinteraction;

    /* There are two conditions to resize the query buffer:
     * 1) Query buffer is > BIG_ARG and too big for latest peak.
     * 2) Client is inactive and the buffer is bigger than 1k. */
    if (((querybuf_size > REDIS_MBULK_BIG_ARG) &&
         (querybuf_size/(c->querybuf_peak+1)) > 2) ||
         (querybuf_size > 1024 && idletime > 2))
    {
        /* Only resize the query buffer if it is actually wasting space. */
        if (sdsavail(c->querybuf) > 1024) {
            c->querybuf = sdsRemoveFreeSpace(c->querybuf);
        }
    }
    /* Reset the peak again to capture the peak memory usage in the next
     * cycle. */
    c->querybuf_peak = 0;
    return 0;
}
```

### 二进制安全

SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据（如不以"\0"当作字符串结尾），程序不会对其中的数据做任何限制、过滤、或者假设，数据在写入时是什么样的，他被读取时就是什么样子。因此Redis可以不仅可以保存文本数据，还可以保存图片、音频、视频、压缩文件这样的二进制数据。

### SDS API

![](https://img.aladdinding.cn/sdsapi.png)

