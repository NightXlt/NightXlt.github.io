title: Kotlin修仙之觉醒篇
date: 2015-12-03 00:00:00
tags: [Android,Kotlin]
categories: Kotlin学习笔记
description: "初步觉醒者唤醒体内沉睡的神秘因子，开启特殊能力"
---
## Kotlin基本语法

### 变量与常量

1. 语句末尾没有分号
2. 变量以var开头，常量以val开头
3. 数据类型放在变量的后面，且用：隔开
4. 数据类型首字母大写
5. 变量未初始化，必须指定数据类型，如果初始化了，就可以不指定数据类型。kotlin会自动推导数据类型
```kotlin
	var a: Int = 3
	val b: Int = 20
	var c =3
```
敲黑板:`和 Java类似的是常量的值是不能修改的.`

![更改常量编译出错](/images/常量修改编译错误.png)

###  函数

函数头除了函数名还有fun关键字，返回值在末尾指定与函数定义部分用：隔开。返回值为void可以不写

举个栗子：
```kotlin
	override fun onCreate(savedInstanceState: Bundle?) {
    	super.onCreate(savedInstanceState)
    	setContentView(R.layout.activity_main)
    }

    fun add(a:Int,b:Int):Int{
    	return a+b
    }
```
## Kotlin数据类型

### 数值

kotlin内置数据类型将Java中基本数据类型首字母大写，诸如Double、Int、Byte.
值得注意的是：`kotlin不像Java一样会自动将数据类型向上转型。`
```kotlin
	var m=20
	var n:Byte=10
	m=n//会报Type mismatch error
```
那如果傲娇的想要转换总归还是有办法的：
* toByte()//转换到Byte
* toShort()//转换到Short
* 、、、

Long与Float表示：
Long:123L
Float:12.F

kotlin支持_进行占位,可以不用马上确定值的大小。

	val oneMillion=1_000_000	

![](/images/happy.png)

kotlin并不支持8进制
![](/images/helpness.png)

### 字符

`kotlin中字符不能当作数字进行食用。`

so，this is wrong.
	
	var a:Char='a'
	if(a==95)//编译出错

难道就没有姿势食用吗？
![](/images/noexist.jpg)

前面不是被我提过那啥to什么的，对就是toInt().

### 数组

  * kotlin中，数组使用Array类。如IntArray、ShortArray
  * arrayOf()函数定义存储`任何值`的数组。
  * arrayOfNulls:定义指定长度的空数组

```kotlin
        val arr= arrayOf(1,2,3,'a')
        arr[2]='b'
        var arr1 = arrayOfNulls<Int>(10)
        //使用Array类的构造器进行定义数组，第一个参数：size,第二个参数：初始化每一个数组元素的值
        //每个数组元素的就是当前数组索引i的乘积的String。
        var arr2=Array(10,{i->(i*i).toString()})
        var arr3=intArrayOf(1,1,2,3)
```
### 字符串

kotlin也是使用String作为字符串。除了普通使用的字符串

	var s="hello world\n"

#### 神奇的保留原格式的字符串：
```kotlin
	var s="""

		hello 
			world
				"""
```
输出格式就是:
```kotlin
		
		hello 
			world
																			.```

Amazing!

#### 字符串模版

   字符串添加占位符，内容可在后期指定。模版用$指定，$后跟的是变量如下所示
```kotlin
		val i=10
		val s="i=$i"
		println(s1)
```
输出：i=3

## 包

kotlin中的包与目录没关系，`kt中的包仅仅是为了引用文件资源（函数和类等）。`kt为什么会想单单import个方法，java也有import一个static方法的做法，但是key point Java中方法是定义在类中的，而kt中方法是可以写在类外面的，经测试，kt中类外的方法是static方法，在同包下可以直接访问，不同包下需要import.

   kotlin会默认导入一些常用包，如java.lang.*、kotlin.jvm.*、kotlin.*等。

## 控制流

### if语句本身就是表达式.

kt中的三表达式只能用if，else完成

	val max=if(a>b) a else b

### 多分支选择之when

   kt中when取代了switch的位置，就这么上位了，小不习惯的
```kotlin
	var x=1
	when(x){
		1->{
			//多于一条语句用{}
		}
		2->{
			//满足条件的分支执行后，会自动终止when语句的执行
		}
		3,4->{
			//多分支执行相同代码
		}
		else->{
			//...
		}
	}
```
   当执行同代码的条件比较多时，可食用in关键字。代码：
```kotlin
	when(n){
		in 1..10->{}
		! in 30..60->{}//!in表示不在
	}
```
   when分支条件不仅可以是常量，还可以是任意表达式。如函数。
```kotlin
	 fun main(args: Array<String>) {
        var n=4
        when (n) {
            get(2)->{}
            get(4)->{}
        }
    }
    private fun get(i: Int): Int {
        
        return i*i
    }
```
### for循环

   for-iterator:
```kotlin
    var arr =intArrayOf(2,4,6,5)
    for(item:Int in arr){
    	println(item)
    }
    for(i in arr.indices){//i：是索引
    	println("arr[$i]="+arr[i])
    }
    for((index,value) in array.withIndex()){//键值对
    	println("arr[$index]="+value)
    }
```
