### 剑指Offer——用两个栈实现队列
>题目描述
用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

分析：
栈和队列是数据结构中常见的两种数据结构，在Java中提供了Stack、Queue进行支持。栈特点是先进后出，队列的特点是先进先出。所以我们的思路就是：

>用其中一个栈stack1存储数据，执行push操作。然后当执行出栈的操作时候，将保存的数据出栈存储到stack2中。每执行一次出栈就将stack2进行一次出栈操作。当stack2位空的时候，在将stack1的元素push到stack2中。

**代码实现**
```java
/**
 * 入队
 * @param node
 */
public static void push(int node){
    stack1.push(node);
}

/**
 * 出队
 * @return
 */
public static int pop(){
    if(stack1.empty() && stack2.empty()){
        throw new RuntimeException("Queue is empty");
    }

    if(stack2.empty()){
        while(!stack1.empty()){
            stack2.push(stack1.pop());
        }
    }

    return stack2.pop();
}
```