## 说明

## 目录

## 一切都是对象

### java中的参数传递是**值传递**还是**引用传递**？

```
都是值传递，对于基本数据类型，传递的是值；对于对象，传递的是对象的一个引用的地址的值
```

### String s = new String("abc")创建了几个对象？

- "abc"本身就是文字池中的一个对象，在运行 new String()时，把常量池即pool中的字符串"abc"复制到堆中，并把这个对象的应用交给s，所以创建了两个String对象，一个在pool中，一个在堆中
  String s = "aa"; 只会创建一个对象，在常量池中
- 只有使用引号包含文本的方式创建的String对象之间使用“+”连接产生的新对象才会被加入字符串池中。对于所有包含new方式新建对象（包括null）的“+”连接表达式，它所产生的新对象都不会被加入字符串池中

### String ss = "ab"+"cd" 会创建几个对象

```
1个，jvm会把"ab"+"cd"优化成"abcd",理由如下：
```

**测试代码**

```java
package com.sugar;

public class Test01 {
    public static void main(String[] args) {
        String ss = "ab"+"cd";
        System.out.println(ss);
    }
}
```

运行后使用 **javap -c** 反编译class文件

```java
Compiled from "Test01.java"
public class com.sugar.Test01 {
  public com.sugar.Test01();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String abcd
       2: astore_1
       3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       6: aload_1
       7: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      10: return
}
```

**结论**

```
从以上jvm字节码可看出，确实只会产生一个对象"abcd"，
ldc    将int, float或String型常量值从常量池中推送至栈顶
```

### java内存模型（JMM）

### 基本类型

- 基本类型数据存储在栈中，访问较快
- java中，每种基本类型所占有的字节数是固定的，方便在不同的机器上进行移植
- java中没有无符号的数值类型
- java提供了BigInteger和BigDecimal进行高精度计算

### 数组

- 引用类型会自动初始化会为null，而基本类型数组会置为0

### 类

- 类的成员属性若没有初始化，会有默认值，引用类型默认值为null

  | 基本类型 | 默认值         |
  | -------- | -------------- |
  | boolean  | false          |
  | char     | '\u0000'(null) |
  | byte     | (byte)0        |
  | short    | (short)0       |
  | int      | 0              |
  | long     | 0L             |
  | float    | 0.0f           |
  | double   | 0.0d           |

- 局部变量没有默认值

## 操作符

- 一元加号可以将较小的数据类型提升为int

### 移位操作符

- <<：无符号左移，低位补0
- \>\>：有符号右移，符号位根据原数符号位确定
- \>\>\>：无符号右移，符号位为0

### 截尾与舍入

- float和double类型转换为int时，会进行截尾，即0.7变成0，-1.5变成-1
- 若想要实现四舍五入，则需要调用Math.round()方法

## 初始化与清理

### 方法重载

- 基本类型作为参数时，会由于类型自动提升，如果有具体匹配到的方法，则直接调用该方法，否则会提升数据类型来调用方法
- char类型参数找不到匹配方法，会被提升为int类型
- 如果传入的值的类型大于参数的类型，就必须进行类型转换（窄化）

### this关键字

- 在调用方法时，编译器会把调用方法的对象引用最为方法的第一个参数进行传递，这对于我们是不可见的,通过this即可访问该对象

  ```java
  Test t = new Test();
  t.fun(a);
  实际上等价于
  Test.fun(t,a)
  ```

- this只能在方法内部使用，并且是非静态方法

- this可以在构造器中调用其它的构造器，只需要填写对应的参数列表即可;注意，只能调用一次，并且只能作为方法的第一句执行

  ```java
  Test(){
      this(a);
  }
  Test(int a){
      
  }
  ```

### finalize

- finalize方法用于执行对象的一些清理操作，会在垃圾回收之前调用，但是垃圾回收不一定会触发，所以该方法不一定会被调用，但是System.gc()可以强制触发

- finalize用于释放通过某种创建对象方式以外的方式为对象分配的存储空间，在java中即本地方法中如使用c语言的malloc分配的空间

- finalize可以用来做对象终结条件验证

  ```java
  package com.sugar;
  
  public class Book {
      boolean checkOut;
  
      Book(boolean checkOut) {
          this.checkOut = checkOut;
      }
  
      void checkIn() {
          checkOut = true;
      }
  
      @Override
      protected void finalize() {
          if (checkOut) {
              System.out.println("error");
          }
      }
  
      public static void main(String[] args) {
          Book book = new Book(true);
          book.checkIn();
  
          //没有调用checkIn方法则finalize里的条件为true
          new Book(true);
          System.gc();
      }
  }
  ```

  