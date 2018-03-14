###第四章，类和接口
>类和接口是Java程序设计语言的核心，也是Java语言的基本抽象单元。Java语言提供了许多强大的基本元素，供程序员用来设计类和接口。

##### 第一条：使类和成员的可访问性最小
一个设计良好的模块，最重要的一点是要对于外部模块隐藏关键的实现细节，只外漏该外抛的方法。模块之间只通过它们之间的API进行通讯访问。在Java的程序设计语言中，通过访问控制来实现类、接口和成员的可访问性限制，这些修饰符有public、protected和private。**类和接口的修饰符只能是public或默认**

对于顶层接口设计，只有两种可访问的级别：包级私有和公有的。如果用public修饰符声明了顶层类或接口，则它是公有访问级别的。否则是包级私有的。如果类或接口能做成包级私有，就应该做成包级私有，这样发布的版本客户就不会影响客户端的操作。

如果一个包级私有的顶层类只在某个类中内部使用，可以考虑把它设计未嵌套内部类。对于成员（域、方法、嵌套类和嵌套接口）有四中可能的访问级别：
- 私有的（private）：只有在声明该成员的顶层类内部才可以访问这个成员。
- 包级私有的：声明该成员的包内部的任何类都可以访问这个成员，一般也被称之为“缺省访问级别default”，如果没有给成员显示指定访问修饰符，就是这个。
- 受保护的（protected）：声明该成员的类的子类可以访问这个成员，并且包内部的任何类也可以访问。

##### 第二条：在共有类中使用访问方法来替代公有域
```java
class Point {
	public int circleX;
	public int circleY;
}
```
由于这种类的数据可以被直接访问，这些类没有提供封装功能，所以如果没有改变API接口，就无法改变它的数据行为，一般我们的建议是**如果类可以在它所在的包外被访问，就应该主动提供访问的方法来访问。**
```java
class Point {
	private int circleX;
	private int circleY;
	public int getCircleX() {
		return circleX;
	}
	public void setCircleX(int circleX) {
		this.circleX = circleX;
	}
	public int getCircleY() {
		return circleY;
	}
	public void setCircleY(int circleY) {
		this.circleY = circleY;
	}
}
```

##### 第三条：使可变性最小化
不可变类只是实例不能被修改的类，每个实例中包含的字段信息都应该在实例创建的时候进行指定，比如String。不可变类更加易于设计、实现和使用，不容易出错。为了使类成为不可变类，一般遵循下面五条规则：
1. 不要提供任何会修改对象状态的方法
2. 保证类不会被继承扩展
3. 使所有的域都是final类型的
4. 使所有的域都是私有的
5. 确保对于任何可变组件的互斥访问

##### 第四条：组合优先于继承
继承是代码重用的有力手段，但是在Java程序开发汇总，我们更推荐在包的内部使用继承，主要是因为这种方式相对于包外的继承更安全。

与通过组合形式的方法调用不同，继承打破了封装性，子类依赖其父类中特定的功能细节。

##### 第五条：接口优于抽象类
Java程序中提供了接口和抽象类来实现多个实现的类型。明显的区别是抽象类中允许包含某些方法的实现，接口中是不允许的。Java中的类是单继承，搜易抽象类作为类型定义会收到极大的限制。使用接口主要有以下好处：
- 现有类可以很容易被更新，以实现新接口
- 接口的定义是混合类型的立项选择
- 接口允许我们构造非层次结构的类型框架，比如一个人是歌唱家又是作曲家。

我们知道通过接口的多实现特性可以构建多层次的类结构，我们同时也可以通过**对每个接口都提供一个抽象的骨架实现类，把接口和抽象类的优点结合起来。**

