### Effective Java——第二章，创建和销毁对象
>本章的主题是创建和销毁对象：何时以及如何创建对象，何时以及如何避免创建新对象，如何确保它们能够适时地销毁，以及如何管理对象销毁之前必须进行的各种清理动作。

##### 第一条：考虑用静态工厂方法代替构造器
类通过提供一个公有的**静态工厂方法**来返回一个实例对象，而不是通过构造器，有以下优势：

**1、静态工厂方法与构造器相比的第一大优势是有名称**

我们知道在一些类中会出现包含众多参数的构造器，这些构造器的参数使我们在使用的时候非常容易出现迷惑，用错参数之类的。而通过静态工厂方法，我们可以慎重选择比较区别度的名称来定义，这样就可以避免这种问题。

**2、静态工厂方法可以在每次调用它们时不必都创建一个新对象**

通过静态工厂方法，我们可以使用预先构建好的实例，或将构建好的实例缓存起来进行重复利用，从而避免不必要的对象创建，这点的思想跟享元模式很像，如果针对创建过程比较复杂的对象，这样使用缓存能极大提高效率。

**3、静态工厂方法可以返回原返回类型的任何子类型对象**

静态工厂方法可以选择返回对象的类型，这样就对我们实现API的隐藏有很好的灵活性，非常适用于基于接口的框架中。

**4、静态工厂方法创建参数化类型实例的时候，使代码变得简洁**


##### 第二条：遇到多个构造器参数时要考虑使用建造器
在设计模式一栏中，我们介绍了**建造者模式**，这里的第二条就是针对多参数创建对象的情况建议使用建造器模式。
```java
public class NutritionFacts {
	private int servingSize;
	private int servings;
	private int fat;
	
	public static class Builder{
		private int servingSize;
		private int servings;
		private int fat;
		public Builder(int fat){
			this.fat = fat;
		}
		
		public Builder servingSize(int servingSize){
			this.servingSize = servingSize;
			return this;
		}
		
		public Builder servings(int servings){
			this.servings = servings;
			return this;
		}
		
		public NutritionFacts builder(){
			return new NutritionFacts(this);
		}
	}
	
	private NutritionFacts(Builder builder){
		servingSize = builder.servingSize;
		servings = builder.servings;
		fat = builder.fat;
	}
}
```

##### 第三条：用私有构造器或者枚举类型强化Singleton属性。
本条规则描述的就是单例模式的实现方式，建议使用枚举类型创建单例。
```java
public enum Singleton {
	INSTANCE;
	
	public Singleton instance(){
		return this;
	}
}
```
##### 第四条：通过私有构造器强化不可实例化的能力
比如在我们实际开发过程中，所创建的很多工具类，这些工具类中一般都包含很多static静态方法，就不需要new一个对象，然后调用。这里我们就可以通过私有构造函数来防止实例化。
```java
public class UtilityClass {
	private UtilityClass(){
		//通过私有构造函数进行私有化
	}
}
```
##### 第五条：避免创建不必要的对象
尽量使用静态工厂方法而不是构造器来创建对象，重用那些不可修改的的对象。比如下面的一个反例：
```java
public boolean isBabyBoomer(){
    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal.set(2018, 3, 5);
    Date boomStart = gmtCal.getTime();
    gmtCal.set(1992, Calendar.JANUARY, 2, 0, 0);
    Date boomEnd = gmtCal.getTime();
    return true;
}
```
在上面的例子中，isBabyBoomer每次调用的时候都会新建一个Calendar、两个Date对象、一个TimeZone对象。这是非常不必要的，优化后的如下：
```java
public class UtilityClass {
	
	private static final Date BOOM_START;
	private static final Date BOOM_END;
	
	static{
		Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
		gmtCal.set(2018, 3, 5);
		BOOM_START = gmtCal.getTime();
		gmtCal.set(1992, Calendar.JANUARY, 2, 0, 0);
		BOOM_END = gmtCal.getTime();
	}

	public boolean isBabyBoomer(){
		return BOOM_END.compareTo(BOOM_START) >=0;
	}
}
```
通过引入静态代码块保证了只有在类第一次加载的时候创建这些实例，而不是每次调用isBabyBoomer方法都创建这些实例。改进后的形式对于频繁调用的方法效率提高会很多，但是如果方法永不被调用，就没必要初始化常量了。这里可以采用延时初始化策略，在需要的时候进行初始化。

这里补充一个概念**自动装箱**，在JDK1.5中允许程序员将基本数据类型和装箱类型混用，按需要自动装箱和拆箱，自动装箱使得基本类型和装箱类型关系越来越模糊，但是效率远不及基本类型。**所以要警惕无意识的自动装箱，优先使用基本数据类型。**

##### 第六条：清除过期的对象引用
我们都知道Java程序一般不需要用户主动进行垃圾回收处理，但是这并不代表我们不需要思索垃圾回收内存管理的事情。其实不然，比如下面的例子：
```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITAL_CAPACITY = 16;
	
	public Stack(){
		elements = new Object[DEFAULT_INITAL_CAPACITY];
	}
	
	public void push(Object obj){
		ensureCapacity();
		elements[size++] = obj;
	}
	
	public Object pop(){
		if(size ==0)throw new EmptyStackException();
		return elements[size--];
	}

	private void ensureCapacity() {
		if(elements.length >= DEFAULT_INITAL_CAPACITY){
			elements = Arrays.copyOf(elements, 2*size + 1);
		}
	}
}
```
在这个例子中，并没有明显的错误。不严格的讲，这段程序有一个“内存泄漏”，体现在一个栈先增长在收缩，那么从栈中弹出来的对象是不会主动被垃圾回收器回收，这是因为栈中维护着这些过期对象的过期引用，**过期应用指不会被解除的引用**。在本例中，所有elements数组的活动部分之外都是引用过期的。一般这类问题通过赋值null来清楚。
```java
public Object pop(){
    if(size ==0)throw new EmptyStackException();
    Object obj = elements[size--];
    elements[size] = null;
    return obj;
}
```
所以在工作中我们要警惕内存泄漏：
- 只要类是自己管理内存，我们就要注意内存泄漏
- 注意缓存是否有内存泄漏
- 注意第三方的监听器和其它回调，这里可以使用弱引用。

#### 总结
在本章中讲解了到很多避免创建新对象的开发技巧，但是并不是意味着“永远不要创建新对象，创建新对象的代价很大”，相反，对于小对象的构造器只是做了很多少量显示工作，所以针对耗费小的可以进行创建一些新对象，提升程序的清晰性、简洁性。

反之通过自己的对象池（Object Pool）来避免维护新对象的创建并不可可取，除非针对非常重量级的对象创建。正在正确使用对象池的经典案例就是数据库的连接池，建立数据库的连接代价非常昂贵，隐刺重用对象很重要。如果无脑使用，肯定会增加内存占用，代码也会多很多逻辑，现在的JVM具有高度优化的垃圾回收器，其性能很容易超过轻量级对象的对象池。