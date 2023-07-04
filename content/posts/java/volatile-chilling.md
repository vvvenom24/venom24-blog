---
title: "***Volatile*** 与 Java 内存模型"
date: 2023-05-24T19:37:01+08:00
lastmod: 2023-07-04T17:26:00+08:00
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

## 内存模型（[Java 8 JLS：§ 17.4. Memory Model](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)）

重排序通常会导致一些令人难以置信的结果的出现，而 Java 内存模型（以下简称：JMM）就是检查执行跟踪中的每次读取，并根据特定规则检查该读取观察到的写入是否有效。

JMM 描述了程序可能的行为，实现可以自由地生成它喜欢的任何代码，只要程序的所有结果执行都会生成可以由内存模型预测的结果。

这为实现者提供了很大的自由来执行无数的代码转换，包括重新排序操作和删除不必要的同步。

> 简单来说，就是 JMM 定义了一套多线程访问或修改共享变量的规则，使得编程者可以准确的预测多线程的执行结果

以下为 Java 8 JLS 重排序示例（[Example 17.4-1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4-1)），结论与上面的单例模式例子相同

---

Java 允许 compilers 和 CPU 对执行指令进行重排序，导致我们会经常看到似是而非的现象。

如表 17.4-A 中所示，有共享变量 A 和 B，局部变量 r1 和 r2 。初始时，令 A = B = 0。

**表 17.4-A. 语句重新排序导致的令人惊讶的结果 - 原始代码**

| Thread1 | Thread 2 |
| --- | --- |
| 1: r2 = A | 3: r1 = B |
| 2: B = 1 | 4: A = 2 |

看起来 `r2 = 2 && r1 = 1` 是不可能的，直观上，指令 1 或指令 3 应该首先执行。如果指令 1 先出现，则不应看到指令 4 处的写入。如果指令 3 先出现，则不应看到指令 2 处的写入。

如果某个执行结果表现出 `r2 = 2 && r1 = 1` 这种行为，那么我们就会知道指令 4 在指令 1 之前，指令 1 在指令 2 之前，指令 2 在指令 3 之前，指令 3 在指令 4 之前。从表面上看，这是荒谬的。

但是，当这不会影响该线程的独立执行时，编译器可以对任一线程中的指令进行重新排序。如果指令 1 与指令 2 重新排序，如表 17.4-B 中的跟踪所示，那么很容易看出结果 `r2 = 2 && r1 = 1` 可能如何出现。

**表 17.4-B. 语句重新排序引起的令人惊讶的结果 - 有效的编译器转换**

| Thread1 | Thread 2 |
| --- | --- |
| B = 1 | r1 = B |
| r2 = A | A = 2 |

对于某些程序员来说，这种行为可能看起来「不正常」。但需要注意的是，这段代码同步不正确：

- 其中有一个线程执行了写操作

- 另一个线程对同一个属性执行了读操作 

- 同时，读操作和写操作没有使用同步来确定它们之间的执行顺序

这个是 ***数据竞争（data race）*** 的一个例子。当代码包含数据竞争时，经常会发生违反我们直觉的结果。

有几个机制会导致表 17.4-B 中的指令重排序。java 的 JIT 编译器可能会重排序代码，或者 CPU 也会做重排序操作。此外，java 虚拟机实现中的内存层次结构也会使代码像重排序一样。在本章中，我们将所有这些会导致代码重排序的东西统称为 compiler。

> 所以，这里的 `compiler` 不能狭义的理解为 `编译器`，这里代表了所有可能产生重排序的 ***机制*** 。比如 JVM 优化，CPU 优化以及内存优化等。

另一个可能产生奇怪的结果的示例如表 17.4-C，初始时 `p = q && p.x = 0` 这段代码也是没有正确使用同步的，在这些写入共享内存的写操作中，没有进行强制的先后排序。

**表 17.4-C. 正向替换带来的令人惊讶的结果**

