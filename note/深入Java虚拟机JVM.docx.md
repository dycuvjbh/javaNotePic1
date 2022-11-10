# 深入Java虚拟机 - JVM

## JVM简介

### 1.1 JVM所处的位置

![](./images/media/image1.png){width="5.118678915135608in"
height="3.52413823272091in"}

JVM是运行在操作系统之上的，它与硬件没有直接的交互。

### 1.2 虚拟机是Java平台无关的保障

![](./images/media/image2.png){width="5.999305555555556in"
height="2.823611111111111in"}

不同的操作系统上安装了不同的版本【Linux版/Windows版/Mac版】的JDK，实际上是由JVM虚拟机屏蔽掉了与平台无关的信息。

**JVM不仅屏蔽了平台相关性，也屏蔽掉了语言相关性**。JVM虚拟机不关注到底是哪种语言生成的class文件，只要是复合JVM规范的class字节码文件都可以被JVM虚拟机解释执行。

**总之，目前JVM不仅仅支持跨平台的语言java，目前也逐步走向跨语言的平台**。

**JVM是一种规范，文档下载地址：<https://docs.oracle.com/en/java/javase/13/>**

### 1.3常见虚拟机实现

**Hotspot: oracle官方,做实验用的JVM。也是我们目前正在使用的**。

Jrockit： 号称世界上最快的JVM,被Oracle收购，合并于Hotspot

J9： IBM公司

Microsoft VM:

TaobaoVM: hotspot深度定制版

LiquidVM: 直接针对硬件

azul zing: 最新垃圾回收的业界标杆。

## JVM运行时内存结构

我们都知道，Java代码是要运行在虚拟机上的，而虚拟机在执行Java程序的过程中会把所管理的内存划分为若干个不同的数据区域，这些区域都有各自的用途。其中有些区域随着虚拟机进程的启动而存在，而有些区域则依赖用户线程的启动和结束而建立和销毁。

在《Java虚拟机规范（Java SE 8）》中描述了JVM运行时内存区域结构如下：

**JVM内存结构，由Java虚拟机规范定义。描述的是Java程序执行过程中，由JVM管理的不同数据区域。各个区域有其特定的功能。**

### JVM整体结构

JVM体系结构：

![](./images/media/image3.png){width="5.032638888888889in"
height="4.543055555555555in"}

**Jvm体系结构**\[**目前网上没有**\]：**重点学习这个**

![](./images/media/image4.png){width="5.833333333333333in"
height="2.4269674103237096in"}

#### 1.1.1. 功能区域

**类加载子系统(类加载器)**

作用 : 类加载器（class loader）用来加载 Java 类到 Java 虚拟机中.

一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源程序（.java
文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class
文件）。类加载器负责读取 Java
字节代码，并转换成`java.lang.Class`类的一个实例。每个这样的实例用来表示一个
Java 类。通过此实例的 `newInstance()`方法就可以创建出该类的一个对象。

![](./images/media/image5.png){width="4.335732720909887in"
height="3.392069116360455in"}

**ClassLoader的分类:四种**

![](./images/media/image6.png){width="5.995833333333334in"
height="2.6708333333333334in"}

Bootstrap ClassLoader :引导类加载器

Extension ClassLoader:扩展类加载器

App ClassLoader:应用类加载器

User-Defined ClassLoader:用户自定以类加载器（实际开发中也不用）

1\. Bootstrap ClassLoader

由C++写的,由JVM启动.

启动类加载器,负责加载java基础类，对应的文件是%JRE_HOME/lib/
目录下的rt.jar、resources.jar、charsets.jar和class等

2.Extension ClassLoader

Java类,继承自URLClassLoader 扩展类加载器,

对应的文件是 %JRE_HOME/lib/ext 目录下的jar和class等

3.App ClassLoader

Java类,继承自URLClassLoader 系统类加载器,

对应的文件是应用程序classpath目录下的所有jar和class等

常见面试题1：java双亲委派机制\[原理\]及作用:参考Excel文件

**垃圾回收系统**

作用 : 负责收集垃圾, 销毁, 回收内存. 对java堆, 方法区,
直接内存进行垃圾回收.

具体垃圾回收系统中的内容比较重点, 比较多, 稍后章节中详细讲解.

**执行引擎:负责将字节码转换成机器码**

作用 :
执行编译后的字节码指令.\[进来的是字节码，出去的是机器码:0101代码\]

主要的执行技术有 : 解释执行，即时编译执行，自适应优化执行,
芯片级直接执行

（1）解释：属于第一代JVM；

（2）即时编译：JIT属于第二代JVM；

（3）自适应优化：（目前Sun的HotspotJVM采用这种技术）吸取第一代JVM和第二代JVM的经验，采用两者结合的方式。

开始对所有的代码都采取解释执行的方式，

并监视代码执行情况，然后对那些 经常
调用的方法启动一个后台线程，将其编译为本地代码，并进行仔细优化。

若方法不再频繁使用，则取消编译过的代码，仍对其进行解释执行；

4.  芯片级直接执行：内嵌在芯片上，用本地方法执行Java字节码。

#### 1.2.1. 线程私有区域

**java栈**

作用 :
栈也叫栈内存,线程私有，它的生命周期与线程相同,线程结束栈内存也就释放，**对于栈来说不存在垃圾回收问题**。存储局部变量,
动态链接, 方法出口等信息.

**栈运行原理**：

栈中的数据都是以栈帧（Stack
Frame）的格式存在，栈帧是一个内存区块，是一个数据集，是一个有关方法(Method)和运行期数据的数据集，当一个方法A被调用时就产生了一个栈帧
F1，并被压入到栈中，

A方法又调用了 B方法，于是产生栈帧 F2 也被压入栈，

B方法又调用了 C方法，于是产生栈帧 F3 也被压入栈，

......

执行完毕后，先弹出F3栈帧，再弹出F2栈帧，再弹出F1栈帧......

**遵循"先进后出"/"后进先出"原则**。

![](./images/media/image7.png){width="5.990972222222222in"
height="2.660416666666667in"}

**栈帧中主要保存3 类数据**：

本地变量（Local Variables）:输入参数和输出参数以及方法内的变量；

栈操作（Operand Stack）:记录出栈、入栈的操作；

栈帧数据（Frame Data）:包括类文件、方法\[父栈，子栈\]等等。

提示：如果我们写的代码有问题，造成循环引用，比如：递归,有可能会导致栈内存溢出问题：

![](./images/media/image8.png){width="5.997916666666667in"
height="2.901388888888889in"}

**结论：由于以上原因会导致栈内存溢出问题，所以通常，如果报栈内存溢出异常，就检查自己的代码是否有循环引用**。

**Native Interface:本地接口**

本地接口的作用是融合不同的编程语言为 Java 所用，它的初衷是融合
C/C++程序，Java 诞生的时候是 C/C++横行的时候，要想立足，必须有调用
C/C++程序，于是就在内存中专门开辟了一块区域处理标记为native的代码。目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过Java程序驱动打印机或者Java系统管理生产设备，在企业级应用中已经比较少见。因为现在的异构领域间的通信很发达，比如可以使用
Socket通信，也可以使用Web Service等等，不多做介绍。

