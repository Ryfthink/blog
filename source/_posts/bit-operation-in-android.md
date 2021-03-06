---
title: 位运算的巧妙应用
date: 2015-06-20 12:00:00
categories: Android
tags: Java
---

Android 中的位运算应用了解一下~

<!--more-->


位运算
---
我们先了解一下 **4种** 最基本的二进制位运算
    
 **`|` 或运算( OR )**
按规则 `Y | 1 = 1` , `Y | 0 = Y` 运算

```java
   10010101   10100101
OR 11110000   11110000
=  11110101   11110101
```

**`&` 与运算( AND )**
按规则 `Y = 1` 时 `Y & 1 = 1` , `Y=*` 时 `Y & 0 = 0` 运算

```java
    10010101   10100101
AND 00001111   00001111
=   00000101   00000101
```

**`^` 异或( XOR )**
按位不同为1，相同为0

```java
    10011101   10010101
XOR 00001111   11111111
=   10010010   01101010
```


**`~` 非( NOT )**
按位取反

```java
~ 10011101   10010101
= 01100010   01101010
```
              

**下面是逻辑运算表**

| bit 1 | bit 2 | OR | AND | XOR |
| :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 | 0 |

<!--more-->

***

掩码运算
---
现在有这样一段代码:

```java
public static final int FLAG_FOCUSABLE = 0X0001; // 0001
public static final int FLAG_CLICKABLE = 0X0002; // 0010
public static final int FLAG_ENABLE = 0X0004; // 0100
public final int flags = 0;
```

`mFlags` 记录了某些状态位，如果想向 `flags` 上添加一个状态 `FLAG_ENABLE` ，可以这样做 :
    
```java
// 向mFlags上添加一个标识位
flags |= FLAG_ENABLE; // flags = 0100;
// 在添加一个标志位
flags |= FLAG_FOCUSABLE; // flags = 0101;
```

检查`flags` 是否含有 `FLAG_ENABLE` :

```java
boolean containEnableFlag;
containEnableFlag = (flags & FLAG_ENABLE) == FLAG_ENABLE
// 或者   
containEnableFlag = (flags & FLAG_ENABLE) != 0
```
    
移除 `flags` 中的 `FLAG_ENABLE` :

```java
flags &= ~FLAG_ENABLE; // flags = 0001
```
    
这几种操作方式为掩码运算，总结有以下几种方式

| 释义 | 运算  |
| :--------------   | :-------- |
| 设置或覆盖标志位  |   flags &#921; flagBit     |
| 移除标志位         |   flags & ~flagBit    |
| 检查是否含有标志位 |   (flags & flagBit) == flagBit |
| 标志位取反         |   ~flags  |

掩码运算并不直观，也不容易理解，所以在编程时并不推荐使用，但是如果考虑性能要求的话，使用掩码运算还是很划算的。

***

在Android 中的应用
---
**View** 类中可以设置其是否可见，可点击等，

```java
    setFocusable(focusable);
    setClickable(clickable);
```
    
查看 **View** 的源代码片段

```java
/**
* The view flags hold various views states.
*/
int mViewFlags;

/**
* This view wants keystrokes. Use with TAKES_FOCUS_MASK when calling
* setFlags.
*/
static final int FOCUSABLE = 0x00000001;

/**
* <p>Indicates this view can be clicked. When clickable, a View reacts
* to clicks by notifying the OnClickListener.<p>
*/
static final int CLICKABLE = 0x00004000;

/**
* Set flags controlling behavior of this view.
*
* @param flags Constant indicating the value which should be set
* @param mask Constant indicating the bit range that should be changed
*/
void setFlags(int flags, int mask) {
    ...     
    int old = mViewFlags;
    mViewFlags = (mViewFlags & ~mask) | (flags & mask);

    int changed = mViewFlags ^ old;
    if (changed == 0) {
        return;
    }
    int privateFlags = mPrivateFlags;
    ...
}

```

可以看到 `mViewFlags` 记录 **View** 的 **states**，当调用 `setFocusable()` , `setClickable()` 时,实际上是调用 `setFlags()` 来设置 `mViewFlags` 的值，像这种 **掩码运算** 的实际应用在 **Android FrameWork** 层很多地方都可以看到

***

Permissions on linux
---
在终端键入 `ls -l` ,会看到类似这样的输出结果

```shell
    -rw-rw-r--@  1 septenary  staff  1158  2 27 14:05 Readme.txt
    drwxrwxr-x@  6 septenary  staff   204  3 27 11:45 add-ons
    drwxr-xr-x   8 septenary  staff   272  6 10 14:43 build-tools
    drwxr-xr-x  47 septenary  staff  1598  3 27 11:40 docs
    drwxr-xr-x   5 septenary  staff   170  5 21 17:05 extras
    drwxr-xr-x  13 septenary  staff   442  7 28 12:08 platform-tools
    drwxrwxr-x@  7 septenary  staff   238  5 29 10:50 platforms
    drwxr-xr-x   5 septenary  staff   170  5 29 10:51 samples
    drwxr-xr-x   5 septenary  staff   170  4  1 20:22 sources
    drwxr-xr-x   4 septenary  staff   136  3 27 11:27 system-images
    drwxr-xr-x   4 septenary  staff   136  6 10 14:43 temp
    drwxr-xr-x  30 septenary  staff  1020  5 29 10:50 tools
```

第一列由 **d rwx rwx rwx** 这种形式构成, **d** 标识文件夹，**rwx** 代表用户或用户组的 **read write excute** 权限，其中
 
 ```shell
    r=4; // 0100
    w=2; // 0010
    x=1; // 0001
```
    
如果想要更改 **Readme.txt** 文件 为可写、可读、可执行，键入

```shell
    chmod 777 ./Readme.txt 
```
    
此处的 **777** 即为 `r + w + x` ,用二进制运算表示 就是 `0100 | 0010 | 0001`

***

变量置换
--- 
有变量 `a = 4; b = 7` , 将 `a`,`b` 值互换，通常这么写:

```shell
    int a = 4;
    int b = 7;
    int temp = a;
    a = b;
    b = temp;
```

利用 **异或`^`** 运算可以这么写

```shell
    int a = 4;
    int b = 7;
    a^=b;
    b^=a;
    a^=b;
```
    
这种方式省去了 `temp` 32个字节的内存消耗，且计算更快

[参考: Bitwise Operators and Bit Masks](http://www.vipan.com/htdocs/bitwisehelp.html)
