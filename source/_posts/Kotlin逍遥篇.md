title: Kotlin修仙之逍遥篇
date: 2015-12-04 00:00:00
tags: [Android,Kotlin]
categories: Kotlin学习笔记
description: "挣断枷锁，超脱万物，无所依赖，绝对自由"
---
## 类

### 构造方法

1. kt中构造方法分为主构造方法和次构造方法。
2. 主构造方法可看作是必定会执行的构造方法，且是唯一；次构造方法是重写了的构造方法，且不唯一，第二构造方法必须在方法声明后调用主构造方法。
3. kt中，主构造方法的参数允许使用var和val关键字，var表示该参数可修改，val不可修改；次构造方法里不能使用var和val关键字，即次构造方法都是只读的，不能在构造方法内部改变参数的值。

```kotlin
class Person  constructor(firstName: String="Night") {//声明主构造器
        //如果constructor前没有修饰符则constructor可省略不写。
        init {//主构造方法

        }

    constructor(a: Int) : this(""){//必须先调用主构造器
        println("")
    }


    fun main(args: Array<String>) {
        Person()
    }

}
}
```

### 方法中的默认参数

emmmm，我们造Java中是不可以设置默认参数的，毕竟底层的JVM就不支持。那可以转化为java字节码的kt却意外地支持。\(≧▽≦)/，那它是怎么做到的呢？从上述代码的字节码出发

在intellj/AndroidStudio中查看kotlin的字节码选项是在 Menu > Tools > Kotlin > Show Kotlin Bytecode。点击Decompile即可得到字节码，字节码还是太鬼畜啦，我们还是看下java代码压下惊吧。

```java
import kotlin.Metadata;
import kotlin.jvm.internal.DefaultConstructorMarker;
import kotlin.jvm.internal.Intrinsics;
import org.jetbrains.annotations.NotNull;

@Metadata(
   mv = {1, 1, 7},
   bv = {1, 0, 2},
   k = 1,
   d1 = {"\u0000(\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010\b\n\u0002\b\u0002\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\u0002\n\u0000\n\u0002\u0010\u0011\n\u0002\b\u0002\u0018\u00002\u00020\u0001B\u000f\b\u0016\u0012\u0006\u0010\u0002\u001a\u00020\u0003¢\u0006\u0002\u0010\u0004B\u000f\u0012\b\b\u0002\u0010\u0005\u001a\u00020\u0006¢\u0006\u0002\u0010\u0007J\u0019\u0010\b\u001a\u00020\t2\f\u0010\n\u001a\b\u0012\u0004\u0012\u00020\u00060\u000b¢\u0006\u0002\u0010\f¨\u0006\r"},
   d2 = {"Lcom/example/shopsystem/Person;", "", "a", "", "(I)V", "firstName", "", "(Ljava/lang/String;)V", "main", "", "args", "", "([Ljava/lang/String;)V", "production sources for module ShopSystem"}
)
public final class Person {
   public final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      new Person((String)null, 1, (DefaultConstructorMarker)null);
   }

   public Person(@NotNull String firstName) {
      Intrinsics.checkParameterIsNotNull(firstName, "firstName");
      super();
   }

   //自动生成的处理默认参数的构造方法
   //DefaultConstructorMarker是kotlin的一个内部类，负责处理构造方法的默认参数值。
   public Person(String var1, int var2, DefaultConstructorMarker var3) {
      if ((var2 & 1) != 0) {
         var1 = "Night";
      }

      this(var1);
   }
//自动生成的没有参数的构造方法
   public Person() {
      this((String)null, 1, (DefaultConstructorMarker)null);
   }

   public Person(int a) {
      this("");
      String var2 = "";
      System.out.println(var2);
   }
}
```

   从上面代码可以看出，源码并没有直接调用主构造方法，而是调用了另一个自动生成的处理默认参数的构造方法。指定参数的默认值，再调用主构造方法。所以其实kt中的参数默认值是通过在中间构造方法硬编码到字节码中的，然后在中间构造方法又调用包含默认参数值的构造方法。

   kt中方法默认值的设置和C++类似，设置默认值是从后往前的，也就是说如果某个参数带默认值，那么该参数后面的所有参数必须都有默认值。

