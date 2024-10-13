# Readme
A comparison between Traditional-copy and Zero-copy.

### Contents

```
问题：把磁盘中的一个文件用网卡发出去。
```

```
解法1：使用Traditional-copy，会发生四次数据拷贝，该应用会发起两次系统调用和经历四次上下文切换：

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲
2. Traditional Memory Access Transfer，内核磁盘缓冲 -> 应用缓冲
3. Traditional Memory Access Transfer，应用缓冲 -> 内核网卡缓冲
4. Direct Memory Access Transfer，内核网卡缓冲 -> 网卡缓冲

1. 当应用调用read()时，应用会被换出
2. 当应用请求的文件数据被拷贝到应用缓冲后，应用会被换入
3. 当应用调用send()时，应用会被换出
4. 当应用请求的文件数据被拷贝到网卡缓冲后，应用会被换入
```

```
解法2.1：使用Zero-copy，需要使用Linux Kernel ^2.6，搭配不支持Scatter/Gather操作的网卡，会发生三次数据拷贝，该应用会发起一次系统调用和经历两次上下文切换：

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲
2. Traditional Memory Access Transfer，内核磁盘缓冲 -> 内核网卡缓冲
3. Direct Memory Access Transfer，内核网卡缓冲 -> 网卡缓冲

1. 当应用调用sendfile()时，应用会被换出
2. 当应用请求的文件数据被拷贝到网卡缓冲后，应用会被换入
```

```
解法2.2：使用Zero-copy，需要使用Linux Kernel ^2.6，搭配支持Scatter/Gather操作的网卡，会发生两次数据拷贝，该应用会会发起一次系统调用和经历两次上下文切换：

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲
2. Direct Memory Access Transfer，内核磁盘缓冲 -> 网卡缓冲

1. 当应用调用sendfile()时，应用会被换出
2. 当应用请求的文件数据被拷贝到网卡缓冲后，应用会被换入
```

### Credits
- [Efficient data transfer through zero copy - IBM Developer](https://developer.ibm.com/articles/j-zerocopy)
