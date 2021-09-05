# 什么是JVM

定义：java Virtual Machine java程序的运行环境(java二进制字节码的运行环境)

好处：

1、一次编写，到处运行

2、自动内存管理、垃圾回收功能

3、数组下标越界检查

4、多态

比较：

![9FDEC0BB-EA79-435B-B991-0450996E1859](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/9FDEC0BB-EA79-435B-B991-0450996E1859.png)

### 学习路线：

![3B628EB8-2B11-486D-805F-660838019122](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/3B628EB8-2B11-486D-805F-660838019122.png)

# 内存结构

## 1.程序计数器

### 1.1作用

Java每一行源代码都有与之相对应的二进制字节码，也就是JVM指令，然而这些JVM指令并不能直接传给CPU，需要通过解释器将这些二进制字节码转换成机器码再传给CPU，而程序计数器呢，是负责记录下一条JVM指令的执行地址，这样就能保证每一个JVM指令都被转换成机器码。

### 1.2特点

1、是线程私有的，每个线程都有属于自己的程序计数器

2、不会存在内存溢出

## 2.虚拟机栈

1、栈：线程运行时所需要的内存空间

2、栈由多个栈帧组成，栈帧是每个方法调用时所占用的内存，比如存一些参数、局部变量，返回地址

3、每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法



问题辨析：

1、垃圾回收是否涉及栈内存？

不涉及，因为栈内存是一次次的方法调用所产生的的栈帧内存，而栈帧每次方法调用完后都会弹出栈。垃圾回收只是回收堆内存中的无用对象。

2、栈内存分配越大越好吗？

不是，栈内存越大，线程数越少

3、方法内的局部变量是否线程安全？

1、共享的变量需要考虑线程安全，私有的不需要考虑(如果方法内局部变量没有逃离方法的作用范围，他就是安全的)

2、如果是局部变量引用了对象，并逃离了方法的范围，需要考虑线程安全

### 2.1栈内存溢出

1.栈帧过多导致栈内存溢出

```java
package com.jvm.stack;
//栈内存溢出演示 java.lang.stackOverFlowError
public class StackFlow {
    private static int count=0;
    public static void main(String[] args) {
        try {
            method1();
        }catch (Throwable e){
            e.printStackTrace();
            System.out.println(count);
        }
    }
    private static void method1(){
        count++;
       method1();
    }
}

```



2.栈帧过大导致栈内存溢出



### 2.2线程运行诊断

案例1：cpu占用过多

案例2：程序运行很长时间没有结果

## 3.本地方法栈

给本地方法提供运行的空间。

本地方法有很多，比如Object类中的notify()，wait()，hashcode()

## 4.堆

### 4.1定义

Heap堆

通过new关键字，创建对象都会使用堆内存

特点：

它是线程共享的，堆中对象都需要考虑线程安全问题

有垃圾回收机制

### 4.2堆内存溢出

为什么堆有垃圾回收机制还可能出现内存溢出问题？

垃圾回收是针对没人使用的对象，但如果不断的产生对象，而且产生的新对象仍然有人在使用，这样的对象就不能被认为是垃圾，这样的对象达到一定的数量时，就会导致堆内存耗尽也就是堆内存溢出。

```java
package com.jvm.Heap;

//-Xmx控制堆空间大小
//java.lang.OutOfMemoryError: Java heap space 堆内存错误
import java.util.ArrayList;
import java.util.List;

public class HeapOverflow {
    public static void main(String[] args) {
        int i= 0;
        try{
            List<String> list = new ArrayList<>();
            String a = "hello";
            while (true){
                list.add(a);
                a = a + a;
                i++;
            }
        }catch (Throwable e){
            e.printStackTrace();
            System.out.println(i);
        }


    }
}

```

### 4.3堆内存诊断

1、jps工具

查看当先系统中有哪些java进程

2、jmap工具

查看堆内存占用情况

3、jconsole工具

图形界面的，多功能的监测工具，可以连续监测