| Thread1 | Thread 2 |
| --- | --- |
| r1 = p | r6 = p |
| r2 = r1.x | r6.x = 3 |
| r3 = q |
| r4 = r3.x |
| r5 = r1.x |

一个简单的编译器优化操作是会复用 r2 的结果给 r5，因为它们都是读取 r1.x，而且在单线程语义中，r2 到 r5 之间没有其他的相关的写入操作，这种情况如表 17.4-D 所示。

**表 17.4-D. 正向替换带来的令人惊讶的结果**

| Thread1 | Thread 2 |
| --- | --- |
| r1 = p | r6 = p |
| r2 = r1.x | r6.x = 3 |
| r3 = q |
| r4 = r3.x |
| <font color=red>r5 = r2</font> |

现在，我们来考虑一种情况，在 Thread1 第一次读取 r1.x 和 r3.x 之间，Thread2 执行 `r6=p; r6.x=3;` 编译器进行了 r5 复用 r2 结果的优化操作，那么 `r2 = r5 = 0 && r4 == 3`，从程序员的角度来看，`p.x` 的值由 0 变为 3，然后又变为 0。

整个过程可能是这样的：

| Thread1 | Thread 2 | p.x |
| --- | --- | --- |
| r1 = p | | 0 |
| r2 = r1.x | | 0 |
| | r6 = p | 0|
| | r6.x = 3 | 3 |
| r3 = q | | 3 |
| r4 = r3.x | | 3 |
| <font color=red>r5 = r2</font> | | 0 |

---

JMM 定义了在程序的每一步，哪些值是内存可见的。对于隔离的每个线程来说，其操作是由我们线程中的语义来决定的，但是线程中读取到的值是由 JMM 来控制的。当我们提到这点时，我们说程序遵守线程内语义，线程内语义说的是单线程内的语义，它允许我们基于线程内读操作看到的值完全预测线程的行为。如果我们要确定线程 t 中的操作是否是合法的，我们只要评估当线程 t 在单线程环境中运行时是否是合法的就可以，该规范的其余部分也在定义这个问题。

***这里描述的内存模型并不是基于 Java 编程语言的面向对象。为了简洁起见，我们经常展示没有类或方法定义的代码片段。大多数示例包含两个或多个线程，其中包含局部变量，共享全局变量或对象的实例字段的语句。我们通常使用诸如 r1 或 r2 之类的变量名来表示方法或线程本地的变量。 其他线程无法访问此类变量。***

### 共享变量（Shared Variables）

所有线程都可以访问到的内存称为***共享内存***或***堆内存***。

所有实例字段、 static 字段和数组元素都存储在堆内存中。

局部变量、方法参数、异常对象，它们不会在线程间共享，也不会受到内存模型定义的任何影响。

两个线程对同一个变量同时进行 `读-写` 操作或 `写-写` 操作，我们称之为「冲突」。

### 操作（Actions）

***线程间操作***是指由一个线程执行的动作，可以被另一个线程检测到或直接影响到。以下是几种可能发生的***线程间操作***：

- 读（普通变量，非 volatile）。读一个变量。

- 写（普通变量，非 volatile）。写一个变量。

- 同步操作，如下：

  - volatile 读。读一个 volatile 变量
  
  - volatile 写。写入一个 volatile 变量
  
  - 加锁。对一个对象的监视器加锁
  
  - 解锁。解除对某个对象的监视器锁
  
  - 线程的第一个和最后一个操作
  
  - 开启线程操作，或检测一个线程是否已经结束

- ***外部操作***（简单说，外部操作的外部指的是在 JVM 之外，如 native 操作。）一个外部操作指的是可能被观察到的在外部执行的操作，同时它的执行结果受外部环境控制。

e.g.

```java
class Test {
    int foo = 0;

    public static void main(String[] args) {
        dost(); // 外部操作
        foo++;
    }

    // 假设 `dost` 是一个 native 方法
    native void dost();
}
```

这里 `dost` 是一个 native 方法，Java 只能获取 `dost` 的返回值，无法知道其具体操作，所以就无法对其重排序，这便是一个外部操作。

