#### 第七章，方法
> 本章讨论总结的是方法设计的几个方面：如何处理方法的参数和返回值，如何设计方法签名，如何为方法编写文档。本章的焦点也集中在代码的可用性、健壮性和灵活性上。

##### 第一条：检查参数的有效性
在开发中，绝大多数的方法和构造器对于传递给他们的参数都会有某些限制。比如索引值必须是非负数、对象不能为null等等。**我们应该在方法的注释中说明这些限制情况，并在方法体的开头出检查参数，以强制事假这些限制。**
```java
/**
* 
* @param openId,非空
* @param openKey，关键字
* @param apiUrl，url
* @return，加密后url
*/
public static String generateSign(String openId, String openKey, String apiUrl){
  if(openId == null || "".equals(openId)){
      throw new NullPointerException("OpenId is null");
  }
  return "";
}
```
对于未被导出的方法，作为包的创建者，我们可以通过assert断言机制来检查参数的合法性。
```java
/**
* 
* @param openId,非空
* @param openKey，关键字
* @param apiUrl，url
* @return，加密后url
*/
public static String generateSign(String openId, String openKey, String apiUrl){
  assert openId != null;
  assert openKey != null && openKey.length() == 18;
  return "";
}
```
这个不同于一般的断言机制，如果断言失败，将会抛出AssertionError。如果他们没有起到作用，本质上也不会有成本开销。

##### 第二条：谨慎设计方法签名
本条中主要是针对API设计的总结：
- 谨慎选择方法的名称，方法名称应该符合习惯，并且易于理解。同一个包中的方法名称命名风格一致。
- 不要过度追求便利方法。方法应该尽其所能，方法太多会导致类复杂，除非一个代码片段经常用到才抽取出来单独成为方法。
- 避免过长的参数列表。

##### 第三条：慎用重载
- 重载（overload）：对于类的方法（包括从父类中继承的方法），方法名相同，参数列表不同的方法之间就构成了重载关系。
- 覆盖 (override)：也叫重写，就是在当父类中的某些方法不能满足要求时，子类中改写父类的方法。当父类中的方法被覆盖了后，除非用super关键字，否则就无法再调用父类中的方法了。

我们写一个用来判定集合类型的类。
```java
public class CollectionClassifier {
	
	public static String classify(Set<?> set){
		return "Set";
	}
	
	public static String classify(List<?> list){
		return "List";
	}
	
	public static String classify(Collection<?> collection){
		return "Unknown";
	}

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		Collection<?>[] clCollections = {new HashSet<String>(),
				new ArrayList<>(),new HashMap<String,String>().values()};
		for(Collection<?> c :clCollections){
			System.out.println(classify(c));
		}
	}
}
```
但是它直接打印出来的结果却是三个Unknown。这是因为调用哪个重载方法是在编译时决定的，对于for循环中的三次迭代，参数编译时期的类型都是Collction，所以使用的重载方法都是第三个。

##### 第四条：慎用可变参数
可变参数是在jdk1.5中新增的语法。可变参数方法接收0个或多个指定类型的参数，可变参数的机制是通过创建一个数组，数组的大小为在调用位置所传递的参数数量，然后将参数值传递到数组中，最后将数组传递给方法。
```java
private static int sum(int... args){
    if(args.length == 0){
        return 1;
    }
    int sum=0;
    for(int temp : args){
        sum+=temp;
    }
    return sum;
}
```

##### 第五条：返回零长度的数组或者集合，而不是null。
保持自己的方法返回非null类型数据，可以避免很多冗余的参数检查。

##### 第六条：为所有导出的API元素编写文档注释
利用Javadoc来为API生成说明文档。为了正确编写API文档，必须在每个被导出的类、接口、构造器、方法之前增加一个文档注释。