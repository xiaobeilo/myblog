---
title: '性能分析:js中如何快速交换两个变量的值?'
date: 2016-11-28 18:24:30
tags: [性能,排序]
---

本文主要探讨js中交换两个变量不同的方法,以及这些方法速度上的差异.
先介绍一下主要的方法,之后再模拟数据去分析比较各方法的性能差异.

### 第一种:声明第三方变量:
```javascript
var a = 3 , b =5;
var temp = a;
a = b;
b = temp;
```
这种方法声明了第三种变量,相当于又在内存中开辟了一个空间.
但是**变量类型不受限制**.
### 第二种: 利用加法运算:
```javascript
var a = 3 , b = 5;
a = a+b;
b = a-b;
a = a-b;
```
上面代码可简化为:
```javascript
b = (a+b)-(a=b);
```
这种只利用了js的简单运算,但是被限制了变量为**number**类型
###第三种:利用位运算XOR(异或操作):
```javascript
var a = 3,b = 5;
a = a^b;
b = b^a;
a = a^b;
```
上面代码可简化为:
```javascript
a = (b^=a^=b)^a;
```
这种方法利用了底层的运算,但是被限制了变量为**number**类型,并且必须是整数
### 第四种:利用数组缓存
```javascript
var a = 3,b = 5;
var arr = [];
arr[0] = b;
arr[1] = a;
a = arr[0];
b = arr[1];
```
上面代码可简化为:
```javascript
a = [b,b=a][0];
```
上面的方法实际本质上也是声明了第三方变量,在内存中新开了一块空间,这个方法同第一种,**不限制数据类型**
第五种: 利用数组的解构赋值
```javascript
var a = 3,b = 5;
[a,b] = [b,a];
```
实际上这种方法是利用了ES6的新特性:变量的结构赋值;
相应的还有对象解构赋值:
```javascript
{a:a,b:b} =  {a:b,b:a};
```
好,那下面开始性能测试:
>我们模拟1W的整数number类型的数组,然后通过冒泡排序的方法多次测量各个方法中的耗时

```javascript
//生成数据:
var count = 10000;
var arr = [];
while(count--){
	arr[arr.length] = ~~(Math.random()*10000);
}
//冒泡排序:
console.time('timer');
function bubble(arr){
	for(var i=0;i<arr.length-1;i++){
		for(var j=0;j<arr.length-i;j++){
			if(arr[j]>arr[j+1]){
				//数据交换
			}
		}
	}
}
bubble(arr)
console.timeEnd('timer');
```
>测试环境统一为node v6.60

测试10次,统计平均结果如下(单位为ms):
{% asset_img sheet.jpg sheet%}

> 结果很明显,es6的数组解构是最慢的.与其他四种方法速度差了将近10倍,其次是数组缓存的数据略慢于其他三种方法.

那么,我们这次把数据加到1000W,这次不进行排序,所有数据统一执行交换操作.只测试三个速度最快的方法.代码如下:

```javascript
var count = 1000000;
var arr1 = [];
var arr2 = [];
while(count--){
	arr1[arr1.length] = ~~(Math.random()*10000);
	arr2[arr2.length] = ~~(Math.random()*10000);
}
console.time('timer');
var len = arr1.length;
for(var i=0;i<len;i++){
	//数据交换
}
console.timeEnd('timer');
```
测试结果如下(单位为ms):

{% asset_img sheet2.jpg sheet%}

> 结果还是能看出来的,声明temp值的方法速度是最快的!而且不限制数据类型!而且不限制数据类型!而且不限制数据类型!重要的事情说三遍!

所以,什么代码美观不美观,都是废话,性能才是最重要的.不过有些同学开始担心内存使用问题了,不过毋须担心,js有完善的内存垃圾回收机制,只要好好写代码,是不容易出现内存泄漏的问题的~

好了综上所诉,性能比较:

### **声明temp值>加法运算≈位运算(异或)>数组缓存>>数组解构**