- ***线程分歧操作***（[§17.4.9](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.9)）。此操作只由处于无限循环的线程执行，在该循环中不执行任何内存操作、同步操作、或外部操作。如果一个线程执行了分歧操作，那么其后将跟着无数的线程分歧操作。

  *引入分歧操作的引入是为了用来说明，线程可能会导致其他所有线程停顿而不能继续执行。*

此规范仅关心线程间操作，我们不关心线程内部的操作（比如将两个局部变量的值相加存到第三个局部变量中）。如前文所说，所有的线程都需要遵守线程内语义。对于线程间操作，我们经常会简单地称为操作。

定义一个 tuple < *t*, *k*, *v*, *u* > 包括：

- *t* - 执行操作的线程

- *k* - 操作的类型

- *v* - 操作涉及的变量或监视器

  对于加锁操作，v 是被锁住的监视器；对于解锁操作，v 是被解锁的监视器

  如果是一个读操作（volatile 读或非 volatile 读），v 是读操作对应的变量

  如果是一个写操作( volatile 写或非 volatile 写)，v 是写操作对应的变量

- *u* - 唯一的标识符标识此操作

外部动作元组还包含一个附加组件，其中包含由执行操作的线程感知的外部操作的结果。 这可能是关于操作的成败的信息，以及操作中所读的任何值。

外部操作的参数（如哪些字节写入哪个 socket）不是外部操作元祖的一部分。这些参数是通过线程中的其他操作进行设置的，并可以通过检查线程内语义进行确定。它们在内存模型中没有被明确讨论。

在非终结执行中，不是所有的外部操作都是可观察的。[§17.4.9](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.9) 小节讨论非终结执行和可观察操作。

e.g.

```java
class Test {
    AtomicInteger foo = new AtomicInteger(0);

    new Thread(() -> {
        while (true) {} // 线程分歧操作
        foo.set(100);
    }).start();

    new Thread(() -> {
        assert foo.get() == 0;  // 这里永远不会失败
    }).start();
}
```

### 程序和程序顺序（Programs and Program Order）

在每个线程 *t* 执行的所有线程间动作中，*t* 的程序顺序是反映 根据 *t* 的线程内语义执行这些动作的顺序的总顺序。

如果所有操作的执行顺序和代码中的顺序一致，那么一组操作就是连续一致的，并且，对变量 *v* 的每个读操作 *r* 会看到写操作 *w* 写入的值，也就是：

- 写操作 *w* 先于读操作 *r* 完成，并且

- 没有其他的写操作 *w'* 使得 *w'* 在 *w* 之后 *r* 之前发生

连续一致性对于可见性和程序执行顺序是一个非常强的保证。在这种场景下，所有的单个操作（比如读和写）构成一个统一的执行顺序，这个执行顺序和代码出现的顺序是一致的，同时每个单个操作都是原子的，且对所有线程来说立即可见。

如果程序没有任何的数据竞争，那么程序的所有执行操作将表现为连续一致。

连续一致性 和/或 数据竞争的自由仍然允许错误从一组操作中产生

***如果我们用连续一致性作为我们的内存模型，那我们讨论的许多关于编译器优化和处理器优化就是非法的。比如在 17.4-C 中，一旦执行 p.x=3，那么后续对于该位置的读操作应该是立即可以读到最新值的。***

***连续一致性的核心在于每一步的操作都是原子的，同时对于所有线程都是可见的，而且不存在重排序。所以，Java 语言定义的内存模型肯定不会采用这种策略，因为它直接限制了编译器和 JVM 的各种优化措施。***

### 同步顺序（Synchronization Order）

每个执行都有一个同步顺序。同步顺序是由执行过程中的每个同步操作组成的顺序。对于每个线程 *t*，同步操作组成的同步顺序是和线程 *t* 中的代码顺序一致的。

同步操作包括了如下同步关系：