##### 第六条：接口只用于定义类型
在我们开发中，我们一定出现过在接口中定义静态常量的情况，这种接口称之为常量接口。
```java
public interface TestClass {
	public static final String AVER_CONST_VALUE = "平均质量";
	public static final String HIGH_CONST_VALUE = "最高质量";
}
```
**常量接口是对接口的不恰当使用。**在Java中如果我们要定义常量，可以使用枚举的方式进行定义。或者通过不可实例化的工具类来导出这些常量，避免通过实现绑定到具体的类上。

##### 第七条：类层次优于标签类
类层次在前面我们已经知道了，那什么是标签类呢？如下面：
```java
class Figure{
	enum Shape{
		RECTANGLE,
		CIRCLE
	};
	final Shape shape;
	double lenght,width;
	double radius;
	public Figure(double lenght, double width) {
		this.lenght = lenght;
		this.width = width;
		this.shape = Shape.RECTANGLE;
	}
	public Figure(double radius) {
		this.radius = radius;
		this.shape = Shape.CIRCLE;
	}
	
	public double area(){
		switch(shape){
		case RECTANGLE:
			return lenght * width;
		case CIRCLE:
			return Math.PI * radius * radius;
		default:
			throw new AssertionError();
		}
	}
}
```
这种标签类存在很多缺点，里面包含了很多样板代码，里面包含了枚举声明、标签句以及条件判断，由于很多乱七八糟的逻辑囊括在一个类中，破坏了可读性，内存占用也响应的增加了。

那么针对标签类我们如何优化呢？这里就可以使用到层次类来进行优化，将标签类转换为层次类首先要为标签类中的每个方法定义一个包含该方法的抽象类，比如area方法。接下来就是为原始标签类定义根类的具体子类。
```java
abstract class Figure{
	abstract double area();
}

class Rectangle extends Figure{
	private double length;
	private double width;
	public Rectangle(double length, double width) {
		super();
		this.length = length;
		this.width = width;
	}
	@Override
	double area() {
		return width * length;
	}
}
class Circle extends Figure{
	private double radius;

	public Circle(double radius) {
		super();
		this.radius = radius;
	}

	@Override
	double area() {
		return Math.PI * radius * radius;
	}
}
```
通过上面的优化体验上来说，就是规划好累的层次，精简类的职能。

##### 第八条：优先考虑静态成员类
定义在一个类中的内部类称之为嵌套类。**嵌套类存在的目的是为了辅助它的外围类，如果嵌套类出现用于外部其它环境中，它就应该被定义成顶层类。**嵌套类有四中：静态内部类、非静态成员类、匿名类和局部类。除了静态内部类外，其它的三种都被称之为内部类。

静态成员类是最简单的一种嵌套类。与普通的类相差不大，只是定义在另一个类的内部，它可以访问外围类的所有成员，包含那些私有成员，它和静态成员一样。	

非静态成员类的每个实例都隐藏着与外围类的一个实例引用。非静态成员类可以调用外围类的方法，或者通过修饰后的this获取外围类的实例引用。**如果嵌套类的实例可以在它的外围类的实例之外独立存在，这个类就必须是静态成员类。**

如果声明成员类不要求访问外部实例，就要始终把static修饰符放到它的声明中，使它成为静态成员类，而不是非静态成员类。如果省略了static修饰符，则每个实例都将包含一个额外指向外围对象的引用，保存这份引用需要耗费时间和空间。

匿名类不同于任何Java中的其它语法单元，它不与其他成员一起被声明，而是在使用的同时被声明和实例化。

##### 总结
通过这一章的接口和类的比较，增加了对接口和类的更深入了解，同时对类的设计有了更进一步的思考。主要体现在以下几个方面：
1. 类和接口的设计要尽量封闭，减少外抛的接口，使之可访问性降低，更加稳定；
2. 对公有域（成员变量）尽量使用方法来进行获取设置，不要直接进行操作，以便后续可以丰富完善；
3. 类和接口设计要精简、干练，避免冗余形成标签类；
4. 多使用接口进行丰富类的功能，降低类的层次；
5. 内部类的作用是为了辅助外围内，不要本末倒置；