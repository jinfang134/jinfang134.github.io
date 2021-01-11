---
title: javascript数组常用方法总结
date: 2020-12-05 19:11:15
author: zuo_ji
categories:
    - 编程技术
tags: 
  - js
  - 前端

---




## 数组常用的api
数组是javascript当中最常用的一种数据结构，熟练掌握其常用的api的用法是一项前端基本技能.

###  pop
> 删除原数组最后一项，并返回删除元素的值；如果数组为空则返回undefined 
```js
var a = [1,2,3,4,5]; 
var b = a.pop(); //a：[1,2,3,4]   b：5 //不用返回的话直接调用就可以了
```
<!-- more -->
### push
>  将参数添加到原数组末尾，并返回数组的长度 
```js
var a = [1,2,3,4,5]; 
var b = a.push(6,7); //a：[1,2,3,4,5,6,7]   b：7 
```

### concat
> 返回一个新数组，是将参数添加到原数组中构成的 
```js
var a = [1,2,3,4,5]; 
var b = a.concat(6,7); //a：[1,2,3,4,5]   b：[1,2,3,4,5,6,7] 
```

### splice(start,deleteCount,val1,val2,...)
>  从start位置开始删除deleteCount项，并从该位置起插入val1,val2,... 
```js
var a = [1,2,3,4,5]; 
var b = a.splice(2,2,7,8,9); //a：[1,2,7,8,9,5]   b：[3,4] 
var b = a.splice(0,1); //同shift a: [2,7,8,9,5],b :[1]
a.splice(0,0,-2,-1); var b = a.length; //同unshift a: [-2, -1, 2, 7, 8, 9, 5], b: 7
var b = a.splice(a.length-1,1); //同pop a: [-2, -1, 2, 7, 8, 9],  b=5
a.splice(a.length,0,6,7); var b = a.length; //同push 
```
### reverse
> 将数组反序 
```js
var a = [1,2,3,4,5]; 
var b = a.reverse(); //a：[5,4,3,2,1]   b：[5,4,3,2,1] 
```

### sort(orderfunction)
>   按指定的参数对数组进行排序 
```js
var a = [1,2,3,4,5]; 
var b = a.sort(); //a：[1,2,3,4,5]   b：[1,2,3,4,5] 
```

### slice(start,end)
>   返回从原数组中指定开始下标到结束下标之间的项组成的新数组 
```js
var a = [1,2,3,4,5]; 
var b = a.slice(2,5); //a：[1,2,3,4,5]   b：[3,4,5] 
```

### join(separator)
> 将数组的元素组起一个字符串，以separator为分隔符，省略的话则用默认用逗号为分隔符 
```js
var a = [1,2,3,4,5]; 
var b = a.join("|"); //a：[1,2,3,4,5]   b："1|2|3|4|5"
```

### forEach
```js
var arr=[1,2,3,4,5];
arr.forEach(function(item,index){
    console.log(item);
});
// 1
// 2
// 3
// 4
// 5

```
### map
```js
var arr=[1,2,3,4,5];
var newArr= arr.map(function(item,index){
    return item*2;
});
//newArr [2, 4, 6, 8, 10]
```
### filter
```js
var arr=[1,2,3,4,5];
var newArr= arr.filter(function(item,index){
    return item>3;
});
//newArr: [4, 5]
```
### reduce
> 让数组中的前项和后项做某种计算，并累计最终值
```js
var arr=[1,2,3,4,5];
var result= arr.reduce(function(prev,next){
    return prev+next;
});
//result: 15
```
### every
> 检测数组中每项是否符合
```js
var arr=[1,2,3,4,5];
var result=arr.every(function(item,index){
    return item>0;
});
//true
```
### some
> 检测数组中是否有某些项符合条件
```js
var arr=[1,2,3,4,5];
var result= arr.some(function(item,index){
    return item>1;
});
//true
```