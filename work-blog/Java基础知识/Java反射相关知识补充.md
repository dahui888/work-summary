### Java反射相关知识补充

#### 一、如何通过反射获取某个成员变量值。

首先通过反射获取对应的Field，然后调用Field.get(Object)获取对应Object的字段值。

```java
public static void main(String[] args) {
    Student stu = new Student("dsw", 12, "20160613");
    Class<? extends Student> clazzStudent = stu.getClass();
    Field[] fields = clazzStudent.getDeclaredFields();
    for(Field field : fields){
        field.setAccessible(true);
        try {
            System.out.print(field.getName() + ":" + field.get(stu));
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

注意：

1. 针对private修饰的成员，需要设置该Field的可获取。field.setAccessible(true)。
2. getFields()获取所有的public属性的Field，getDeclaredFields()获取所有的属性。

#### 二、如何获取类对象成员的相关字段

首先通过类的反射获取对应类对象的Field，然后获取该Field的值。通过该值obj.getClass()获取对应的Class对象，然后在进行操作。

```java
public static void main(String[] args) {
    Student stu = new Student("dsw", 12, "20160613",new Score("英语", "001"));
    Class<? extends Student> clazzStudent = stu.getClass();
    try {
        Field field = clazzStudent.getDeclaredField("score");
        field.setAccessible(true);
        Object objectScore = field.get(stu);
        Field[] fields = objectScore.getClass().getDeclaredFields();
        for(Field temfield : fields){
            temfield.setAccessible(true);
            System.out.println(temfield.getName() + ":" + temfield.get(objectScore));
        }

    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    } catch (SecurityException e) {
        e.printStackTrace();
    } catch (IllegalArgumentException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}
```

#### 三、Class类的源码

Class类是没有public类型的构造方法，所以我们无法通过new来进行创建一个Class对象。给我们提供了以下方式来获取构建一个Class对象：
-  obj.getClass()
- 	类名.class
- 	Class.forName

通过Class类的生命：
```java
public final
    class Class<T> implements java.io.Serializable,
                              java.lang.reflect.GenericDeclaration,
                              java.lang.reflect.Type,
                              java.lang.reflect.AnnotatedElement {}
```
可以看到Class类是可序列化的类。同时是不可集成修改final类型。

#### 四、泛型的类型擦除使用

##### 1、在泛型为Integer的ArrayList中存放一个String类型的对象。
```java
public static void main(String[] args) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    Method method;
    try {
        method = list.getClass().getMethod("add", Object.class);
        method.invoke(list, "Java反射机制实例在类型擦除中的使用。");
        System.out.println(list.get(0));
    } catch (NoSuchMethodException | SecurityException | IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
}
```
##### 2、在泛型为String的ArrayList中存放一个Activity类型的对象。
```java
ArrayList<String> list = new ArrayList<>();
Method method;
try {
    method = list.getClass().getMethod("add", Object.class);
    method.invoke(list, new MainActivity());
} catch (NoSuchMethodException e) {
    e.printStackTrace();
} catch (SecurityException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (IllegalArgumentException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

#### 五、反射能干什么？

- 在运行时判断任意一个对象所属的类；

- 在运行时构造任意一个类的对象；

- 在运行时判断任意一个类所具有的成员变量和方法；

- 在运行时调用任意一个对象的方法；

- 生成动态代理。