- 对于监视器 *m* 的解锁与所有后续操作对于 *m* 的加锁同步

- 对 *volatile* 变量 *v* 的写入，与所有其他线程后续对 *v* 的读同步

- 启动线程的操作与线程中的第一个操作同步

- 对于每个属性写入默认值（0，false，null）与每个线程对其进行的操作同步

  尽管在创建对象完成之前对对象属性写入默认值有点奇怪，但从概念上来说，每个对象都是在程序启动时用默认值初始化来创建的。

- 线程 *T1* 的最后操作与线程 *T2* 发现线程 *T1* 已经结束同步

  线程 *T2* 可以通过 `T1.isAlive()` 或 `T1.join()` 方法来判断 *T1* 是否已经终结

- 如果线程 *T1* 中断了 *T2*，那么线程 *T1* 的中断操作与其他所有线程发现 *T2* 被中断了同步（通过抛出 `InterruptedException` 异常，或者调用 `Thread.interrupted` 或 `Thread.isInterrupted`）

以上同步顺序可以理解为对于某资源的释放先于其他操作对同一资源的获取。

### Happens-before顺序（Happens-before Order）

两个操作可以用 happens-before 来确定它们的执行顺序，如果一个操作 happens-before 于另一个操作，那么我们说第一个操作对于第二个操作是可见的。

happens-before 强调的是可见性问题

如果我们分别有操作 *x* 和操作 *y*，我们写成 `hb(x, y)` 来表示 *x happens-before y*。

这里不代表不可以重排序，只要没有数据依赖关系，重排序就是可能的。

- 如果操作 *x* 和操作 *y* 是同一个线程的两个操作，并且在代码上操作 *x* 先于操作 *y* 出现，那么有 `hb(x, y)`

- 对象构造方法的最后一行指令 happens-before 于 `finalize()` 方法的第一行指令

- 如果操作 *x* 与随后的操作 *y* 构成同步，那么 `hb(x, y)`

- `hb(x, y)` 和 `hb(y, z)`，那么可以推断出 `hb(x, z)`

对象的 wait 方法关联了加锁和解锁的操作，它们的 happens-before 关系即是 *加锁* happens-before *解锁*

我们应该注意到，两个操作之间的 happens-before 的关系并不一定表示它们在 JVM 的具体实现上必须是这个顺序，如果重排序后的操作结果和合法的执行结果是一致的，那么这种实现就不是非法的

比如说，在线程中对对象的每个属性写入初始默认值并不需要先于线程的开始，只要这个事实没有被读到就可以了

更具体地说，如果两个操作是 happens-before 的关系，但是在代码中它们并没有这种顺序，那么就没有必要表现出 happens-before 关系。如线程 1 对变量进行写入，线程 2 随后对变量进行读操作，那么这两个操作是没有 happens-before 关系的

happens-before 关系用于定义当发生数据竞争的时候

将上面所有的规则简化成以下列表：

- 对一个监视器的解锁操作 happens-before 于后续的对这个监视器的加锁操作

- 对 volatile 属性的写操作先于后续对这个属性的读操作

- 线程的 start() 先于任何在线程中定义的语句

- 如果 A 线程中调用了 B.join()，那么 B 线程中的操作先于 A 线程 join() 返回之后的任何语句

- 对象的默认初始值 happens-before 于程序中对它的其他操作

当程序出现两个没有 happens-before 关系的操作对同一数据进行访问时，我们称之为程序中有数据竞争

除了线程间操作，数据竞争不直接影响其他操作的语义，如读取数组的长度、检查转换的执行、虚拟方法的调用

因此，数据竞争不会导致错误的行为，例如为数组返回错误的长度

当且仅当所有连续一致的操作都没有数据争用时，程序就是正确同步的

如果一个程序是正确同步的，那么程序中的所有操作就会表现出连续一致性

***这是一个对于程序员来说强有力的保证，程序员不需要知道重排序的原因，就可以确定他们的代码是否包含数据争用。因此，他们不需要知道重排序的原因，来确定他们的代码是否是正确同步的。一旦确定了代码是正确同步的，程序员也就不需要担心重排序对于代码的影响。***