**Native Method Stack:本地方法栈**

它的具体做法是 Native Method Stack中登记 native方法，在Execution Engine
执行时加载native libraies。

**注意: (Sun HotSpot虚拟机）直接就把本地方法栈和Java栈合二为一**.

顺带提一句：JVM的类型(3种)：

Sun公司的HotSpot、BEA公司的JRockit、IBM公司的J9 VM

**PC寄存器(程序计数器):**

作用 :
每个线程启动的时候，都会创建一个PC寄存器。保存下一条将要执行的指令地址,pc寄存器是服务于cpu的.
PC寄存器的内容总是指向下一条将被执行指令的地址，由执行引擎读取下一条指令，PC寄存器是一个非常小的内存空间，几乎可以忽略不记。

#### 1.3.1. 线程共享区域

**方法区**

作用 :
Java方法区和堆一样，方法区是一块所有线程共享的内存区域，他保存系统的类信息。比如类的字段、方法、常量池以及一些特殊方法如构造函数，接口代码也在此定义。

**注意：Cart cart = new Cart();
在堆内存中，实例变量存在堆内存中,和方法区无关。**

方法区的大小决定系统可以保存多少个类。如果系统定义太多的类，导致方法区溢出。虚拟机同样会抛出内存溢出的错误。

**java堆**

作用 : **堆内存用于存放由new创建的对象和数组**。

在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，在栈中的这个特殊的变量就变成了数组或者对象的引用变量，以后就可以在程序中使用栈内存中的引用变量来访问堆中的数组或者对象，引用变量相当于为数组或者对象起的一个别名或者代号

![](./images/media/image9.png){width="5.995833333333334in"
height="2.390972222222222in"}

**整个堆大小：新生代+老年代**

**新生代: Eden+存活区\[同时只能使用一个，加1个存活区大小。\]**

**直接内存:**

**介绍：也称为堆外内存，直接在内存中申请，而不是在JVM虚拟机中申请内存，受限于服务器物理内存大小，不能调节。**

作用 : 提高一些场景中的性能.给NIO（New
IO或者非阻塞IO,普通IO称支持阻塞IO,）使用的,netty 就是这种机制。

1.阻塞IO： 资源不可用时，IO请求一直阻塞，直到反馈结果（有数据或超时）。\
2.非阻塞IO：资源不可用时，IO请求离开返回，返回数据标识资源不可用

直接内存并不是虚拟机运行时数据区的一部分，也不是Java
虚拟机规范中农定义的内存区域。在JDK1.4 中新加入了NIO(New
Input/Output)类，引入了一种基于通道(Channel)与缓冲区（Buffer）的I/O
方式，它可以使用native
函数库直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer
对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native堆中来回复制数据。

![](./images/media/image10.png){width="5.997222222222222in"
height="3.2in"}

本机直接内存的分配不会受到Java
堆大小的限制，受到本机总内存大小限制，具体大小可以在编写NIO代码时，直接向物理内存申请。

当然也可以通过-XX:MaxDirectMemorySize设置最大直接内存大小。

配置虚拟机参数时，不要忽略直接内存 防止出现OutOfMemoryError异常。

**直接内存（堆外内存）与堆内存比较**

直接内存申请空间耗费更高的性能，当频繁申请到一定量时尤为明显
**直接内存IO读写的性能要优于普通的堆内存**，在多次读写操作的情况下差异明显

**代码如下：使用ByteBuffer直接分配1G的直接内存**

**public class** DirectMemoryTest {\
**private static final int *BUFFER***=1024\*1024\*1024; **//1GB\
public static void** main(String\[\] args) {\
ByteBuffer byteBuffer = ByteBuffer.*allocate*(***BUFFER***);\
System.***out***.println(**\"直接内存分配完毕，请求指示!\"**);\
\
Scanner scanner = **new** Scanner(System.***in***);\
scanner.next();\
\
System.***out***.println(**\"直接内存释放开始\"**);\
byteBuffer=**null**;\
System.*gc*();\
scanner.next();\
}\
}

**可以直接查看分配的1G的直接内存!**

![](./images/media/image11.png){width="6.0in"
height="3.198611111111111in"}

**提示常见面试题1：JVM是如何调优的？**

JVM调优主要从两个方面：1.内存管理、2、垃圾回收。

其中内存管理部分，我们只能管理JVM内存部分，不能管理直接内存部分。

**提示常见面试题2：JVM组成？**

1.  ### 堆中内部结构

    JDK7及以前版本：堆内存**逻辑上**分为三部分：新生+养老+永久

![](./images/media/image12.png){width="5.833333333333333in"
height="2.339571303587052in"}

**JDK8开始取消永久代, 使用元数据区替代永久代, 元数据区不在堆中,
是堆外的一块物理内存. Eden:Survivor from:Survivor to的比例是8:1:1**

![](./images/media/image13.png){width="5.846527777777778in"
height="2.8069444444444445in"}

方法区也是所有线程共享。主要用于存储类的信息、常量池、静态变量、及时编译器编译后的代码等数据。**方法区逻辑上属于堆的一部分**，但是为了与堆进行区分，通常又叫"**非堆(Non-Heap)**"。

PermGen space（永久区） 和
Metaspace(元空间)是HotSpot对于方法区的不同实现,在Java虚拟机(以下简称JVM)中，类包含其对应的元数据，比如类名,父类名,类的类型,访问修饰符,字段信息,方法信息,静态变量,常量,类加载器的引用,类的引用。在HotSpot
JDK 1.8之前这些类元数据信息存放在一个叫永久代的区域(PermGen
space)，永久代一段连续的内存空间。在JDK
1.8开始，方法区实现采用Metaspace代替，这些元数据信息直接使用本地内存来分配。**元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存**。方法区还有一个别名:Non-Heap(非堆)

根据垃圾回收机制的不同，Java堆有可能拥有不同的结构，最为常见的就是将整个Java堆分为新生代和老年代。其中新生代存放新生的对象或者年龄不大的对象，老年代则存放老年对象。

新生代分为eden区、s0区、s1区，s0和s1也被称为from和to区域，他们是两块大小相等并且可以互相变换角色的空间，**to区总为空**。

绝大多数情况下，对象首先分配在eden区，在新生代回收后，如果对象还存活，则进入s0或s1区，之后每经过一次新生代回收，如果对象存活则它的年龄就加1，对象达到一定的年龄(15次数)后，则进入老年代。

代码如下：

**public class** HeapTest {\
**byte**\[\] **buffer**=**new byte**\[**new**
Random().nextInt(1024\*200)\];\
**public static void** main(String\[\] args) {\
ArrayList\<HeapTest\> list =**new** ArrayList\<HeapTest\>();\
\
**while** (**true**){\
list.add(**new** HeapTest());\
\
**try** {\
Thread.*sleep*(10);\
} **catch** (Exception e) {\
e.printStackTrace();\
}\
}\
}\
}

设置Heap占用的最大最小内存:

![](./images/media/image14.png){width="5.995138888888889in"
height="1.761111111111111in"}

通过命令行工具：jvisualvm查看堆空间变化。

![](./images/media/image15.png){width="5.997916666666667in"
height="3.283333333333333in"}

当然上述代码也可以使用Jprofiler分析：

**提示:**安装11版本的Jprofiler、安装这个软件的时候，它会提示选择相应版本的idea，这样就自动将对应版本的idea的插件安装好。

![](./images/media/image16.png){width="5.9944444444444445in"
height="3.017361111111111in"}

就能看到各种监控指标：Cpu、Memory、GC活跃度、类、线程等。

![](./images/media/image17.png){width="5.9944444444444445in"
height="3.6055555555555556in"}

![](./images/media/image18.png){width="5.997222222222222in"
height="3.540277777777778in"}

运行到最后，老年代没有空间的时候，就会报出异常：

![](./images/media/image19.png){width="5.875in"
height="0.7708333333333334in"}

**提示1：为什么JVM新生代中有两个survivor**？

答案：**为了保证任何时候总有一个survivor是空的**。

因为将eden区的存活对象复制到survivor区时，必须保证survivor区是空的，如果survivor区中已有上次复制的存活对象时，这次再复制的对象肯定和上次的内存地址是不连续的，会产生内存碎片，浪费survivor空间。

而如果有两个survivor区，第一次GC后，把eden区和survivor0区一起复制到survivor1区，然后清空survivor0和eden区，此时survivor1非空，survivor0和eden区为空，下一次GC时把survivor0和survivor1交换，这样就能保证向survivor区复制时始终都有一个survivor区是空的，也就能保证新对象能始终在eden区出生。

**提示2：什么时候Eden去对象会跑到Survivor区或者Old区?**

GC:分为普通GC(minor GC:**针对Edon区**)和**Major
GC**(**:针对养老区**)+FullGC三种

首先说如果没有Survivor区会出现什么情况：此时每触发一次Minor
GC，就会把Eden区的对象复制到老年代，这样当老年代满了之后会触发Major
Gc(通常伴随着MinorGC，可以看做**Full GC=Minor GC + Major
GC+方法区清理**)，比较耗时。

**通常情况下:**

所有的新生代首先会在Eden区进行内存分配，当Eden区满时会进行一次Minor
GC操作，将Eden区进行回收，此时判断存活的对象会被复制进入Survivor
from区（年龄加1），对于大对象直接进入老年代，实际上是为了保证Eden区具有充足的空间可用的一种策略，采用-XX:PretenureSizeThreshold参数可以设置多大的对象可以直接进入老年代内存区域。对于长期存活的对象直接进入老年代，实际上时对Eden区到Survivor区过度的一种策略，是为了保证Eden区到Survivor区不会频繁的进行复制一直存活的对象且对Survivor区也能保证不会具有太多的一直占据的内存，采用-XX:MaxTenuringThreshold=数字
参数可以设置对象在经过多少次普通GC后会被放入老年代（年龄达到设置值，默认为15）。

**小结**：

Eden园区：每次满了，进行Minor GC，进入Survivor区

Survivor区：默认经过15次普通GC后，对象依旧存在，则进入养老区

Old Memory:养老区满了，会发生MajorGC\[Full GC\],如果Full
GC后，依旧无法存储对象，会产生OOM异常。需要注意的是：对象不会进入永久区。

**提示3:若养老区执行了Full
GC之后发现依然无法进行对象的保存，就会产生OOM异常"OutOfMemoryError"\[堆溢出异常\]。**

如果出现java.lang.OutOfMemoryError: Java heap
space异常，说明Java虚拟机的堆内存不够。原因有二：

（1）Java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整\[解决方案\]。

（2）代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。

**提示4:方法区、元空间、永久区关系**

`Jdk1.6及之前： 有永久代, 常量池1.6在方法区`

`Jdk1.7：       有永久代，但已经逐步“去永久代”，常量池1.7在堆`

`Jdk1.8及之后： 无永久代，常量池1.8在元空间`

**堆溢出代码演示**：

public static void main(String\[\] args) {

List\<byte\[\]\> list = new ArrayList\<\>();

int i=0;

while(true){

list.add(new byte\[5\*1024\*1024\]);

System.out.println(\"分配次数：\"+(++i));

}

}

### 1.3. 栈中内部结构

![](./images/media/image20.png){width="5.833333333333333in"
height="2.835353237095363in"}

**作用 :** 线程私有，它的生命周期与线程相同。存储局部变量, 动态链接,
方法出口等信息.

**栈中放置以下内容**

栈中存放基本类型的原值和引用类型的地址值.

基本类型包括：byte,short,int,long,char,float,double,Boolean,returnAddress
引用类型包括：类类型，接口类型和数组。

**栈的内部结构如下:**

栈由一个一个栈帧组成, 栈帧中的结构又由局部变量表, 操作数栈,
帧数据区组成.

局部变量表 : 保存函数参数,局部变量(当前函数有效,函数执行结束它销毁)

操作数栈 : 存中间运算结果, 临时存储空间

帧数据区 : 保存访问常量池指针, 异常处理表

**栈溢出异常：通常发生递归的时候会报这种异常,没有结束条件**

**public class** StackOverTest {\
**private int num**=0;\
**private int** sum(**int** a,**int** b){\
**num**++;\
**return** sum(a+**num**, b);\
}\
**public static void** main(String\[\] args) {\
StackOverTest stackOverTest = **new** StackOverTest();\
**try** {\
stackOverTest.sum(1,2);\
} **catch** (Exception e) {\
e.printStackTrace();\
}**finally** {\
System.***out***.println(**\"=========num:\"**+stackOverTest.**num**);\
}\
}\
}

可以通过修改虚拟机的:-Xss1m设置每个栈占用的内存大小为256k，**该值会影响栈的深度，及线程数\[值越大，JVM内存中容纳的线程数就越少\]**。

![](./images/media/image21.png){width="5.995833333333334in"
height="1.7472222222222222in"}

再次执行main方法会发现:输出的num值变小了。

### 1.4. 案例 : 堆、栈、方法区关系

有一个类HeapDemo, 代码如下:

    public class HeapDemo {
    	
        //成员变量
        private int id;

        //构造方法
        public HeapDemo(int id) {
            this.id = id;
        }

        //普通方法
        public void see() {
            System.out.println("我的id是:" + id);
        }

        //主线程入口
        public static void main(String[] args) {
            HeapDemo d1 = new HeapDemo(1);
            HeapDemo d2 = new HeapDemo(2);
            d1.see();
            d2.see();
        }
    }

**上面代码在java堆, java栈, 方法区中的使用情况如下图**

每一个Java类，在被JVM加载的时候，JVM会给这个类创建一个`instanceKlass`对象，保存在方法区，用来在JVM层表示该Java类。当我们在Java代码中，使用new创建一个对象的时候，JVM会创建一个`instanceOopDesc`对象，这个对象中包含了对象头以及实例数据。

![](./images/media/image22.jpeg){width="5.833333333333333in"
height="2.6154483814523184in"}

这就是一个简单的Java对象的OOP-Klass模型，即Java对象模型。

**或者如下图所示**：

![](./images/media/image23.png){width="5.827083333333333in"
height="2.222916666666667in"}

HotSpot是使用指针的方式来访问对象：

Java堆中会存放访问类元数据的地址，

reference存储的就直接是对象的地址

**元空间异常：**

如果出现java.lang.OutOfMemoryError: PermGen
space，说明是Java虚拟机对永久代Perm内存设置不够。一般出现这种情况，都是程序启动需要加载大量的第三方jar包（我的电脑以前测试时大约是41165个类就会出现元空间异常）。例如：在一个Tomcat下部署了太多的应用。或者大量动态反射生成的类不断被加载，最终导致Perm区被占满。

## 3. 堆参数调优入门 {#堆参数调优入门 .list-paragraph}

**均以JDK1.8+HotSpot为例**

jdk1.7：

![](./images/media/image24.png){width="5.833333333333333in"
height="2.547820428696413in"}

jdk1.8：

![](./images/media/image25.png){width="5.833333333333333in"
height="2.349449912510936in"}

### 3.1. 常用JVM参数

怎么对jvm进行调优？通过**参数配置**

  ----------------------------------------------------------------------------------------------
  **参数**                            **备注**
  ----------------------------------- ----------------------------------------------------------
  -Xms                                初始堆大小。只要启动，就占用的堆大小，默认是内存的1/64。

  -Xmx                                最大堆大小。默认是内存的1/4

  -Xmn                                新生区堆大小

  -XX:+PrintGCDetails                 输出详细的GC处理日志

  -XX:MetaspaceSize=128m              设置元空间初始大小 例如：128M

  -XX:MaxMetaspaceSize=128m           设置元空间最大值大小,默认没有限制
  ----------------------------------------------------------------------------------------------

java代码查看jvm堆的默认值大小：

    Runtime.getRuntime().maxMemory()   // 堆的最大值，默认是内存的1/4
    Runtime.getRuntime().totalMemory()  //堆的当前总大小

System.***in***.read();//停住，便于监控

### 3.2. 怎么设置JVM参数

idea运行时设置方式如下：

![](./images/media/image26.png){width="5.833333333333333in"
height="2.0519258530183726in"}

![](./images/media/image27.png){width="5.833333333333333in"
height="3.8907174103237097in"}

### 3.3. 查看堆内存详情

    public class Demo2 {
        public static void main(String[] args) {

            System.out.print("java虚拟机试图使用的最大堆内存量：");
            System.out.println(Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");
            System.out.print("java虚拟机中堆内存总量：");
            System.out.println(Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");
            System.out.println("==================================================");

            byte[] b = null;
            for (int i = 0; i < 10; i++) {
                               Thread.sleep(2000);
                b = new byte[1 * 1024 * 1024];
            }
        }
    }

执行前配置参数：-Xmx50m -Xms30m -XX:+PrintGCDetails

执行：看到如下信息

![](./images/media/image28.png){width="5.833333333333333in"
height="2.221059711286089in"}

新生代和老年代的堆大小之和是Runtime.getRuntime().totalMemory()

查看GC有两种方式：

第一种：-XX:+PrintGCDetails: 表示打印GC情况

第一种: 在命令行黑窗口：jps查看进程id \| jstat -gc 进程id(pid)

![](./images/media/image29.png){width="6.440972222222222in"
height="1.0409722222222222in"}

![](./images/media/image30.png){width="5.991666666666666in"
height="2.0902777777777777in"}

**当然，也可以通过jvisualvm查看我们设置的完整heap的大小**。

### 3.4. GC演示

    public class HeapDemo1 {

        public static void main(String args[]) {
            System.out.println("=====================Begin=========================");
            System.out.print("最大堆大小：Xmx=");
            System.out.println(Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");

            System.out.print("剩余堆大小：free mem=");
            System.out.println(Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");

            System.out.print("当前堆大小：total mem=");
            System.out.println(Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");

            System.out.println("==================First Allocated===================");
            byte[] b1 = new byte[5 * 1024 * 1024];
            System.out.println("5MB array allocated");

            System.out.print("剩余堆大小：free mem=");
            System.out.println(Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");

            System.out.print("当前堆大小：total mem=");
            System.out.println(Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");

            System.out.println("=================Second Allocated===================");
            byte[] b2 = new byte[10 * 1024 * 1024];
            System.out.println("10MB array allocated");

            System.out.print("剩余堆大小：free mem=");
            System.out.println(Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");

            System.out.print("当前堆大小：total mem=");
            System.out.println(Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");

            System.out.println("=====================OOM=========================");
            System.out.println("OOM!!!");
            System.gc();
            byte[] b3 = new byte[40 * 1024 * 1024];
        }
    }

jvm参数设置成最大堆内存100M，当前堆内存10M：-Xmx100m -Xms10m
-XX:+PrintGCDetails

再次运行，可以看到minor GC和full GC日志：

![](./images/media/image31.png){width="5.833333333333333in"
height="1.3545778652668417in"}

### 3.5. OOM演示

把上面案例中的jvm参数改成最大堆内存设置成50M，当前堆内存设置成10M，执行测试：
-Xmx50m -Xms10m

    =====================Begin=========================
    最大堆大小：Xmx=44.5M
    剩余堆大小：free mem=8.186859130859375M
    当前堆大小：total mem=9.5M
    =================First Allocated=====================
    5MB array allocated
    剩余堆大小：free mem=3.1868438720703125M
    当前堆大小：total mem=9.5M
    ================Second Allocated====================
    10MB array allocated
    剩余堆大小：free mem=3.68682861328125M
    当前堆大小：total mem=20.0M
    =====================OOM=========================
    OOM!!!
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
    	at com.atguigu.demo.HeapDemo.main(HeapDemo.java:40)

### 3.6. OOM溢出分析

第一步代码：

**public class HeapAnalyseDemo**{\
**public static void** main(String\[\] args) {\
String str = **\"www.atguigu.com\"** ;\
**while**(**true**)\
{\
str += str + **new** Random().nextInt(88888888) + **new**
Random().nextInt(999999999) ;\
}\
}\
}

第二步设置：

-Xms1m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath=D:\\

![](./images/media/image32.png){width="5.996527777777778in"
height="1.0541666666666667in"}

第三步: 使用jvisualvm.exe解读dump文件

![](./images/media/image33.png){width="5.889376640419948in"
height="4.443003062117235in"}

其中-XX:+HeapDumpOnOutOfMemoryError:OOM时导出堆到文件

-XX:HeapDumpPath=D:\\ ：指定导出文件的路径

![](./images/media/image34.png){width="5.872222222222222in"
height="3.183333333333333in"}

## 4. 垃圾回收系统

### 4.1. 垃圾回收算法:

**基本垃圾回收算法有四种 : 引用计数法, 标记清除法, 标记压缩法,
复制算法**

现代流行的垃圾收集算法一般是由这四种中的其中几种算法相互组合而成.

例如: 分代算法, 分区算法

#### 4.1.1. 引用计数法(基本)

**原理**

引用计数算法很简单，它实际上是通过在对象头中分配一个空间来保存该对象被引用的次数。如果该对象被其它对象引用，则它的引用计数加一，如果删除对该对象的引用，那么它的引用计数就减一，当该对象的引用计数为0时，那么该对象就会被回收。

**案例**

有代码如下:

    String p = new String("abc");

abc这个字符串对象的引用计数值为1.

![](./images/media/image35.png){width="5.833333333333333in"
height="4.3836975065616794in"}

将引用变量p的值设置为0

    p = null;

![](./images/media/image36.png){width="5.833333333333333in"
height="3.285133420822397in"}

**小结**

引用计数垃圾收集机制，它只是在引用计数变化为0时即刻发生，而且只针对某一个对象以及它所依赖的其它对象。所以，我们一般也称呼引用计数垃圾收集为**直接**的垃圾收集机制.垃圾收集的开销被分摊到整个应用程序的运行当中了，而不是在进行垃圾收集时，要挂起整个应用的运行，直到对堆中所有对象的处理都结束。因此，采用引用计数的垃圾收集不属于严格意义上的\"Stop-The-World\"的垃圾收集机制。

优点:

实时性较高, 不需要等到内存不够时才回收

垃圾回收时不用挂起整个程序, 不影响程序正常运行.

缺点:

回收时不移动对象, 所以会造成内存碎片问题.

解决不了对象相互引用(形成垃圾对象)

#### 4.1.2. 标记清除法(基本)

**原理**

它的做法是当堆中的有效内存空间被耗尽的时候，就会停止整个程序（也被成为stop
the world），然后进行两项工作，第一项则是标记，第二项则是清除。

标记：标记的过程其实就是，从根对象开始遍历所有的对象，然后将所有存活的对象标记为可达的对象。

清除：清除的过程将遍历堆中所有的对象，将没有标记的对象全部清除掉。

![](./images/media/image37.png){width="5.992361111111111in"
height="1.8319444444444444in"}

**小结**

优点 : 实现简单

缺点 :

效率低, 因为标记和清除两个动作都要遍历所有的对象

垃圾收集后有可能会造成大量的内存碎片, 垃圾回收时会造成应用程序暂停.

#### 4.1.3. 标记压缩法(基本)

**原理**

既然叫标记-压缩算法，那么它也分为两个阶段，一个是标记(mark)，一个是压缩(compact).

标记压缩算法是在标记清除算法的基础之上，做了优化改进的算法。和标记清除算法一样，也是从根节点开始，对对象的引用进行标记，在清理阶段，并不是简单的清理未标记的对象，而是将存活的对象压缩到内存的一端，然后清理边界以外的垃圾，从而解决
了碎片化的问题。

标记 :
标记的过程其实就是，从根对象开始遍历所有的对象，然后将所有存活的对象标记为可达的对象。

压缩 :
移动所有的可达对象到堆内存的同一个区域中，使他们紧凑的排列在一起，从而将所有非可达对象释放出来的空闲内存都集中在一起，通过这样的方式来达到减少内存碎片的目的。

![](./images/media/image38.png){width="5.996527777777778in"
height="2.1479166666666667in"}

**小结**

优点 :

标记压缩算法是对标记清除算法的优化, 解决了碎片化的问题

缺点 :

还是效率问题, 在标记清除算法上又多加了一步, 效率可想而知了

#### 4.1.4. 复制算法(基本)

**原理**

复制算法的核心就是，将原有的内存空间一分为二，每次只用其中的一块，在垃圾回收时，将正在使用的对象复制到另一个内存空间中，并以此排列,
然后将该内存空间清空，交换两个内存的角色，完成垃圾的回收。

![](./images/media/image39.png){width="5.995138888888889in"
height="1.88125in"}

**小结**

优点:

在垃圾多的情况下(新生代), 效率较高

清理后, 内存无碎片

缺点:

浪费了一半的内存空间

在存活对象较多的情况下(老年代), 效率较差

#### 4.1.5. 分代模型

**原理**

前面介绍了多种回收算法，每一种算法都有自己的优点也有缺点，谁都不能替代谁，所以根据垃圾回收对象的特点进行选择，才是明智的选择。

分代算法其实就是这样的，根据回收对象的特点进行选择，在jvm中，

-   **新生代适合使用复制算法**，

-   **老年代适合使用标记清除或标记压缩算法**

![](./images/media/image12.png){width="5.833333333333333in"
height="2.339571303587052in"}

#### 4.1.6. 分区模型

**原理**

上面介绍的分代收集算法是将对象的生命周期按长短划分为两个部分,
而分区算法则将整个堆空间划分为连续的不同小区间, 每个小区间独立使用,
独立回收. 这样做的好处是可以**控制一次回收多少个小区间**. 在相同条件下,
堆空间越大, 一次GC耗时就越长, 从而产生的停顿也越长.
为了更好地控制GC产生的停顿时间, 将一块大的内存区域分割为多个小块,
根据目标停顿时间, 每次合理地回收若干个小区间(而不是整个堆),
从而减少一次GC所产生的停顿.

![](./images/media/image40.png){width="5.833333333333333in"
height="2.5413429571303587in"}

### 4.3. 垃圾回收器分类（了解）

#### 4.3.1. 串行垃圾回收器

新生代串行回收器SerialGC : 采用复制算法实现, 单线程垃圾回收,
**独占式**垃圾回收器

老年代串行回收器SerialOldGC : 采用标记压缩算法, 单线程独占式垃圾回收器。

**串行收集器示意图**：

![](./images/media/image41.png){width="5.993055555555555in"
height="3.5243055555555554in"}

#### 4.3.2. 并行垃圾回收器

新生代parNew回收器: 采用复制算法实现, 多线程回收器, 独占式垃圾回收器.

新生代ParallelScavengeGC回收器: 采用复制算法多线程独占式回收器

老年代ParallelOldGC回收器: 采用标记压缩算法, 多线程独占式回收器

![](./images/media/image42.png){width="5.645833333333333in"
height="2.1354166666666665in"}

###### 4.3.2.1. CMS回收器

CMS全称 (Concurrent Mark
Sweep)，是一款并发的、使用标记-清除算法的垃圾回收器.

**启用CMS回收器参数 :** -XX:+UseConcMarkSweepGC。

**使用场景：**

GC过程短暂停，适合对时延要求较高的服务，用户线程不允许长时间的停顿。

**优点**:

CMS收集器是一种以获取最短回收停顿时间为目标的收集器.

并发收集，低停顿

**缺点：**

服务长时间运行，造成严重的内存碎片化。算法实现比较复杂

CMS收集器对CPU资源非常敏感

###### 4.3.2.2. G1回收器

**G1(Garbage-First)是一款面向服务端应用的并发垃圾回收器,
主要目标用于配备多颗CPU的服务器治理大内存. 是 JDK1.7
提供的一个新收集器，是当今收集器技术发展的最前沿成果之一**

G1计划作为并发标记-清除收集器的长期替代品

启用G1收集器参数: -XX:+UseG1GC 启用G1收集器.

G1将整个Java堆划分为多个大小相等的独立区域(Region),
虽然还保留有新生代和老年代的概念, 但新生代和老年代不再是物理隔离的了,
它们都是一部分Region(不需要连续)的集合.

![](./images/media/image43.png){width="5.833333333333333in"
height="3.1618350831146107in"}

每块区域既有可能属于Old区、也有可能是Young区,
因此不需要一次就对整个老年代/新生代回收.
而是**当线程并发寻找可回收的对象时,
有些区块包含可回收的对象要比其他区块多很多.
虽然在清理这些区块时G1仍然需要暂停应用线程,
但可以用相对较少的时间优先回收垃圾较多的Region**(这也是G1命名的来源).
这种方式保证了G1可以在有限的时间内获取尽可能高的收集效率.

**特点**

1.  一整块堆内存被分成多个独立的区域Regions

2.  存活对象被拷贝到新的Survivor区

3.  新生代内存由一组不连续的堆 heap 区组成，使得可以动态调整各个区域

4.  多线程并发 GC

5.  young GC 会有 STW 事件

### 4.4. 垃圾回收器对比

#### 4.4.1. 新生代回收器

  ------------------------------------------------------------------------------------------------
  **名称**             **串行/并行/并发**   **回收算法**   **适用场景**        **可以与cms配合**
  -------------------- -------------------- -------------- ------------------- -------------------
  SerialGC             串行                 复制           单cpu               是

  ParNewGC             并行                 复制           多cpu               是

  ParallelScavengeGC   并行                 复制           多cpu且关注吞吐量   否
  ------------------------------------------------------------------------------------------------

#### 4.4.2. 老年代回收器

+-----------------+-----------------+-----------------+-----------------+
| **名称**        | **串            | **回收算法**    | **适用场景**    |
|                 | 行/并行/并发**  |                 |                 |
+=================+=================+=================+=================+
| SerialOldGC     | 串行            | 标记整理        | 单cpu           |
+-----------------+-----------------+-----------------+-----------------+
| ParOldGC        | 并行            | 标记整理        | 多cpu           |
+-----------------+-----------------+-----------------+-----------------+
| CMS             | 并发,几乎不     | 标记清除        | 多cpu且         |
|                 | 会暂停用户线程  |                 |                 |
|                 |                 |                 | 与用户线程共存  |
+-----------------+-----------------+-----------------+-----------------+

小结如下图所示：

![](./images/media/image44.png){width="5.99375in"
height="3.645138888888889in"}

新生代和老年代的垃圾回收器一般是协调工作、搭配干活的（例如：新生代Seirial、老年代就用Serial
old）。具体是怎么搭配的参考图中新生代和老年代的连线。其中：**G1垃圾回收器同时支持新生代和老年代**。

图中**老年代部分**有一个特殊连线，CMS与Serial
Old连线，表示老年代的垃圾回收器CMS不能正常工作的时候，就把老年代的垃圾回收器切换为：Serial
Old。

可以在安装了jdk的环境下，执行如下命令查看JDK到底使用的是哪种回收器：

java -XX:+PrintCommandLineFlags -version

-Xmx50m -Xms50m -XX:+UseG1GC -Xloggc:d:\\\\gc.log结合如下使用：

**在线工具：https://gceasy.io/，可以上传GC日志生成GC报告,设置最大停顿间隔**

## 5. 常见面试题 {#常见面试题 .list-paragraph}

1.谈谈JVM的内存模型

2.JVM的运行时数据区有哪些组件，作用是什么？

3.JVM的堆内存结构是怎样的？哪些情况会触发GC?会触发哪些GC？

4.GC类型:3种

5.四大垃圾回收算法和垃圾回收器

# 附录：生产时的JVM参数设置

+-----------------------------------------------------------------------+
| \$JAVA_ARGS                                                           |
|                                                                       |
| .=                                                                    |
|                                                                       |
| \"                                                                    |
|                                                                       |
| -server                                                               |
|                                                                       |
| -Xmx3000M                                                             |
|                                                                       |
| -Xms3000M                                                             |
|                                                                       |
| -Xmn600M                                                              |
|                                                                       |
| -XX:PermSize=500M                                                     |
|                                                                       |
| -XX:MaxPermSize=500M                                                  |
|                                                                       |
| -Xss256K                                                              |
|                                                                       |
| -XX:+DisableExplicitGC                                                |
|                                                                       |
| -XX:SurvivorRatio=1                                                   |
|                                                                       |
| -XX:+UseConcMarkSweepGC                                               |
|                                                                       |
| -XX:+UseParNewGC                                                      |
|                                                                       |
| -XX:+CMSParallelRemarkEnabled                                         |
|                                                                       |
| -XX:+UseCMSCompactAtFullCollection                                    |
|                                                                       |
| -XX:CMSFullGCsBeforeCompaction=0                                      |
|                                                                       |
| -XX:+CMSClassUnloadingEnabled                                         |
|                                                                       |
| -XX:LargePageSizeInBytes=128M                                         |
|                                                                       |
| -XX:+UseFastAccessorMethods                                           |
|                                                                       |
| -XX:+UseCMSInitiatingOccupancyOnly                                    |
|                                                                       |
| -XX:CMSInitiatingOccupancyFraction=70                                 |
|                                                                       |
| -XX:SoftRefLRUPolicyMSPerMB=0                                         |
|                                                                       |
| -XX:+PrintClassHistogram                                              |
|                                                                       |
| -XX:+PrintGCDetails                                                   |
|                                                                       |
| -XX:+PrintGCTimeStamps                                                |
|                                                                       |
| -XX:+PrintHeapAtGC                                                    |
|                                                                       |
| -Xloggc:log/gc.log                                                    |
+-----------------------------------------------------------------------+

### [JVM的参数含义及设置](https://www.cnblogs.com/levontor/p/11340466.html)

+--------------+-----------------+------------+-----------------------+
| **参数名称** | **含义**        | **默认值** |                       |
+--------------+-----------------+------------+-----------------------+
| -Xms         | 初始堆大小      | 物         | 默认                  |
|              |                 | 理内存的1  | (MinHeapFreeRatio参数 |
|              |                 | /64(\<1GB) | 可以调整)空余堆内存小 |
|              |                 |            | 于40%时，JVM就会增大  |
|              |                 |            | 堆直到-Xmx的最大限制. |
+--------------+-----------------+------------+-----------------------+
| -Xmx         | 最大堆大小      | 物理内存的 | 默认(Max              |
|              |                 | 1/4(\<1GB) | HeapFreeRatio参数可以 |
|              |                 |            | 调整)空余堆内存大于7  |
|              |                 |            | 0%时，JVM会减少堆直到 |
|              |                 |            | -Xms的最小限制        |
+--------------+-----------------+------------+-----------------------+
| -Xmn         | 年              |            | **注意**              |
|              | 轻代大小(1.4or  |            | ：此处的大小是（eden+ |
|              | later)          |            | 2 survivor            |
|              |                 |            | space).与jmap         |
|              |                 |            | -heap中显示的New      |
|              |                 |            | gen是不同的。\        |
|              |                 |            | 整                    |
|              |                 |            | 个堆大小=年轻代大小 + |
|              |                 |            | 年老代大小 +          |
|              |                 |            | 持久代大小.\          |
|              |                 |            | 增大年轻              |
|              |                 |            | 代后,将会减小年老代大 |
|              |                 |            | 小.此值对系统性能影响 |
|              |                 |            | 较大,Sun官方推荐配置  |
|              |                 |            | 为整个堆的3/8(默认值) |
+--------------+-----------------+------------+-----------------------+
| -XX:NewSize  | 设置            |            |                       |
|              | 年轻代大小(for  |            |                       |
|              | 1.3/1.4)        |            |                       |
+--------------+-----------------+------------+-----------------------+
| -X           | 年              |            |                       |
| X:MaxNewSize | 轻代最大值(for  |            |                       |
|              | 1.3/1.4)        |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX:PermSize | 设置持久代(perm | 物理       |                       |
|              | gen)初始值      | 内存的1/64 |                       |
+--------------+-----------------+------------+-----------------------+
| -XX          | 设              | 物理       |                       |
| :MaxPermSize | 置持久代最大值  | 内存的1/4  |                       |
+--------------+-----------------+------------+-----------------------+
| -Xss         | 每个            |            | JDK5.                 |
|              | 线程的堆栈大小, |            | 0以后每个线程堆栈大小 |
|              |                 |            | 为1M,以前每个线程堆栈 |
|              | 该              |            | 大小为256K.更具应用的 |
|              | 值会影响栈的深  |            | 线程所需内存大小进行  |
|              | 度、也会影响生  |            | 调整.**在相           |
|              | 成多少个线程。  |            | 同物理内存下,减小这个 |
|              |                 |            | 值能生成更多的线程**  |
|              |                 |            | .但是操作系统对一个进 |
|              |                 |            | 程内的线程数还是有限  |
|              |                 |            | 制的,不能无限生成,经  |
|              |                 |            | 验值在3000\~5000左右\ |
|              |                 |            | 一般小的应用，        |
|              |                 |            | 如果栈不是很深，      |
|              |                 |            | 应该是128k够用的      |
|              |                 |            | 大的应用建议          |
|              |                 |            | 使用256k。这个选项对  |
|              |                 |            | 性能影响比较大，需要  |
|              |                 |            | 严格的测试。（校长）\ |
|              |                 |            | 和threadstacksi       |
|              |                 |            | ze选项解释很类似,官方 |
|              |                 |            | 文档似乎没有解释,在论 |
|              |                 |            | 坛中有这样一句话:\""\ |
|              |                 |            | -Xss is translated in |
|              |                 |            | a VM flag named       |
|              |                 |            | ThreadStackSize"\     |
|              |                 |            | 一般                  |
|              |                 |            | 设置这个值就可以了。  |
+--------------+-----------------+------------+-----------------------+
| -XX:Thr      | Thread Stack    |            | (0 means use default  |
| eadStackSize | Size            |            | stack size) \[Sparc:  |
|              |                 |            | 512; Solaris x86: 320 |
|              |                 |            | (was 256 prior in 5.0 |
|              |                 |            | and earlier); Sparc   |
|              |                 |            | 64 bit: 1024; Linux   |
|              |                 |            | amd64: 1024 (was 0 in |
|              |                 |            | 5.0 and earlier); all |
|              |                 |            | others 0.\]           |
+--------------+-----------------+------------+-----------------------+
| -XX:NewRatio | 年轻代(包括Ede  |            | -XX:New               |
|              | n和两个Survivor |            | Ratio=4表示年轻代与年 |
|              | 区)与年老代的比 |            | 老代所占比值为1:4,年  |
|              | 值(除去持久代)  |            | 轻代占整个堆栈的1/5\  |
|              |                 |            | Xms=Xmx并且           |
|              |                 |            | 设置了Xmn的情况下，该 |
|              |                 |            | 参数不需要进行设置。  |
+--------------+-----------------+------------+-----------------------+
| -XX:S        | Eden区与Survi   |            | 设置为8,则两个Sur     |
| urvivorRatio | vor区的大小比值 |            | vivor区与一个Eden区的 |
|              |                 |            | 比值为2:8,一个Survivo |
|              |                 |            | r区占整个年轻代的1/10 |
+--------------+-----------------+------------+-----------------------+
| -XX:LargePag | 内存页的大小    |            | =128m                 |
| eSizeInBytes | 不可设置过大，  |            |                       |
|              | 会              |            |                       |
|              | 影响Perm的大小  |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX          | 原始            |            |                       |
| :+UseFastAcc | 类型的快速优化  |            |                       |
| essorMethods |                 |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX:+Disab   | 关闭System.gc() |            | 这                    |
| leExplicitGC |                 |            | 个参数需要严格的测试  |
+--------------+-----------------+------------+-----------------------+
| -XX:MaxTenur | 垃圾最大年龄    |            | 如果设置为0的话,则    |
| ingThreshold |                 |            | 年轻代对象不经过Survi |
|              |                 |            | vor区,直接进入年老代. |
|              |                 |            | 对于年老              |
|              |                 |            | 代比较多的应用,可以提 |
|              |                 |            | 高效率.如果将此值设置 |
|              |                 |            | 为一个较大值,则年轻代 |
|              |                 |            | 对象会在Survivor区进  |
|              |                 |            | 行多次复制,这样可以增 |
|              |                 |            | 加对象再年轻代的存活  |
|              |                 |            | 时间,增加在           |
|              |                 |            | 年轻代即被回收的概率\ |
|              |                 |            | 该参数                |
|              |                 |            | 只有在串行GC时才有效. |
+--------------+-----------------+------------+-----------------------+
| -XX:+Ag      | 加快编译        |            |                       |
| gressiveOpts |                 |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX:+UseB    | 锁              |            |                       |
| iasedLocking | 机制的性能改善  |            |                       |
+--------------+-----------------+------------+-----------------------+
| -Xnoclassgc  | 禁用垃圾回收    |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX          | 每兆堆空闲      | 1s         | softly reachable      |
| :SoftRefLRUP | 空间中SoftRefe  |            | objects will remain   |
| olicyMSPerMB | rence的存活时间 |            | alive for some amount |
|              |                 |            | of time after the     |
|              |                 |            | last time they were   |
|              |                 |            | referenced. The       |
|              |                 |            | default value is one  |
|              |                 |            | second of lifetime    |
|              |                 |            | per free megabyte in  |
|              |                 |            | the heap              |
+--------------+-----------------+------------+-----------------------+
| -X           | 对              | 0          | 单位字节              |
| X:PretenureS | 象超过多大是直  |            | 新生代采用Parallel    |
| izeThreshold | 接在旧生代分配  |            | Scavenge GC时无效\    |
|              |                 |            | 另一                  |
|              |                 |            | 种直接在旧生代分配的  |
|              |                 |            | 情况是大的数组对象,且 |
|              |                 |            | 数组中无外部引用对象. |
+--------------+-----------------+------------+-----------------------+
| -X           | TLAB占          | 1%         |                       |
| X:TLABWasteT | eden区的百分比  |            |                       |
| argetPercent |                 |            |                       |
+--------------+-----------------+------------+-----------------------+
| -XX:+Coll    | Fu              | false      |                       |
| ectGen0First | llGC时是否先YGC |            |                       |
+--------------+-----------------+------------+-----------------------+

 **并行收集器相关参数**

  ---------------------------- -------------------------------------------- ------ -------------------------------------------------------------------------------------------------------------------------------------------------------
  -XX:+UseParallelGC           Full GC采用parallel MSC\                            选择垃圾收集器为并行收集器.此配置仅对年轻代有效.即上述配置下,年轻代使用并发收集,而年老代仍旧使用串行收集.(此项待验证)
                               (此项待验证)                                        

  -XX:+UseParNewGC             设置年轻代为并行收集                                可与CMS收集同时使用\
                                                                                   JDK5.0以上,JVM会根据系统配置自行设置,所以无需再设置此值

  -XX:ParallelGCThreads        并行收集器的线程数                                  此值最好配置与处理器数目相等 同样适用于CMS

  -XX:+UseParallelOldGC        年老代垃圾收集方式为并行收集(Parallel               这个是JAVA 6出现的参数选项
                               Compacting)                                         

  -XX:MaxGCPauseMillis         设置GC的最大暂停时间,谨慎设置                       如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.

  -XX:+UseAdaptiveSizePolicy   自动选择年轻代区大小和相应的Survivor区比例          设置此选项后,并行收集器会自动选择年轻代区大小和相应的Survivor区比例,以达到目标系统规定的最低相应时间或者收集频率等,此值建议使用并行收集器时,一直打开.

  -XX:GCTimeRatio              设置垃圾回收时间占程序运行时间的百分比              公式为1/(1+n)

  -XX:+ScavengeBeforeFullGC    Full GC前调用YGC                             true   Do young generation GC prior to a full GC. (Introduced in 1.4.1.)
  ---------------------------- -------------------------------------------- ------ -------------------------------------------------------------------------------------------------------------------------------------------------------

**辅助信息**

+------------------------------+-------------+-----+-------------------+
| -XX:+PrintGC                 |             |     | 输出形式:         |
|                              |             |     |                   |
|                              |             |     | \[GC              |
|                              |             |     | 118250K-\>        |
|                              |             |     | 113543K(130112K), |
|                              |             |     | 0.0094143 secs\]\ |
|                              |             |     | \[Full GC         |
|                              |             |     | 121376K-\         |
|                              |             |     | >10414K(130112K), |
|                              |             |     | 0.0650971 secs\]  |
+------------------------------+-------------+-----+-------------------+
| -XX:+PrintGCDetails          |             |     | 输出形式:\[GC     |
|                              |             |     | \[DefNew:         |
|                              |             |     | 861               |
|                              |             |     | 4K-\>781K(9088K), |
|                              |             |     | 0.0123035 secs\]  |
|                              |             |     | 118250K-\>        |
|                              |             |     | 113543K(130112K), |
|                              |             |     | 0.0124633 secs\]\ |
|                              |             |     | \[GC \[DefNew:    |
|                              |             |     | 8614              |
|                              |             |     | K-\>8614K(9088K), |
|                              |             |     | 0.0000665         |
|                              |             |     | secs\]\[Tenured:  |
|                              |             |     | 112761K-\         |
|                              |             |     | >10414K(121024K), |
|                              |             |     | 0.0433488 secs\]  |
|                              |             |     | 121376K-\         |
|                              |             |     | >10414K(130112K), |
|                              |             |     | 0.0436268 secs\]  |
+------------------------------+-------------+-----+-------------------+
| -XX:+PrintGCTimeStamps       |             |     |                   |
+------------------------------+-------------+-----+-------------------+
| -X                           |             |     | 可与-XX:+PrintGC  |
| X:+PrintGC:PrintGCTimeStamps |             |     | -XX:+PrintG       |
|                              |             |     | CDetails混合使用\ |
|                              |             |     | 输出形式:11.851:  |
|                              |             |     | \[GC              |
|                              |             |     | 98328K-\          |
|                              |             |     | >93620K(130112K), |
|                              |             |     | 0.0082960 secs\]  |
+------------------------------+-------------+-----+-------------------+
| -XX:+P                       | 打印垃      |     | 输出形式:Total    |
| rintGCApplicationStoppedTime | 圾回收期间  |     | time for which    |
|                              | 程序暂停的  |     | application       |
|                              | 时间.可与上 |     | threads were      |
|                              | 面混合使用  |     | stopped:          |
|                              |             |     | 0.0468229 seconds |
+------------------------------+-------------+-----+-------------------+
| -XX:+Prin                    | 打印        |     | 输出              |
| tGCApplicationConcurrentTime | 每次垃圾回  |     | 形式:Application  |
|                              | 收前,程序未 |     | time: 0.5291524   |
|                              | 中断的执行  |     | seconds           |
|                              | 时间.可与上 |     |                   |
|                              | 面混合使用  |     |                   |
+------------------------------+-------------+-----+-------------------+
| -XX:+PrintHeapAtGC           | 打印        |     |                   |
|                              | GC前后的详  |     |                   |
|                              | 细堆栈信息  |     |                   |
+------------------------------+-------------+-----+-------------------+
| -Xloggc:filename             | 把相        |     |                   |
|                              | 关日志信息  |     |                   |
|                              | 记录到文件  |     |                   |
|                              | 以便分析.\  |     |                   |
|                              | 与上面几    |     |                   |
|                              | 个配合使用  |     |                   |
+------------------------------+-------------+-----+-------------------+
| -XX:+PrintClassHistogram     | garbage     |     |                   |
|                              | collects    |     |                   |
|                              | before      |     |                   |
|                              | printing    |     |                   |
|                              | the         |     |                   |
|                              | histogram.  |     |                   |
+------------------------------+-------------+-----+-------------------+
| -XX:+PrintTLAB               | 查          |     |                   |
|                              | 看TLAB空间  |     |                   |
|                              | 的使用情况  |     |                   |
+------------------------------+-------------+-----+-------------------+
| X                            | 查          |     | Desired survivor  |
| X:+PrintTenuringDistribution | 看每次minor |     | size 1048576      |
|                              | G           |     | bytes, new        |
|                              | C后新的存活 |     | threshold 7 (max  |
|                              | 周期的阈值  |     | 15)\              |
|                              |             |     | new threshold     |
|                              |             |     | 7即标识新的存     |
|                              |             |     | 活周期的阈值为7。 |
+------------------------------+-------------+-----+-------------------+
