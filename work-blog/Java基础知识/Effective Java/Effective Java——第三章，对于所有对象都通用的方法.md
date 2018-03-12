### 第三章，对象通用的方法覆盖技巧
>尽管Object类的设计是一个具体类，但是设计它也是为了扩展。它的所有非final方法（equals、hashCode、toStrng、clone和finalize）都有明确的通用约定，因为Java中很多类都是遵照这个设计约定进行，不然就无法和这些约定的类（HashMap和HashSet）结合相互工作。

##### 第一条：覆盖equals时遵守的通用约定
覆盖equals方法看起来很简单，但是如果覆盖方式不对会导致严重的错误，最容易避免这类问题的办法就是不覆盖equals方法，在这种情况下，类的每个实例都只与它自身相等。如果满足以下任何一个条件即可：
- 类的每个实例本质上都是唯一的。
- 不关心类是否提供“逻辑相等”的测试功能。
- 超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。
- 类是私有的或包级私有的，可以确定它的equals方法永远不被调用。

什么时候覆盖呢？**在类出现特有比较逻辑相等概念的时候进行覆盖。**
在覆盖equals时，有以下约定：
- 自反性。对于任何非null的引用值x，e.equals(x)必须返回true。
- 对称性。对于任何非null的引用值x和y，当且仅当y.equals(x)返回true时，x.equals(y)必须返回true。
- 传递性。对于任何非null值引用x、y和z，如果x.equals(y)返回true，并且y.equals(z)也返回true，那么x.equals(z)也必须返回true。
- 一致性。对于任何非null值引用x和y，只要equals比较操作在对象中所有的信息都没有被修改，多次调用x.equals(y)就一致返回true，或者一致返回false。
- 对于任何非null值引用x，x.equals(null)必须返回false。

##### 第二条：覆盖equals时总要覆盖hashCode
在每个覆盖了equals的方法中都必须覆盖hashCode。如果不这样就违背了Object.hashCode的通用约定，从而导致该类无法与散列集合一起正常工作。比如HashMap、HashSet。约定如下：
- 在应用程序执行期间，只要对象的equals方法比较所用到的信息没有被修改，那么针对这同一对象调用多次hashCode都必须返回同一数字。
- 如果两个对象调用equals是相等的，那么调用hashCode也必须产生相同的整数结果。
- 如果调用两个对象的equals方法比较是不相等的，那么调用任一个对象的hashCode方法不一定要产生不同的整数结果。

```java
public static void main(String[] args) {
    NutritionFacts facts1 = new NutritionFacts(0,0,0);
    NutritionFacts facts2 = new NutritionFacts(0, 0, 0);
    HashMap<NutritionFacts,String> sets = new HashMap<>();
    sets.put(facts1,"123");
    System.out.print(sets.get(facts2));
}
```
在上面的例子中，结果是null。就是因为尽管fact1和fact2在equals比较上是相等的，但是他们的hashCode是不同的。

一个好的散列函数通常倾向于“为不相等的对象产生不相等的散列码”，理想情况下：散列函数应该把集合中不相等的实例均匀分布到所有可能的散列值上，这里就设计到散列计算的常见方法：

1.  直接寻址法：取关键字或关键字的某个线性函数值为散列地址。即H(key)=key或H(key) = a•key + b，其中a和b为常数（这种散列函数叫做自身函数）
2. 数字分析法：分析一组数据，比如一组员工的出生年月日，这时我们发现出生年月日的前几位数字大体相同，这样的话，出现冲突的几率就会很大，但是我们发现年月日的后几位表示月份和具体日期的数字差别很大，如果用后面的数字来构成散列地址，则冲突的几率会明显降低。因此数字分析法就是找出数字的规律，尽可能利用这些数据来构造冲突几率较低的散列地址。
3. 平方取中法：取关键字平方后的中间几位作为散列地址。
4. 折叠法：将关键字分割成位数相同的几部分，最后一部分位数可以不同，然后取这几部分的叠加和（去除进位）作为散列地址。
5. 随机数法：选择一随机函数，取关键字的随机值作为散列地址，通常用于关键字长度不同的场合。
6. 除留余数法：取关键字被某个不大于散列表表长m的数p除后所得的余数为散列地址。即 H(key) = key MOD p, p<=m。不仅可以对关键字直接取模，也可在折叠、平方取中等运算之后取模。对p的选择很重要，一般取素数或m，若p选的不好，容易产生同义词。

##### 第三条：始终覆盖toString方法
我们都知道Object给我们提供了toString方法，但是在多数情况下通过System.out.print方法打印出来的并不是我们想要的值，所以我们要在实际的开发中**toString方法应该返回对象中包含的关键信息。**

##### 第四条：谨慎地覆盖clone方法
我们都知道Cloneable接口的目的是为了创建对象方便，表明这个对象是允许克隆的，但是在Colneable接口中并没有提供对应的clone方法。

##### 第五条：考虑实现Comparable接口
compareTo方法并没有早Object中声明，它是Comparable接口中唯一的方法，compareTo方法不但允许进行简单的等同性比较，而且允许执行顺序比较。
compareTo方法的通用约定与equals方法相似：将这个对象与指定的对象进行比较，当该对象小于、等于或大于指定对象的时候，分别返回一个负整数、零活正整数。如果由于指定对象的类型而无法与该对象进行比较，则抛出ClassCastException异常。
```java
@Override
public int compareTo(NutritionFacts o) {
    if(this.fat > o.fat)return 1;
    if(this.fat <= o.fat)return -1;
    return 0;
}
```
