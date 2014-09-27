#深入理解JAVA虚拟机之方法调用

方法调用并不等同于方法执行，`方法调用阶段唯一的任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程`。在程序运行时，进行方法调用是最普遍、最频繁的操作，但前面已经讲过了，Class文件的编译过程中不包含传统编译中的连接步骤，`一切方法调用在Class文件里面存储的都只是符号引用，而不是方法在实际运行时内存布局中的入口地址（相当于之前所说的直接引用）`。这个特性给Java带来了更强大的动态扩展能力，但也使用Java方法的调用过程变得相对复杂起来，需要在类加载期间甚至到运行期间才能确定目标方法的直接引用。

##解析

继续前面关于方法调用的话题，所有方法调用中的目标方法在Class文件里面都只一个常量池中的符号引用，`在类加载的解析阶段，会将其中的一部分符号引用转化为直接引用，这种解析能成立的前提是：方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不可改变的。`换句话说，调用目标写、编译器进行编译时就必须确定下来。这类方法的调用称为解析（Resolution）。

在Java语言中，符合“编译期可知，运行期不可变”这个要求的方法主要有`静态方法和私有方法两大类`，前者与类型直接关联，后者在外部不可被访问，这两种方法都不可能通过继承或别的方式重写其他版本，因此它们都适合在类加载阶段进行解析。

与之相对应，在Java虚拟机里面提供了四条方法调用字节码指令，分别是：

*    invokestatic:调用静态方法；
*    invokespecial:调用实例构造器<init>方法、私有方法和父类方法；
*    invokevirtual:调用所有的虚方法；
*    invokeinterface:调用接口方法，会在运行时再确定一个实现此接口的对象。

`只要能被invokestatic和invokespecial指令调用的方法，都可以在解析阶段确定唯一的调用版本，符合这个条件的有静态方法、私有方法、实例构造器和父类方法回类，它们在类加载的时候就会把符号引用解析为该方法的直接引用。这些方法可以称为非虚方法，与之相反，其他方法就称为虚方法（除去final方法）。`

Java中的非虚方法除了使用invokestatic和invokespecial调用的方法之处还有一种，就是被final修饰的方法。虽然final方法使用invokevirtual指令来调用的，但是由于它无法被覆盖，没有其它版本，所以无须对方法接收者进行多态选择，又或者说多态选择的结果肯定是唯一的。`在Java语言规范中明确说明了final方法是一种非虚方法`。

`解析调用一定是个静态的过程，在编译期间就完全确定`，在类装载的解析阶段就会把涉及的符号引用全部转变为可确定的直接引用，不会延迟到运行期再去完成。`而分派（Dispatch）调用则可能是静态的也可能是动态的`，根据分派依据的`宗量数`可分为`单分派`和`多分派`。这两类分派方式两两组合就构成了`静态单分派、静态多分派、动态单分派、动态多分派`四种分派情况。

##分派

####静态分派

在开始讲解静态分派前，笔者准备了一段经常出现在面试题中和程序代码：

```Java
package thinking.jvm;

public class StaticDispatch {

	static abstract class Human{
		
	}
	
	static class Man extends Human{
		
	}
	
	static class Women extends Human{
		
	}
	
	public void sayHello(Human guy){
		System.out.println("hello,guy!");
	}
	
	public void sayHello(Man guy){
		System.out.println("hello,gentleman!");
	}
	
	public void sayHello(Women guy){
		System.out.println("hello,lady!");
	}
	
	public static void main(String[] args) {
		Human man = new Man();
		Human wowen = new Women();
		StaticDispatch sd = new StaticDispatch();
		sd.sayHello(man);
		sd.sayHello(wowen);
	}

}
```
运行结果：
```
hello,guy!
hello,guy!
```
以上代码实际上是在考察阅读者对重载的理解程度，对Java稍有经验的程序员看完程序后都能得出正确的运行结果，但为什么会选择执行参数类型为Human的重载呢？在解决这个问题之前，我们先接如下代码定义两个重要的概念：
```Java
Human man = new Man();
```
我们把上面代码中的“Human”称为变量的静态类型（Static Type）或者处观类型（Apparent Type），后面的“Man”则称为变量的实际类型（Actual Type），静态类型和实际类型在程序中都可以发生一些变化，区别是静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型是在编译期可知的；而实际类型变化的结果在运行期才可确定，编译器在编译程序的时候并不知道一个对象的实际类型是什么。如下代码：
```Java
//实际类型变化
Human man = new Man();
man = new Women();
//静态类型变化
sd.sayHello((Man) man);
sd.sayHello((Women) man);
```
解释了这两个概念，再回到StaticDispatch.java代码中。main()里面的两次sayHello()方法调用，在方法接收者已经确定是对象"sd"的前提下，使用哪个重载版本，就完全取决于传入参数的数量和数据类型。代码中刻意地定义了两个静态类型相同、实际类型不同的变量，`但虚拟机（准确地说是编译器）在重载时是通过参数的静态类型而不是实际类型作为判定依据`。并且静态类型是编译期可知的，所以在编译阶段，Javac编译器就根据参数类型决定使用哪个重载版本，所以选择了sayHello(Human)作为调用目标，并把这个方法的符号引用写到了main()方法里的两条invokevirtual指令的参数中。

`所有依赖静态类型来定位方法执行版本的分派动作，都称为静态分派`。静态分派典型应用就是方法重载。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的。另外，编译器虽然能确定出方法的重载版本，但在很多情况下这个重载版本并不是“唯一的”，往往只能确定一个“更加合适的”版本。产生这种模糊结论的主要原因是字面量不需要定义，所以字面量没有显式的静态类型，它的静态类型只能通过语言上的规则去理解和推断。下面代码演示了何为“更加合适的”版本。
```Java
package thinking.jvm;

import java.io.Serializable;

public class Overload {

	public static void sayHello(Object arg){
		System.out.println("hello Object");
	}
	
	public static void sayHello(int arg){
		System.out.println("hello int");
	}
	
	public static void sayHello(long arg){
		System.out.println("hello long");
	}
	
	public static void sayHello(Character arg){
		System.out.println("hello Character");
	}
	
	public static void sayHello(char arg){
		System.out.println("hello char");
	}
	
	public static void sayHello(char... args){
		System.out.println("hello char ...");
	}
	
	public static void sayHello(Serializable arg){
		System.out.println("hello Serializable");
	}
	
	public static void sayHello(Comparable<Character> arg){
		System.out.println("hello comparable");
	}
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		sayHello('a');

	}
}
```
输出：
```
hello char
```

另外还有一点可能比较容易混淆：解析与分派这两者之间的关系并不是二选一的排他关系，它们是在不同层次上去筛选和确定目标方法的过程。例如，前面说过静态方法会在类加载期就进行解析，而静态方法显然也是可以拥有重载版本的，选择重载版本的过程是通过静态分派完成的。


