#深入理解JAVA虚拟机之运行时数区域

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都各有用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则是依赖用户线程的启动和结束而建站和销毁。

![Java虚拟机运行时数据区](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/Java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.jpg)

##程序计数器

程序计数器是一块较小的内存空间，它的作用可以看做当前字节码的`行号指示器`。

