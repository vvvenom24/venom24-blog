---
title: "***Volatile*** 与 Java 内存模型"
date: 2023-05-24T19:37:01+08:00
lastmod: 2023-06-09T16:13:00+08:00
draft: false
series: [Java]
tags: [Concurrent]
summary: "***Volatile*** 关键字，可以保证多线程环境下的可见性和有序性但是不保证原子性，这是八股文中背烂了的内容，但背后隐藏的真相更值得仔细揣摩"
---
运行下面一段代码：

```java
public class Test {
    private static int x = 0;
    private static int y = 0;
    private static int a = 0;
    private static int b = 0;
    private static CountDownLatch latch = null;
    
    public static void main(String[] args) throws InterruptedException {
        long start = System.currentTimeMillis();

        int i = 0;
        for (;;) {
            i++;

            x = 0;
            y = 0;
            a = 0;
            b = 0;

            latch = new CountDownLatch(1);

            Thread t1 = new Thread(() -> {
                try {
                    latch.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                a = 1;
                x = b;
            });

            Thread t2 = new Thread(() -> {
                try {
                    latch.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                b = 1;
                y = a;
            });

            t1.start();
            t2.start();

            latch.countDown();

            t1.join();
            t2.join();

            if(x == 0 && y == 0) {
                System.err.println("第" + i + "次 (" + x + "," + y + "）");
                break;
            }
        }

        long end = System.currentTimeMillis();

        System.out.println(end - start);
    }
}
```

40 多分钟之后，程序执行结束：

