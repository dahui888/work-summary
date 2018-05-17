#### 剑指Offer——二维数组的查找
**题目：**
>时间限制：1秒 空间限制：32768K
>
>在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

**分析：**
因为是有序的序列，并且始终右下角最大，所以从右下角开始比较，如果比右下角大，则返回false不存在。反之向左遍历，又因为是有序，可以使用折半查找。

**编码：**
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
    	if(array == null || array.length == 0)return false;
    	int row = array.length-1;
    	int column = array[0].length-1;
    	while(row >= 0){
    		int left = 0,right = column;
    		while(left <= right){
        		int mid = (left + right) / 2;
    			if(target == array[row][mid]){
    				return true;
    			}else if(target < array[row][mid]){//左边
    				right = mid-1;
    			}else{
    				left = mid + 1;
    			}
    		}
    		row--;
    	}
    	return false;
    }
}
```

##### 补充
```java
/* 思路
* 矩阵是有序的，从左下角来看，向上数字递减，向右数字递增，
* 因此从左下角开始查找，当要查找数字比左下角数字大时。右移
* 要查找数字比左下角数字小时，上移
*/
public class Solution {
    public boolean Find(int [][] array,int target) {
int len = array.length-1;
        int i = 0;
        while((len >= 0)&& (i < array[0].length)){
            if(array[len][i] > target){
                len--;
            }else if(array[len][i] < target){
                i++;
            }else{
                return true;
            }
        }
        return false;
    }
}
```

算法还是很重要，所以要利用零碎时间慢慢补习。