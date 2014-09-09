#深入分析 JAVA I/O的工作机制

##Java的I/O类库的基本架构##
I/O问题是任何编程语言都无法回避的问题，可以说I/O问题是整个人机交互的核心问题，因为I/O是机器获取和交换信息的主要渠道。

Java的I/O操作类在包java.io下，大概有将近80个类，但是这些类大概可以分成四组，分别是：

*    1、基于字节操作的I/O接口：InputStream和OutputStream
*    2、基于字符操作的I/O接口：Writer和Reader
*    3、基于磁盘操作的I/O接口：File
*    4、基于网络操作的I/O接口：Socket

前两组主要是根据传输数据的数据格式，后两组主要是根据传输数据的方式，虽然Socket类并不在java.io包下，但是我仍然把它们划分在一起，因为我个人认为I/O的核心问题是么是数据格式影响I/O操作，要么是传输方式影响I/O操作，也就是将什么样的数据写到什么地方的问题，I/O只是人与机器或者机器与机器交互的手段，除了在它们能够完成这个交互功能外，我们关注的就是如何提高它的运行效率了，而数据格式和传输方式是影响效率最关键的因素。

##基于字节的I/O操作接口##

基于字节的I/O操作接口输入和输出分别是：InputStream和OutputStream,InputStream输入流的类继承层次如下图所示：

图1.InputStream相关类层次结构图
![InputStream相关类层次结构图](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/InputStream%E7%9B%B8%E5%85%B3%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpg)

输入流根据数据类型和操作方式又被划分成若干个子类，每个子类分别处理不同操作类型，OutputStream 输出流的类层次结构也是类似，如下图所示：

图2.OutputStream相关类层次结构
![OutputStream 相关类层次结构](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/OutputStream%E7%9B%B8%E5%85%B3%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpg)