![](https://cdn.jsdelivr.net/gh/vvvenom24/images/20230530183622.png)

正常情况下，程序应该会一直执行下去，永远都不会停下

之所以出现这样的情况，原因有二：重排序与内存可见性

## 重排序

重排序是指编译器、处理器或运行时系统在不改变程序语义的前提下，改变指令的执行顺序的优化行为，目的是为了提供性能和并行度。

简单了解下三种重排序的类型

- 编译器重排序

    编译器在将源代码编译为可执行代码的过程中，可能会对指令进行重排序，以提高程序的执行效率。

- 处理器重排序

    处理器在执行指令时，也可以对指令进行重排序，以提高指令级并行度和执行效率。处理器重排序受到处理器架构和优化策略的影响，可能与编译器重排序不同。

上面的概念了解即可，需要注意的是：

重排序不会影响 **单线程** 下程序的执行结果。单线程下可能会发生重排序，但需要遵循重排序的安全规则[^1]，保证结果的正常。

## 内存可见性

简单来说，就是现在多核 CPU 中每个核心都拥有自己的缓存（多级缓存），每个核心都有自己的 **本地缓存**，它用于存储该核心近期访问的内存数据。缓存的目的是提高内存访问的速度和效率。当核心需要读取或写入内存数据时，它首先查找自己的缓存。如果数据在缓存中存在，那么可以直接从缓存中读取或写入，而不需要访问主内存。

多核 CPU 的各个核心之间会 **共享主内存**。主内存是所有核心共享的内存空间，它存储着程序的数据和指令。当一个核心需要访问主内存中的数据时，它会将数据从主内存读取到自己的缓存中，然后进行操作。当核心对数据进行修改后，它并不会立即将修改后的数据写回主内存，而是在 **适当** 的时机将修改后的数据写回主内存。

由于各个核心拥有自己的缓存，因此在多核 CPU 中，不同核心之间的缓存可能包含不同的数据副本。这就引发了可见性问题，即一个核心对数据的修改可能不会立即被其他核心看到，导致不一致的结果。

可以理解为分布式环境下的各个节点，这样可以提高效率，但是也带来了数据不一致的问题。

为了解决可见性问题，多核 CPU 使用了一些机制来确保数据的一致性和可见性，例如：

- 缓存一致性协议（Cache Coherence Protocol）

    多核 CPU 通过缓存一致性协议来保证各个核心之间的缓存数据的一致性。缓存一致性协议定义了缓存之间的通信和数据同步机制，确保当一个核心对数据进行修改时，其他核心能够看到最新的值。

- 内存屏障（Memory Barrier）

    内存屏障是一种同步指令，用于控制指令的执行顺序和内存访问的可见性。通过插入内存屏障，可以强制核心将修改后的数据写回主内存，并保证其他核心在读取数据时能够看到最新的值。

## 程序中止的原因

1. 根据重排序的规则，`线程 t1` 中的 `a = 1;` 和 `x = b;` 并没有数据的依赖性所以是会发生重排序的，具体哪种重排序我们也无法窥见，但可以明确的是可能会导致先执行 `x = b;` 然后执行 `a = 1;`。`线程 t2` 同理。

2. 如果在没有发生重排序的情况下，`线程 t1` 在核心 1 中执行了 `a = 1;` 与此同时 `线程 t2` 在核心 2 中 执行了 `y = a`，假如核心 1 尚未将缓存刷到主内存中，此时 y 便为 0。同理，x 也可能为 0。

## 老生常谈的懒汉模式中的双重检查

***1.0 版本***

```java
public final class Singleton {

    private static Singleton instance = null;

    private Singleton() {}
    
    public static Singleton getInstance() {
        if (null == instance) {         // 第一次检查
          synchronized (Singleton.class) {
             if (null == instance) {    // 第二次检查
                instance = new Singleton();
             }
          } 
        }
        return instance;
    }
}
```

就直接说问题所在吧

问题的关键在于 `instance = new Singleton();` 这一句，执行这一行代码分为了三个步骤

1. 申请内存空间

2. 初始化各个属性，执行构造函数

3. 将这个对象的引用赋值给变量（`instance`）

比较容易理解的一种情况，这个过程中 [2] 与 [3] 可能会发生重排序，导致可能 A 线程先执行完了 [2]，此时 B 线程执行 ***第一次检查*** 就会判断为 `false` 直接返回 `instance`，但此时的 `instance` 是不完整的。

还有一种比较难理解的可能，A 线程执行完 [3] 但 **未从 synchronized 代码块中出来**，此时对象的赋值仅在 A 线程的本地缓存中，还未刷到主内存中，此时 B 线程也是执行 ***第一次检查*** 就会判断为 `false`，然后返回并不完整的 `instance`。

***Tip***：`synchronized` 关键字的作用：`原子性`、`可见性`、`有序性`。

***2.0 版本***

```java
1     public final class Singleton {
2
3         private static Singleton instance = null;
4 
5         private Singleton() {}
6         
7         public static Singleton getInstance() {
8             if (null == instance) {
9                 Singleton temp;
10                synchronized (Singleton.class) {
12                    temp = instance;
13                    if (null == temp) {
14                        synchronized (Singleton.class) {  // 内嵌一个synchronized代码块
15                            temp = new Singleton();
16                        }
17                    }
18                    instace = temp;
19                }
20            }
21            return instance;
22        }
23    }
```

这个版本增加了一个 synchronized 代码块，想通过 `synchronized` 的 ***可见性*** 保证对象的完整，但是忽略了 ***重排序***

重排序会导致 [18] 行的代码挪到 [16] 行，此时可以发现，又变成了 1.0 版本

所以问题依旧存在

***3.0 版本***

```java
public final class Singleton {

    private static volatile Singleton instance = null;

    private Singleton() {}
    
    public static Singleton getInstance() {
        if (null == instance) {         // 第一次检查
          synchronized (Singleton.class) {
             if (null == instance) {    // 第二次检查
                instance = new Singleton();
             }
          } 
        }
        return instance;
    }
}
```

再熟悉不过了的懒汉模式

## ***synchronized*** 不能替代 ***volatile***

synchronized 很长很 * 很强大，但是它并不能替代 volatile

前面的 ***2.0 版本*** 中，完美的展示了他们二者的区别

synchronized 的原子性先按下不表

synchronized 的有序性，保证了多线程环境下 **线程执行的有序** ，但是没法保证不发生重排序，因为即使是单线程下，重排序也是会发生的，所以才有了 2.0 版本的失败。

synchronized 的可见性，在进入 synchronized 代码块后，线程会到主内存中读取共享变量的最新值（最新的前提是其他线程已将共享变量写入主内存），当结束 synchronized 代码块后，线程会将共享变量的值刷到主内存中。

值得注意的是，synchronized 只能保证自己代码块内的操作的可见性，如果在进入代码块前，主内存中共享变量的值就不是最新值，那么后面的可见性也就无从谈起。简单理解来说就是，***多线程对于共享变量的操作必须确保它们有且是同一把监视器锁***

## 内存模型（[§ 17.4. Memory Model](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)）

（未完待续）



## 特别鸣谢

- [Java 并发基础之内存模型](https://www.javadoop.com/post/java-memory-model)

- [The Myth of volatileJDK1.5之前的volatileJDK1.5之后的volatilevolatile足够了吗？volatile VS 锁结论](https://cloud.tencent.com/developer/article/1121728)



[^1]: 指令重排序必须遵守一些安全规则，以确保在优化指令执行顺序时不改变程序的语义和行为,重排序的安全规则：1. 数据依赖性（Data Dependency）：如果两个操作之间存在数据依赖关系，那么这两个操作的重排序是受限制的。在单线程环境下，编译器和处理器必须确保后续的读操作在先前的写操作之后执行。这样可以保证程序的语义不变。2. as-if-serial 语义（As-If-Serial Semantics）：指令重排序不能改变单线程程序的执行结果。程序的执行结果必须与在指令没有重排序的情况下的执行结果一致。即使编译器和处理器对指令进行了优化和重排序，程序的外部观察结果也必须与按照原始顺序执行的结果相同。3. 顺序一致性（Sequential Consistency）：重排序不能违反程序的顺序一致性。在多线程环境下，编译器和处理器必须确保对共享变量的操作遵循全局的顺序一致性，即所有线程都能观察到相同的操作顺序。