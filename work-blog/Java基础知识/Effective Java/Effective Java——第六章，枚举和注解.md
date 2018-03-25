#### 第六章：枚举和注解
>在Java1.5版本中增加了两个新的引用类型，分别是枚举类型和注解类型。

##### 第一条：用enum代替int常量
枚举类型是指由一组固定的常量组成的合法值的类型集合。本条规则主要是针对在实际项目中定义的int常量可以使用enum来替代。这是因为**Java的枚举类型是功能十分齐全的类。**
```java
public enum Color {
	RED(1),
	BLUE(2),
	GREEN(3),
	BLACK(4);
	
	private int type;
	public void getType(){
		System.out.println("getType");
	}
	
	public Color(int type)
	{
		this.type = type;
	}
}
```
通过上面的介绍我们知道Java的枚举类型是一个类，所以我们可以给它定义构造函数，枚举值就是类对象，比如上面的RED就是Color的一个类对象。

前面我们说过通过枚举实现单例模式，所以我们上面的代码中构造函数就会报错

	Illegal modifier for the enum constructor; only private is permitted.

这也说明，我们必须创建private类型的构造函数。
```java
public enum Color {
	RED(1),
	BLUE(2),
	GREEN(3),
	BLACK(4);
	
	private int type;
	public void getType(){
		System.out.println("getType");
	}
	
	private Color(int type)
	{
		this.type = type;
	}
	
	public static void main(String []args){
		for(Color tem : Color.values()){
			System.out.println(tem.type
					+ "--" + tem.name() + "--" + tem.ordinal());
		}
	}
}
```
运行结果：
```java
1--RED--0
2--BLUE--1
3--GREEN--2
4--BLACK--3
```
从上面可以看到，ordinal方法是获取的枚举值在常量中的position位置。

##### 第二条：用实例域代替序数
在上面我们介绍了ordinal方法获取的值是枚举值在常量中的位置，但是这样不容易维护，所以要定义实例域来表示它们的位置。例子如上面所示。

##### 第三条：注解优先于命名模式
这里主要距离了命名模式的几个缺点：
- 比如JUnit框架要求测试方法以test开头，所以通过命名模式文字拼写容易出错。
- 无法确保用于对应的程序上；
- 没有提供将参数值与程序关联的方法。

所以这个时候通过注解就可以强制约定，避免错误。

##### 第四条：坚持使用Override注解
Override注解表明这个方法覆盖了超类中的方法，使用这个方法可以在编译器的编译的时候避免错误。