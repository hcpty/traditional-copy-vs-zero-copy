# Readme
A comparison between Traditional-copy and Zero-copy.

### Traditional-copy vs Zero-copy

问题：把磁盘中的一个文件用网卡发出去。

解法1：使用Traditional-copy，应用程序会发起两次系统调用和经历四次上下文切换，会发生四次数据拷贝：

```
1. 当应用程序调用read()时，应用程序会被换出。
2. 当应用程序请求的文件数据被拷贝到应用程序缓冲后，应用程序会被换入。
3. 当应用程序调用send()时，应用程序会被换出。
4. 当应用程序请求的文件数据被拷贝到网卡缓冲后，应用程序会被换入。

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲。
2. Traditional Memory Access Transfer，内核磁盘缓冲 -> 应用程序缓冲。
3. Traditional Memory Access Transfer，应用程序缓冲 -> 内核网卡缓冲。
4. Direct Memory Access Transfer，内核网卡缓冲 -> 网卡缓冲。
```

解法2.1：使用Zero-copy，需要支持Zero-copy的内核，搭配不支持Scatter/Gather操作的网卡，应用程序会发起一次系统调用和经历两次上下文切换，会发生三次数据拷贝：

```
1. 当应用程序调用sendfile()时，应用程序会被换出。
2. 当应用程序请求的文件数据被拷贝到网卡缓冲后，应用程序会被换入。

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲。
2. Traditional Memory Access Transfer，内核磁盘缓冲 -> 内核网卡缓冲。
3. Direct Memory Access Transfer，内核网卡缓冲 -> 网卡缓冲。
```

解法2.2：使用Zero-copy，需要支持Zero-copy的内核，搭配支持Scatter/Gather操作的网卡，应用程序会发起一次系统调用和经历两次上下文切换，会发生两次数据拷贝：

```
1. 当应用程序调用sendfile()时，应用程序会被换出。
2. 当应用程序请求的文件数据被拷贝到网卡缓冲后，应用程序会被换入。

1. Direct Memory Access Transfer，磁盘缓冲 -> 内核磁盘缓冲。
2. Direct Memory Access Transfer，内核磁盘缓冲 -> 网卡缓冲。
```

### Credits
- [Efficient data transfer through zero copy - IBM Developer](https://developer.ibm.com/articles/j-zerocopy)