## 5.方法区

### 5.1定义

*java虚拟机规范中明确说明，尽管所有的方法区在逻辑上属于堆的一部分，但是一些简单的实现可能不会选择去进行垃圾收集或者进行压缩（避免碎片）,但是对于hotspot虚拟机，方法区还有个别名叫做Non-Heap(非堆)，目的就是要和堆分开，所以方法区可以看作是一块独立于Java堆的内存空间*

### 5.2组成

![9753EAA0-6E8B-48D8-864D-2CFE58D3BC23](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/9753EAA0-6E8B-48D8-864D-2CFE58D3BC23.png)

### 5.3方法区内存溢出

1.8以前会导致永久代内存溢出

```java
//演示永久代内存溢出java.lang.OutOfMemoryError: PerGen space
//-XX:MaxPermSize=8m
```



1.8以后会导致元空间内存溢出

```java
//演示元空间的内存溢出 java.lang.OutOfMemoryError: Metaspace
//-XX:MaxMetaspaceSize=8m
```



```java
package com.jvm;

import jdk.internal.org.objectweb.asm.ClassWriter;
import jdk.internal.org.objectweb.asm.Opcodes;

//演示元空间的内存溢出
//-XX:MaxMetaspaceSize=8m
public class methodClock extends ClassLoader {//可以用来加载类的二进制字节码
    public static void main(String[] args) {
        int j=0;
        try{
            methodClock methodClock = new methodClock();
            for (int i = 0; i < 10000; i++,j++) {
                //ClassWriter作用是生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                //版本号，public，类名，包名，父类，接口
                cw.visit(Opcodes.V1_8,Opcodes.ACC_PUBLIC,"Class"+i,null,"java/lang/Object",null);
                //返回byte[]
                byte[] code = cw.toByteArray();
                //执行类的加载
                methodClock.defineClass("Class"+i,code,0,code.length);
            }
        }finally {
            System.out.println(j);
        }

    }
}

```



### 5.4运行时常量池

常量池：就是一张表，虚拟机指令根据这张常量表找到要执行的类名，方法名，参数类型，字面量等信息。

运行时常量池：常量池是*.class文件中的，当该类被加载，它的常量池信息就会放入运行时常量池，并把里面的符号地址变为真实地址

### 5.5常量池与串池的关系

```java
package com.jvm;
//StringTable[]
public class Demo {
    //常量池中的信息，都会被加载到运行时常量池中，这是a,b,ab都是常量池中的符号，还没有变为java字符串对象
    //ldc #2会把 a 符号变为"a"字符串对象 然后把"a"作为key到StringTable里去找看有没有与之相同的key，没有的话就将"a"放入StringTableldc #2会把 a 符号变为"a"字符串对象
    //ldc #3会把 b 符号变为"b"字符串对象 重复上述操作
    //ldc #4会把 ab 符号变为"ab"字符串对象 重复上述操作
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1+s2;//等于 new StringBuilder().append("a").append("b").toString() 相当于 new String="ab"
        String s5 = "a"+"b";//javac 在编译期间的优化，结果已经在编译期确定为ab
        System.out.println(s3==s4);//false
        System.out.println(s3==s5);//true
    }
}

```

         0: ldc           #2                  // String a
         2: astore_1
         3: ldc           #3                  // String b
         5: astore_2
         6: ldc           #4                  // String ab
         8: astore_3
         9: return
### 5.6、StringTable特性

常量池中字符串仅是符号，第一次用到时才变为对象

利用串池的机制，来避免重复创建字符串对象

字符串变量拼接的原理是StringBuilder(1.8)

字符串常量拼接的原理是编译期优化

可以使用intern方法，主动将串池中还没有的字符串放入串池

Intern:

