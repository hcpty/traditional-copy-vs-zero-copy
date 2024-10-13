# Readme
A comparison between Traditional-copy and Zero-copy.

```
问题：把磁盘中的一个文件用网卡发出去。
```

```
解法1：使用Traditional-copy：

会发生四次数据拷贝，会浪费CPU周期和内存空间：
1. Direct Memory Access拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. Traditional Memory Access拷贝，内核磁盘缓冲 -> 应用缓冲
3. Traditional Memory Access拷贝，应用缓冲 -> 内核网卡缓冲
4. Direct Memory Access拷贝，内核网卡缓冲 -> 网卡缓冲

会发生两次系统调用，该应用会经历四次上下文切换：
1. 当应用调用read()时，应用会被换出
2. 当应用请求的文件数据被写入应用缓冲后，应用会被换入
3. 当应用调用send()时，应用会被换出
4. 当应用请求的文件数据被写入网卡缓冲后，应用会被换入
```

```
解法2.1：使用Zero-copy，搭配不支持Scatter/Gather功能的网卡：

会发生三次数据拷贝，会浪费CPU周期和内存空间：
1. Direct Memory Access拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. Traditional Memory Access拷贝，内核磁盘缓冲 -> 内核网卡缓冲
3. Direct Memory Access拷贝，内核网卡缓冲 -> 网卡缓冲

会发生一次系统调用，该应用会经历两次上下文切换：
1. 当应用调用sendfile()时，应用会被换出
2. 当应用请求的文件数据被写入网卡缓冲后，应用会被换入
```

```
解法2.2：使用Zero-copy，搭配支持Scatter/Gather功能的网卡：

会发生两次数据拷贝，不会浪费CPU周期和内存空间：
1. Direct Memory Access拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. Direct Memory Access拷贝，内核磁盘缓冲 -> 网卡缓冲

会发生一次系统调用，该应用会经历两次上下文切换：
1. 当应用调用sendfile()时，应用会被换出
2. 当应用请求的文件数据被写入网卡缓冲后，应用会被换入
```
