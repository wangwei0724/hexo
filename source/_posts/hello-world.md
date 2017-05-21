---
 title: 剑指offer javascript实现
 categories:
	- 剑指offer
---


## 剑指offer javascript实现

>1.输入一个链表，从尾到头打印链表每个节点的值

```
function printListFromTailToHead(head){
    var node= head;
    var list=[];
    while(node!=null){
        list.unshift(node.val);
        node=node.next;
    }
    return list;
}
module.exports = {
    printListFromTailToHead : printListFromTailToHead
}; 
```
>2.请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

```
function replaceSpace(str){
    var arr = str.split(' ');
    str = arr.join('%20');
    return str;
}
module.exports = {
    replaceSpace : replaceSpace
};
```
>3.用两个栈来实现一个队列，完成队列的Push和Pop操作。 队列中的元素为int类型。

```
var stack = [];
var stack_temp = [];
function push(node)
{
    stack.push(node);
}
function pop()
{   
    if(stack_temp.length==0){//若不为空直接返回之前一步最后压入的元素即可
        while(stack.length>0){
            stack_temp.push(stack.pop());
        }
    }
    return stack_temp.pop();

}
module.exports = {
    push : push,
    pop : pop
};
```
>4.把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。
NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。

```
//方法1:这是比较low的解决方法，直接遍历数组，通过前后元素的大小比较确定
function minNumberInRotateArray(rotateArray){
    if(rotateArray.length==0){
        return 0;
    }
    for(var i=0;i<rotateArray.length;i++){
        if(rotateArray[i+1]<rotateArray[i]){
            return rotateArray[i+1];
        }
    }
    
}
module.exports = {
    minNumberInRotateArray : minNumberInRotateArray
};
```

```
//方法2效率较高，避免了一个一个比较
function minNumberInRotateArray(rotateArray)
{
    if(rotateArray.length==0){
        return 0;
    }
	var start= 0;
    var end = rotateArray.length-1;
    var middle = 0;
    while(rotateArray[start]>=rotateArray[end]){
    	if(start==end-1){
            middle = end;
    		break;
    	}
    	middle=Math.floor((start+end)/2);
        if(rotateArray[middle]>=rotateArray[start]){
            start=middle;
        }
        else if(rotateArray[middle]<rotateArray[start]){
            end=middle;
        }

    }
    return rotateArray[middle];
}
module.exports = {
    minNumberInRotateArray : minNumberInRotateArray
};
```
>5.大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。

>tip:这道题其实可以用递归，代码或许会更简洁，但是递归在这里有严重的效率问题，比如说求f(10),必须先求f(8)和f(9)，f(9)又要计算f(7)和f(8)，可以发现f(8)被重复计算。

```
function Fibonacci(n){
    var sum=0;
    var first=1;
    var second=1;
    if(n==1||n==2){
        sum = 1;
    }else{
        var i=3;
        while(i<=n){
            sum = first+second;
            first = second;
            second = sum;
            i++;
        }
    }
    return sum;
}
module.exports = {
    Fibonacci : Fibonacci
};
```
>6.一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。

>解题思路：青蛙最后一步可以跳一级或者两级;那么假定第一次跳的是一阶，那么剩下的是n-1个台阶，跳法是f(n-1);假定第一次跳的是2阶，那么剩下的是n-2个台阶，跳法是f(n-2);所以最后一步的跳法f(n)=f(n-1)+f(n-2)

```
function jumpFloor(number){
    if(number==0){
    	return 0;
    }else if(number>0&&number<3){
    	return number;
    }else{
    	var first =1;
    	var second = 2;
    	var sum;
        var i=3;
    	while(i<=number){
    		sum = first+second;
    		first = second;
    		second = sum;
            i++
    	}
    	return >;
    }
}
module.exports = {
    jumpFloor : jumpFloor
};
```
>7.一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法

>解题思路：若将n级台阶的跳法设为f(n)，可以想到f(n)=f(n-1)+f(n-2)+f(n-3)....+f(1)，而f(n-1)=f(n-2)+f(n-3)....+f(1)，所以f(n)=2f(n-1)。所以代码思路就很清晰了

```
function jumpFloorII(number){
    if(number == 0){//特殊情况单独拿出来讨论一下
        return 0;
    }else{
       return Math.pow(2, number-1) 
    }   
}
module.exports = {
    jumpFloorII : jumpFloorII
};
```
>8.我们可以用2 <code>*</code> 1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2 <code>*</code> 1的小矩形无重叠地覆盖一个2 <code>*</code> n的大矩形，总共有多少种方法？

>解题思路：这和上面一样也是一道斐波那契数列的题目，具体思路和第一个青蛙跳台阶差不多

```
function rectCover(number){
	if(number==2||number==1||number==0){
		return number;
	}else{
		var sum;
		var first= 1;
		var second = 2;
		var i=3;
		while(i<=number){
			sum = first+second;
			first = second;
			second = sum;
			i++;
		}
		return sum;
	}
    
}
module.exports = {
    rectCover : rectCover
};
```
>9.输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

```
function reOrderArray(array){
    var first=[];//按顺序存奇数
    var second=[];
    for(var i=0;i<array.length;i++){
        if((array[i]%2)==0){
            second.push(array[i]);
        }else{
            first.push(array[i]);
        }
    }
    first=first.concat(second);
    return first;
}
module.exports = {
    reOrderArray : reOrderArray
};
```