```java
public class Demo {
    //StringTable["a","b","ab"]
    public static void main(String[] args) {
        String s = new String("a") + new String("b"); //new String("ab")
        //堆：new String("a") new String("b") new String("ab")
        String s2 = s.intern();//将字符串对象尝试放入串池,如果有则并不会放入，如果没有则放入串池，会把串池中的对象返回
        System.out.println(s2=="ab");//true
        System.out.println(s =="ab");//true
    }
}

public class Demo {
    //StringTable["ab","a","b"]
    public static void main(String[] args) {
        String x ="ab";
        String s = new String("a") + new String("b"); //new String("ab")
        //堆：new String("a") new String("b") new String("ab")
        String s2 = s.intern();//将字符串对象尝试放入串池,如果有则并不会放入，如果没有则放入串池，会把串池中的对象返回
        System.out.println(s2==x);//true
        System.out.println(s ==x);//false
    }
}

```

### 5.7、StringTable位置

jdk1.6 StringTable在永久代

jdk1.8 StringTable在堆空间

### 5.8、StringTable 垃圾回收

StringTable也是受到垃圾回收机制管理的，当内存空间不足时，StringTable哪些没有被引用的字符串常量仍然会被垃圾回收

### 5.9、StringTable性能调优

调整-XX: StringTableSize=XXXX;设置桶的个数，桶的个数越多，性能越好，存的越快

考虑将字符串对象是否入池

## 6.直接内存

### 6.1、定义

常见于NIO操作时，用于数据缓冲区

分配回收成本较高，但读写性能高

不受JVM内存回收管理

### 6.2、分配和回收原理

使用了 Unsafe 对象完成直接内存的分配回收，并且回收需要主动调用 freeMemory 方法

ByteBuffffer 的实现类内部，使用了 Cleaner （虚引用）来监测 ByteBuffffer 对象，一旦

ByteBuffffer 对象被垃圾回收，那么就会由 ReferenceHandler 线程通过 Cleaner 的 clean 方法调

用 freeMemory 来释放直接内存

# 垃圾回收

## 1、如何判断对象可以回收

### 1.1、引用计数法

只要一个对象被其他变量所引用，那就让这个对象的计数加一，如果被引用了两次，就加二，如果某一个变量不在引用它了，那就让计数减一，当这个对象的引用计数变为零是，就代表没人再引用它了，就可以作为一个垃圾回收

缺点：循环引用导致引用计数器不能为零，不能被回收，例子：A引用B，B+1，B引用A，A+1，但再次过后A和B都没有被再次引用，而引用计数器却不能归零，导致不能回收，造成内存泄漏

![5C6E5D90-73A7-45CB-9AD4-52A0192482D8](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/5C6E5D90-73A7-45CB-9AD4-52A0192482D8.png)

### 1.2、可达性分析算法

首先确定根对象(哪些肯定不能被当成垃圾的对象)，在垃圾回收之前，我们会对堆中的所有对象进行一遍扫描，然后看看每一个对象是不是被根对象直接或者间接的使用，如果是，那么这个对象就不能被回收，反之，就可以作为垃圾，将来被回收

那么，哪些对象可以作为GC Root？

**a.** java虚拟机栈中的引用的对象。 

  **b**.方法区中的类静态属性引用的对象。 （一般指被static修饰的对象，加载类的时候就加载到内存中。）

  **c**.方法区中的常量引用的对象。 

  **d**.本地方法栈中的JNI（native方法）引用的对象

### 1.3、四种引用

![E13A5310-9544-4CEE-9198-AEBF93E1BF83](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/E13A5310-9544-4CEE-9198-AEBF93E1BF83.png)

1.强引用

被GC Root强引用引用的对象不会被垃圾回收，实线代表强引用，当所有实线都断开的时候，A1对象可以被垃圾回收

2.软引用

背 GC Root软引用的对象满足下列条件时会被垃圾回收：在执行垃圾回收后发现内存还是不够时，软引用的对象会被垃圾回收

3.弱引用

跟软引用很像，区别是不管垃圾回收后内存够不够，都会被垃圾回收

4.虚引用

