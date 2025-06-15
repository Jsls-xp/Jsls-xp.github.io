---
title: "Kotlin companion"
date: 2025-02-21
categories: [Android, Kotlin]
tag: [object, companion]
toc: false
---

## 一、object关键字

### 1.1 对象声明
对象声明是定义单例的一种方式，对象声明与类一样也可以包含属性、方法、初始化语句块等的声明，唯一不允许的是有构造方法(包括主构造方法和从构造方法)。与普通类的实例不同，对象声明在定义的时候就立即创建了，不需要在代码的其他地方调用构造方法，具体语法为：
```kotlin
object 类名 [： 继承父类、实现接口] {
    成员属性
    成员函数
}
```

```kotlin
object PayRoll: Comparator<String>{
    val allNames = arrayListOf<String>()
    
    fun calculateSalary(){
        for (name in allNames){
            println(name)
        }
    }

    override fun compare(p0: String, p1: String): Int {
        return p0.compareTo(p1)
    }
}
```
编译成Java代码如下，Kotlin中的对象声明被编译成了通过静态字段来持有它的单一实例的类，这个字段名称始终都是INSTANCE。
```java
public final class PayRoll implements Comparator {
   @NotNull
   private static final ArrayList allNames;
   @NotNull
   public static final PayRoll INSTANCE;

   @NotNull
   public final ArrayList getAllNames() {
      return allNames;
   }

   public final void calculateSalary() {
      Iterator var2 = allNames.iterator();

      while(var2.hasNext()) {
         String name = (String)var2.next();
         System.out.println(name);
      }

   }

   public int compare(@NotNull String p0, @NotNull String p1) {
      Intrinsics.checkNotNullParameter(p0, "p0");
      Intrinsics.checkNotNullParameter(p1, "p1");
      return p0.compareTo(p1);
   }

   // $FF: synthetic method
   // $FF: bridge method
   public int compare(Object var1, Object var2) {
      return this.compare((String)var1, (String)var2);
   }

   private PayRoll() {
   }

   static {
      PayRoll var0 = new PayRoll();
      INSTANCE = var0;
      allNames = new ArrayList();
   }
}
```

```kotlin

```
 ### 1.2 对象表达式
 对象表达式类似于Java的匿名内部类，与Java匿名内部类相比有以下特性：
 * 支持实现多个接口
 * 能够访问非final修饰的变量
```kotlin
interface MouseAdapter{
    fun mouseClick()
    fun mouseEntered()
}

fun registerListener(adapter: MouseAdapter){
    
}

fun main() {
    registerListener(object : MouseAdapter{
        override fun mouseClick() {
            
        }

        override fun mouseEntered() {
            
        }
    })

    val listener = object : MouseAdapter{
        override fun mouseClick() {

        }

        override fun mouseEntered() {

        }
    }
    //或者
    registerListener(listener)
}
```


 ## 二、伴生对象 companion object
Kotlin没有static关键字，就不能拥有静态属性和静态方法，使用companion关键字后，就获得了直接通过容器类名称来访问对象的方法和属性，不再需要显示地指明对象的名称，最终看起来像Java中的静态方法调用。
### 2.1 作为普通的静态方法调用
```kotlin

```
可以通过伴生对象的类名来引用伴生对象，但是也可以设置伴生对象的名字。如果省略了伴生对象的名字，名字将会默认分配为Companion。
### 2.2 实现接口的调用
```kotlin
interface Person{
    fun work()
}
interface Other{
    fun doOtherThing()
}

class Student {
    companion object : Person, Other{
        override fun work() {
            
        }

        override fun doOtherThing() {
            
        }
    }
}
```
转换成Java代码：
```java
public interface Person {
   void work();
}

public interface Other {
   void doOtherThing();
}

public final class Student {
   @NotNull
   public static final Companion Companion = new Companion((DefaultConstructorMarker)null);

   public static final class Companion implements Person, Other {
      private Companion() {
      }

      public void work() {
      }

      public void doOtherThing() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```
### 2.3 伴生对象扩展
```kotlin
class Person{
    companion object{
        
    }
}

fun Person.Companion.from(){
    
}

val p = Person.from()
```