```kotlin
class Person constructor(a :String ,b:Int=3,c:Int=4){}
```
   
   那如果有这么一种情况，一个方法里指定了很多默认值，你只想修改一个默认值的输入，其余采取原默认值。kt中可以命名参数来传递参数值。

```kotlin
Person("",c=5)
```

   kt中创建类和调用方法语法上是没有区别的，都是类名、方法名（）；因此kt中，`调用方法和创建类的实例是通过语义和上下文来进行区分`

### 方法中空参数

  kt默认在方法中的参数是不允许为空，当传入的参数为空，编译器会直接编译报错；如果想指定方法中的参数可为空，在参数后面加入？即可
  
```kotlin
	fun b(s:String?){
        
	}
```

### 属性语法

   属性语法：类似JavaBean中属性的getter,setter方法；
   看一段代码


```kotlin
	var p = Person()
    p.a=5
```
	
   代码很简单是吧，但p.a=5其实并不是直接赋值，而是调用了属性a的get属性方法。你可能想我并没有写a的get和set方法；其实类的成员变量有默认的get、set方法。

   当然你可能想重写get、set方法，这就需要属性语法啦~
   完整的属性语法如下：

	var/val <propertyName>[: <PropertyType>] [= <property_initializer>]
	[<getter>]
	[<setter>]

   操作如下：

```kotlin
	 var a: Int=0
        get() = field
        set(value) {
            field = value
        }
```     
   
   和Java类似如果变量声明为private，还想在类外访问必须手写get、set方法。不造intellij idea为什么没有自动生成功能⊙︿⊙.

```kolin
	fun geta(): Int {
        return a
    }
```

   方法中的可变参数：

```kotlin
  fun add(vararg persons: Person) {//persons为可变参数，可传入任意多个Person对象
        
    }
```

   当函数体只有一行代码时，可直接在函数体声明后直接加=号，后面直接加代码。返回值类型可省略

```kotlin
	fun get():String = a
	//fun get()= a  两种表达方式等价，
```

### 本地函数
   
   就是函数中套函数，在java中函数中函数是不允许的。

```kolin
	fun add(vararg person: Person) {
        fun a() {}
    }
```

### 修饰符

   和Java有差异的就只有kt并没有default修饰符，kt默认是public。kt多了一个internel修饰符。

   internal：任何在Module内部类都可以访问。

### 类的继承

#### 重写方法
   如果一个类是可继承的则需要显示的使用open关键字表明该类是可继承的。如果父类中的方法需要被重写则也需要加上open关键字。子类对应的方法也需要加上override关键字。如下所示：
```kotlin
	open class Person constructor(a :String ,b:Int=3,c:Int=3){
    //声明主构造器
    private var b: Int = 3
    private var a: Int=0

   

    open fun geta(): Int {
        return a
      }

   
	}
	class student : Person("") {

    override fun geta():Int{
        return super.geta()
      }
	}
```


#### 重写属性

属性也可以重写，是的，你没有听错，只要998，只要998，你就能......咳咳，要注意哈，val属性可被重写为var属性，反之则不行。

```kotlin
open class Person constructor(a :String ,b:Int=3,c:Int=3){
    //声明主构造器
    private var b: Int = 3
    open  val a: Int=0



    open fun geta(): Int {
        return a
    }


}
class student : Person("") {
    override var a:Int=3
    override fun geta():Int{
        return super.geta()
    }
}

```

## 接口

与Java的差异：kt中的接口可以有默认方法实现，有默认方法实现的方法可以不重写该方法

```kotlin
class Person constructor(a :String ,b:Int=3,c:Int=3):Talk{
    override fun a(): String {
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }


}
interface Talk{//接口所有属性、方法都是open的

    fun say(): String{
        return "bb"
    }
    fun a():String
}

```