虚引用引用的对象被垃圾回收时，虚引用对象自己就会放入引用队列，从而间接的用一个线程调用虚引用方法，调用unsafe.freememory，去释放直接内存

5.终结器引用

当终结器引用的对象重写了finallize()方法时，并且这个对象没有没强引用的时候，这个对象就可以被当做垃圾回收，当这个对象被垃圾回收时，终结器引用也加入引用队列，注意：此时这个对象还没有被垃圾回收，再又一个优先级很低的线程(finallizeHandler)去查看引用队列里是不是有终结器引用，如果有，就根据终结器引用找到要作为垃圾的对象，并且调用finalize方法，等调用完了，下一次垃圾回收时就可以把这个对象占用的内存回收掉

## 2、垃圾回收算法

### 2.1、标记清除

第一阶段，沿着GC Root的引用链找，扫描整个堆对象的过程中，如果发现对象确实被引用了，那么就保留，否则就标记成垃圾。

第二阶段，把标记成垃圾的对象清除

优点：速度比较快

缺点：容易产生内存碎片，容易产生内存溢出

![0FF07C27-77FF-4A1F-A94C-32BA2A14D035](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/0FF07C27-77FF-4A1F-A94C-32BA2A14D035.png)

### 2.2、标记整理

第一阶段和标记清除一样

第二阶段整理：避免标记清除产生的内存碎片的问题 ，在清理垃圾的过程中，把可用的对象向前移动，让内存更为紧凑

优点：没有内存碎片

缺点：效率较低

![CC46DA5E-A94A-42A5-9BFE-403796BC3913](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/CC46DA5E-A94A-42A5-9BFE-403796BC3913.png)

### 2.3、复制

第一阶段也是标记垃圾。

第二阶段把from区域存活的对象复制到To区中，复制过程中有内存碎片的整理，所以也不会产生内存碎片 ，然后一次清空from，并且交换from和To的位置，保证To总是空闲的区域

优点：不会产生内存碎片

缺点：会占用双倍的内存空间

![D206D9C6-0A5F-4737-8CF4-862470531FDA](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/D206D9C6-0A5F-4737-8CF4-862470531FDA.png)

## 3、分代垃圾回收

![2EFE0F1A-0719-40FB-AE82-5F3E5E8A5D3C](/Users/apple/Library/Containers/com.tencent.qq/Data/Library/Application Support/QQ/Users/781936292/QQ/Temp.db/2EFE0F1A-0719-40FB-AE82-5F3E5E8A5D3C.png)

对象首先分配在伊甸园区，新生代空间不足时，触发minor GC，伊甸园和from存活的对象使用copy复制到to中，存货的对象年龄加一，并且交换from To

minor GC会引发stop the world，暂停其他用户的线程，等垃圾回收结束，用户线程才恢复运行

当对象寿命超过阈值时，会晋升到老年代，最大寿命是15次(4bit)

当老年代空间不足时，先尝试进行一次minor GC，如果空间还不足，这时就触发一次Full GC，Full GC也会引发一次stop the world且时间更长，如果此时空间还不足，则触发OutOfMeomaryError：Heap Space

### 3.1、相关VM参数

| 堆初始大小         | -Xms                                                         |
| ------------------ | ------------------------------------------------------------ |
| 堆最大大小         | -Xmx 或 -XX:MaxHeapSize=size                                 |
| 新生代大小         | -Xmn 或 (-XX:NewSize=size + -XX:MaxNewSize=size )            |
| 幸存区比例（动态） | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
| 幸存区比例         | -XX:SurvivorRatio=ratio                                      |
| 晋升阈值           | -XX:MaxTenuringThreshold=threshold                           |
| 晋升详情           | -XX:+PrintTenuringDistribution                               |
| GC详情             | -XX:+PrintGCDetails -verbose:gc                              |
| FullGC 前 MinorGC  | -XX:+ScavengeBeforeFullGC                                    |

4、垃圾回收器

5、垃圾回收调优



# 类加载与字节码技术

