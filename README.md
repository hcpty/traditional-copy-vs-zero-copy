# Readme
A note about zero-copy.

问题：把一个文件从磁盘中读出来，然后通过网卡发出去。

解法一：使用传统的方法。

发生四次数据拷贝：
1. DMA拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. CPU拷贝，内核磁盘缓冲 -> 应用缓冲
3. CPU拷贝，应用缓冲 -> 内核网卡缓冲
4. DMA拷贝，内核网卡缓冲 -> 网卡缓冲

发生两次系统调用，该应用经历四次上下文切换：
1. 当应用调用read()时，应用被换出
2. 当应用请求的文件数据被写入应用缓冲后，应用被换入
3. 当应用调用send()时，应用被换出
4. 当应用请求的文件数据被写入网卡缓冲后，应用被换入

解法二：使用zero-copy的方法。

发生三次数据拷贝：
1. DMA拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. CPU拷贝，内核磁盘缓冲 -> 内核网卡缓冲
3. DMA拷贝，内核网卡缓冲 -> 网卡缓冲

发生一次系统调用，该应用经历两次上下文切换：
1. 当应用调用sendfile()时，应用被换出
2. 当应用请求的文件数据被写入网卡缓冲后，应用被换入

解法三：使用zero-copy结合支持gather operations的网卡的方法。

发生两次数据拷贝：
1. DMA拷贝，磁盘缓冲 -> 内核磁盘缓冲
2. DMA拷贝，内核磁盘缓冲 -> 网卡缓冲

发生一次系统调用，该应用经历两次上下文切换：
1. 当应用调用sendfile()时，应用被换出
2. 当应用请求的文件数据被写入网卡缓冲后，应用被换入
