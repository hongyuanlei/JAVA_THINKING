#Java I/O 与装饰器模式

装饰器模式定义：

>装饰模式是在不必改变原类文件和使用继承的情况下，动态的扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

我们注意其中的几点：
*    1、不改变原类文件；
*    2、不使用继承；
*    3、动态扩展；

上述三句话一语道出了装饰器模式的特点，下面给出装饰器模式的类图：

![装饰器模式类图](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/%E8%A3%85%E9%A5%B0%E5%99%A8%E6%A8%A1%E5%BC%8F%E7%9A%84%E7%B1%BB%E5%9B%BE.jpg)

从图中可以看到，我们装饰的是一个接口的任何实现，而这些实现也包括装饰器本身，装饰器本身也可以再被装饰。

另外，这个类图只是装饰器模式的完整结构，但其实里面有很多可以变化的地方：

*    1、Component接口可以是接口也可以是抽象类，甚至是一个普通的父类（这个强烈不推荐，普通的类作为继承体系的超级父类不易于维护）。
*    2、装饰器的抽象父类Decorator并不是必须的。

那么我们将上述标准的装饰器模式用Java代码诠释下：

首先是待装饰的接口Component。
```Java
package thinking.decorator;

public interface Component {

	void method();
}
```

接下来便是我们的一个具体的接口的实现类，也就是俗称的原始对象，或者说待装饰对象。
```Java
package thinking.decorator;

public class ConcreteComponent implements Component {

	@Override
	public void method() {
		System.out.println("原来的方法");
	}

}
```

下面便是我们的抽象装饰器的父类，它主要是为装饰器定义了我们需要装饰的目标是什么，并对Component进行基础的装饰。
```Java
package thinking.decorator;

public abstract class Decorator implements Component{

	protected Component component;
	
	public Decorator(Component component){
		super();
		this.component = component;
	}
	
	public void method(){
		component.method();
	}
}
```

再来便是我们具体的装饰器A和装饰器B。
```Java
package thinking.decorator;

public class ConcreteDecoratorA extends Decorator {

	public ConcreteDecoratorA(Component component) {
		super(component);
	}

	public void methodA(){
		System.out.println("被装饰器A扩展的功能");
	}
	
	public void method(){
		System.out.println("针对该方法加一层A包装");
		super.method();
		System.out.println("A包装结束");
	}
}
```

```Java
package thinking.decorator;

public class ConcreteDecoratorB extends Decorator {

	public ConcreteDecoratorB(Component component) {
		super(component);
	}

	public void methodB(){
		System.out.println("被装饰器B扩展的功能");
	}
	
	public void method(){
		System.out.println("针对该方法加一层B包装");
		super.method();
		System.out.println("B包装结束");
	}
}
```

下面给出我们的测试类。我们针对多种情况进行包装。
```Java
package thinking.decorator;

public class Main {

	public static void main(String[] args){
		//原来的对象
		Component component = new ConcreteComponent();
		System.out.println("------------------------");
		//原来的方法
		component.method();
		System.out.println("------------------------");
		ConcreteDecoratorA concreteDecoratorA = new ConcreteDecoratorA(component);
		concreteDecoratorA.method();
		concreteDecoratorA.methodA();
		System.out.println("------------------------");
		ConcreteDecoratorB concreteDecoratorB = new ConcreteDecoratorB(component);
		concreteDecoratorB.method();
		concreteDecoratorB.methodB();
		System.out.println("------------------------");
		concreteDecoratorB = new ConcreteDecoratorB(concreteDecoratorA);
		concreteDecoratorB.method();
		concreteDecoratorB.methodB();
	}
}
```
下面看下我们运行的结果，到底是产生了什么效果。
```
------------------------
原来的方法
------------------------
针对该方法加一层A包装
原来的方法
A包装结束
被装饰器A扩展的功能
------------------------
针对该方法加一层B包装
原来的方法
B包装结束
被装饰器B扩展的功能
------------------------
针对该方法加一层B包装
针对该方法加一层A包装
原来的方法
A包装结束
B包装结束
被装饰器B扩展的功能

```

下面给出IO包中所涉及的类的类图，可以和上面的标准装饰器模式对比一下：

![IO包中所涉及的类的类图](https://raw.githubusercontent.com/hongyuanlei/JAVA_THINKING/master/images/IO%E5%8C%85%E4%B8%AD%E6%89%80%E6%B6%89%E5%8F%8A%E7%9A%84%E7%B1%BB%E7%9A%84%E7%B1%BB%E5%9B%BE.jpg)