程序必须正确同步，以避免当出现重排序时，会出现一系列的奇怪的行为。正确同步的使用，不能保证程序的全部行为都是正确的。但是，它的使用可以让程序员以很简单的方式就能知道可能发生的行为。正确同步的程序表现出来的行为更不会依赖于可能的重排序。没有使用正确同步，非常奇怪、令人疑惑、违反直觉的任何行为都是可能的

我们说，对变量 *v* 的读操作 *r* 能看到对 *v* 的写操作 *w*，如果：

- 读操作 *r* 不是先于 *w* 发生（比如不是 hb(r, w) ），同时

- 没有写操作 *w'* 穿插在 *w* 和 *r* 中间（如不存在 hb(w, w') 和 hb(w', r)）

非正式地，如果没有 happens-before 关系阻止读操作 r，那么读操作 r 就能看到写操作 w 的结果

#### 从 JDK 1.5 前后的 volatile 了解 happen-before

```java
public class SharedObj {
  public int calcResult = -1; // 表示计算结果, -1表示还没计算呢
  public volatile boolean jobDone = false; // 表示计算任务是否已经做完
}

public CalcJob extends Thread {
  private SharedObj sharedObj;
  public CalJob(SharedObj so) {
    sharedObj = so;
  }
  
  public void run() {
    // 耗时很长的计算任务
    so.calcResult = xxxx; // 设置计算结果
    so.jobDone = true; // 表示任务已经完成了
  }
}

public ShowJob extends Thread {
  private SharedObj sharedObj;
  public ShowJob(SharedObj so) {
    sharedObj = so;
  }
  
  public void run() {
    while(!so.jobDone) {
      // 如果job没完，则等待。注意实际中尽量避免使用sleep。这里仅仅为示例。
      Thread.sleep(10000); 
    };
    System.out.println(so.calcResult);
  }
}

// ... 拼接代码
SharedObj so = new SharedObj;
new CalcJob(so).start();
new ShowJob(so).start();
```
在 JDK 1.5 之前，这段代码的结果很可能是：-1

因为在这里：

```java
public void run() {
    // 耗时很长的计算任务
    so.calcResult = xxxx; // 设置计算结果
    so.jobDone = true; // 表示任务已经完成了
}
```

会发生重排序，导致先执行了 `so.jobDone = true;`，即使是有加 `volatile` 关键字。

原因在于，JDK 1.5 前，*volatile* 只能保证声明的变量。

但在 JDK 1.5 之后，引入了 happens-before，使之不会随意重排序，保证整体的有序。

e.g.

```java
class So {
    Integer a;
    Integer b;
    Integer c;
    Integer d;
    volatile Integer x;
}

ThreadA: 
    so.a = 1, so.b = 2, so.c = 3, so.d = 4;
    so.x = 10;
ThreadB:
    int x = so.x;
    System.out.println(String.format("%d %d %d %d", so.a, so.b, so.c, so.d));
```

对于上面的代码，假如实际执行的顺序是 *ThreadA* 先执行完，*ThreadB* 再执行的，那么最终一定会输出 `1,2,3,4`。*ThreadA* 设置了 `so.x = 10` 后，*a*，*b*，*c*，*d* 还不一定对外可见。*ThreadB* 在尚未执行 `int x = so.x` 时，不一定能看见 *a*，*b*，*c*，*d*；但是一旦 *ThreadB* 执行了 `int x = so.x`, Happens-Before 保证，后续代码一定能看到 *a*，*b*，*c*，*d* 最新被修改的值

#### Happens-before 一致性（Happens-before Consistency）

对于表 17.4.5-A 中的过程，最初为 `A = B = 0` 。跟踪可以观察 `r2 = 0` 和 `r1 = 0` 并且仍然是 happens-before 一致的，因为存在允许每个读取看到适当的写入的执行顺序。

**表 17.4.5-A. Happens-before 一致性允许的行为但是连续一致性不允许**

| Thread1 | Thread2 |
| --- | --- |
| B = 1 | A = 2 |
| r2 = A | r1 = B |

因为没有同步，所以每次读取都可以看到初始值以及其他线程的写入，这样的执行顺序是：

1. B = 1
2. A = 2
3. r2 = A   // 看到写入初始值 0
4. r1 = B   // 看到写入初始值 0

另一个 happens-before 一致性的执行顺序是：

1. r2 = A   // 看到写入 A = 2
2. r1 = B   // 看到写入 B = 1
3. B = 1
4. A = 2

在这个执行顺序中，读取能够看到执行顺序中稍后发生的写入。这可能看起来违反直觉，但是这在 happens-before 一致性中是允许的，允许读取看到稍后的写入有时会产生不可接受的行为。

### Executions

### Well-Formed Executions

### Executions and Causality Requirements

### Observable Behavior and Nonterminating Executions

## JMM 是什么？

至此，以上便是 Java 8 JLS 中关于 JMM 的解释

JMM 是一种规范，定义了 Java 程序中多线程并发访问共享内存时的行为和规则。它描述了线程之间如何进行通信、内存如何同步以及如何保证数据的可见性。

JMM 主要解决的问题是在多线程环境下保证程序的正确性和一致性。在多线程编程中，由于线程之间的交替执行和并发访问共享数据，可能会引发一些问题，如线程安全问题、可见性问题、有序性问题等。JMM 提供了一组规则和机制来确保多线程程序的正确执行。

JMM 定义了以下几个重要的概念和规则：

-  主内存（Main Memory）：主内存是共享的内存区域，存储所有的变量数据。

- 工作内存（Working Memory）：工作内存是线程独立的内存区域，存储线程使用到的变量的副本。

- 内存间的交互操作：线程通过读写工作内存来访问变量，但变量的真实值存储在主内存中，线程之间的数据传输需要通过主内存进行。

- 原子性（Atomicity）：JMM 保证某些特定操作的原子性，即这些操作要么完全执行成功，要么完全不执行。

- 可见性（Visibility）：JMM 保证一个线程对变量的修改对其他线程可见。

- 有序性（Ordering）：JMM 保证程序执行的顺序按照代码的先后顺序执行，但不保证所有线程的执行顺序。

JMM 的规则和机制包括 volatile 关键字、synchronized 关键字、锁、happens-before 原则等，它们提供了线程之间的同步和通信机制，保证了共享数据的可靠性和一致性。遵循 JMM 的规则可以有效地避免多线程编程中的一些常见问题，如竞态条件、死锁、数据不一致等。

## 特别鸣谢

- [Java 并发基础之内存模型](https://www.javadoop.com/post/java-memory-model)

- [The Myth of volatileJDK1.5之前的volatileJDK1.5之后的volatilevolatile足够了吗？volatile VS 锁结论](https://cloud.tencent.com/developer/article/1121728)



[^1]: 指令重排序必须遵守一些安全规则，以确保在优化指令执行顺序时不改变程序的语义和行为,重排序的安全规则：1. 数据依赖性（Data Dependency）：如果两个操作之间存在数据依赖关系，那么这两个操作的重排序是受限制的。在单线程环境下，编译器和处理器必须确保后续的读操作在先前的写操作之后执行。这样可以保证程序的语义不变。2. as-if-serial 语义（As-If-Serial Semantics）：指令重排序不能改变单线程程序的执行结果。程序的执行结果必须与在指令没有重排序的情况下的执行结果一致。即使编译器和处理器对指令进行了优化和重排序，程序的外部观察结果也必须与按照原始顺序执行的结果相同。3. 顺序一致性（Sequential Consistency）：重排序不能违反程序的顺序一致性。在多线程环境下，编译器和处理器必须确保对共享变量的操作遵循全局的顺序一致性，即所有线程都能观察到相同的操作顺序。