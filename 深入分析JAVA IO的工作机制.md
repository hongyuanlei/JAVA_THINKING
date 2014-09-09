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

这里就不详细解释每个子类如何使用了，如果不清楚的话可以参考一下 JDK 的 API 说明文档，这里只想说明两点，一个是操作数据的方式是可以组合使用的，如这样组合使用

```Java
OutputStream out = new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream(new File("path"))));
```

还有一点是流最终写到了什么地方必须要指定，要么是写到磁盘要么是写到网络中，其实从上面的类图中我们发现，写网络实际上也是写文件，只不过写网络还有一步需要处理就是底层操作系统再将数据传送到其它地方面不是本地磁盘。

##基于字符的I/O操作接口##

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符，所以I/O操作的都是字节而不是字符，但是为啥有操作字符的I/O接口呢？这是因为我们的程序中通常操作的数据都是以字符形式，为了操作方便当然要提供一个直接写字符的I/O接口，如些而已。我们知道字符到字节必须要经过编码转码，而这个编码又非常耗时，而且还会经常出现乱码，所以I/O的编码问题经常让人头痛。

下图是写字符的 I/O 操作接口涉及到的类，Writer 类提供了一个抽象方法 write(char cbuf[], int off, int len) 由子类去实现。

图3.Writer 相关类层次结构
![Writer 相关类层次结构](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/Writer%E7%9B%B8%E5%85%B3%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpg)

读字符的操作接口也有类似的类结构，如下图所示：

图4.Reader 类层次结构
![Reader 类层次结构](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/Reader%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84.jpg)

读字符的操作接口中也是 int read(char cbuf[], int off, int len)，返回读到的 n 个字节数，不管是 Writer 还是 Reader 类它们都只定义了读取或写入的数据字符的方式，也就是怎么写或读，但是并没有规定数据要写到哪去，写到哪去就是我们后面要讨论的基于磁盘和网络的工作机制。








