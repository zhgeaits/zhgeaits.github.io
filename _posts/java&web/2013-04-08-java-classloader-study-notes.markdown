---
layout: post
title:  "Java系列之class字节码，内存模型，classloader类加载及反射机制"
date:   2013-04-08 13:00:03
categories: java
supertype: career
type: java&web
---

## 1 什么是java的类Class

刚学习C语言的时候，我才开始了解到编译器把C代码编译成obj二进制文件，然后进行链接，最后成为机器代码，这样计算机才能识别进行执行。而当我学习java的时候，确实把java代码编译成了class文件，那时候只知道这是被java虚拟机专属执行的字节码文件。因为java的目标是夸平台性的，所以它希望一次编译成功以后的class文件，不管在什么平台上面，只要装有java虚拟机都能够执行；而其实java虚拟机更是跨语言性的，不管是什么语言，只要被相应的编译器编译成class文件，都能够被执行，而目前这些语言包括了：java，groovy，jruby，scala等等。因此，class文件不过是被定义的一种文件格式罢了，因为它是字节码文件，它的每一个字节都被定义规范利用起来，然后虚拟机便能够识别来执行了。而这些定义都可以在The Java Language Specification和The Java Virtual Machine Specification上面查询。只要我们认真研究了这些规范以后，我们也就明白了其实可以进行反编译，那些反编译的工具也是这样来的，根据规范解析class文件罢了。

### 1.1 Class类文件结构

定义class类的结构用了两种数据类型：无符号数和表。无符号数只有u1,u2,u4,u8，分表描述一个结构用了1，2，4，8个字节。表则是由多个无符号数或者其他表作为数据项的符合类型（递归的定义方式）。本质上class文件就是一个表，它的项有无符号类型的，也有表类型的，就像代码里的类定义一样。

class文件结构大致包括了如下项：magic，version，constant\_pool，access_flags，this，super，interface，fields，methods，attributes。

我们可以使用jdk提供的工具来分析class文件：javap -verbose TestClass

下面分表描述各项：

魔法数就像文件扩展名一样告诉我们它的格式，确定这个文件是否为一个能被虚拟机接受的class文件，它的值固定为0xCAFEBABE，就是咖啡宝贝，哈哈，难怪java的图标是咖啡。

版本定义的就是编译的主版本和次版本，用来被虚拟机确定能否执行，或者兼容。

常量池这里存放了所有的常量，它是从1开始按顺序给每个常量编号，当别的项或者其他地方需要用到常量的时候，就指定一个索引指向即可。如果指向了常量池的0索引，则表示的是“不引用任何一个常量池项目”。常量池的项目类型包含多种，每一个类型都是表。某一项的表的值又可以指向了常量池的某一项。总之，常量池比较复杂，存放了类名，属性，方法名，方法的代码等等。

access_flags是16位的二进制表示类的访问限制。

this，super，interface的值分别指向不同的常量池的项，告诉我们本类名，父类名和实现的接口是什么。

每一个字段都需要一个表来存放，信息包括：访问标记，名字索引，描述符索引和属性。访问标记和类的差不多，名字索引是指向了常量池。描述符其实就是这个类型的简写，描述符索引也是指向了常量池，包括：B(byte)，C(char)，D(double)，F(float)，I(int)，J(long)，S(short)，Z(boolean)，V(void)，L(对象类型，如Ljava/lang/Object)，对于数组类型，则用一个[来表示，多一维就多一个[。属性则是指向了最后的属性集合。如果字段被final修饰，则这个就是常量，会出现在常量池和属性表（为什么出现两次？）。测试发现只用final修饰或者final+static修饰结构是一样的，但是只用static就不一样了。

存放方法的表结构和属性的表一样，但是方法的访问标记不一样，方法的描述符更复杂，先描述参数值，再描述返回值，例如：String a(int[] b)的描述符为([I)Ljava/lang/String。最后属性项存放的就是指向属性集合的索引。实际存放的就是方法体内的指令代码。

属性表存放了比较多的内容，包括了21项，方法指令，final修饰的常量，过时的方法和字段，异常表，行号，签名，源文件等等。属性表和常量池的除了有一个常量相同以外，其他都不同了。

### 1.2 什么是常量，static和final的区别

常量的意思是它初始化时的初值就是固定的了，不能再被修改了，而且只会初始化一次，就是说不管new一个类多少次，它只存在一次，那么常量就需要static和final来同时修饰了。

只用static来修饰，只能说明它是存放在方法区的变量，虽然不会随new的次数而增加，但是它是可以被修改的，static修饰的变量初始值都是零值，只有在类初始化（不是实例初始化）的时候才会被赋值。

final的意思是不能被修改了，如果只用final来修饰的话，按道理来说，这个是实例变量，不是类变量，不会存放在方法区的，而是存在堆的，但是测试发现final的变量存放在了class文件的常量池里面，也存放在字段的属性ConstantValue里面，我猜测在类加载的时候不会存放在方法区的，所以class文件的常量池和方法区的常量池有一些区别。

同时使用static和final正好符合这个需求，这个变量同时存放在class文件的常量池里面，又存放在了字段属性的ConstantValue里面。

## 2 类加载机制

上面描述了java代码编译成为class文件的结构，它只是一个字节流，只有存放到内存以后里面的常量和代码指令才会起作用。从上面的class结构可以知道java其实是运行期的动态加载和动态连接的语言，例如我们根据接口编程，然后实际运行的时候才去加载真正的实现类。这是因为class文件里面存的是符号引用而不是直接引用。类加载的意思其实就是把class文件交给jvm，让jvm做相应的处理，然后存放在java的内存里面，然后就可以对这个类进行使用了。

### 2.1 类的生命周期和加载的时机

类的整个生命周期包括七个阶段：`加载(Loading)`，`验证(Verification)`，`准备(Preparation)`，`解析(Resolution)`，`初始化(Initialization)`，`使用(Using)`和`卸载(Unloading)`。其中验证，准备和解析统称为`连接(linking)`阶段。前5个阶段并不是严格串行的，通常会在一个阶段执行的过程中调用激活另外一个阶段。虚拟机规范并没有指定什么时候必须进行第一步的加载，但是却严格规定了5中情况必须对类进行初始化，而初始化之前肯定先得加载和连接过程了。

对象被GC回收以后会被卸载吗？如果会，什么时候卸载？

### 2.2 加载

加载阶段只需要完成3件事情：  
1.通过一个类的全限定名（包名+类名）来获取定义此类的二进制字节流（class文件）。通常可以从zip包（jar，war，ear），网络，运行时生成，数据库等地方读取，这都是开放实现的。  
2.将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。其实就是把这个class存储在了方法区。  
3.在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据结构的访问入口。Class对象可能存在堆中，也可能存在方法区中，根据不同的虚拟机而定，Hotspot虚拟机就是存在方法区的。这个Class类包含了ClassLoader。

从第三点可以知道每个对象都有一个Class对象，可以通过getClass获取，那么我们其实就知道了instanceof的意思了，objectA instanceof A，其实意思就是objectA.getClass()==A.class或者objectA.getClass().equals(A.class)的意思。

PS：在android里面this.getClass()可能会报错，说不知道this是哪个Object的。在IDEA上面有问题报错，所以，这样写一下就OK：
  
>((Object) this).getClass();

### 2.3 验证

加载阶段还没完成，其实验证阶段就开始了，它需要对class文件进行校验是否合法与安全。因为只要你了解了class文件的结构以后，你其实就可以自定义生成一些恶意的class文件让虚拟机执行。验证的工作太多了，大概包括：文件格式验证（魔数，版本，常量是否支持，文件是否完整等等），元数据验证（继承的父类，实现的接口是否合法等等），字节码验证（验证各种指令）和符号引用验证（在解析阶段才会触发，验证能否找到相应的引用的类和接口，字段和方法等等）。

### 2.4 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。就是说会把被static修饰的变量分配在方法区，class的常量池包含了final和final+static修饰的变量，不清楚这个阶段是否把部分常量池分配内存，但是方法除了常量区也分了静态区的。注意并不是实例变量，实例变量是存在堆中的，也不是这个阶段的事情。

### 2.5 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。符号引用只是一个可以描述目标的任何形式的字面量，例如包名+类名来指定一个类。直接引用可以是直接指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。这个阶段需要解析接口，类，字段，方法等。例如解析一个类的时候而找不到就会抛出ClassNotFoundException，还有NoSuchMethodException等等。

### 2.6 初始化

这个初始化是类的初始化<clinit>，而不是实例的初始化<init>，就是不会去执行类的构造方法。<clinit>可以理解为initialize()方法，与之对应的是finalize()，但是Object类只有finalize()，找不到initialize()，因为这个初始化是在类加载的时候完成的。一般类有static属性或者static块的时候才会自动生成<clinit>方法，static是按编写顺序执行初始化的，也必须先执行父类的<clinit>才能执行子类的。

虚拟机规范中规定了有且仅有一下五种情况才会去触发一个类的初始化：  
1.遇到new,getstatic,putstatic,invokestatic指令的时候。即new一个类，读取或设置一个类的static字段，调用一个类的static方法时候。  
2.使用java.lang.reflect包进行对一个类反射的时候。  
3.初始化一个类发现父类没初始化。  
4.虚拟机启动时候包含mian方法的类。  
5.使用jdk1.7的动态语言支持的时候。  

### 2.7 其他

虽然java可以重载方法，就是方法名可以一样，但是每个重载方法都必须有一个独一无二的参数列表，包括顺序不一样，但是不能返回参数不一样，因为调用的时候不一定用到返回参数，所以不确定。为什么在方法内可以调用this对象，实际上，在调用一个对象的方法时，编译器会把修改方法的参数，加入该对象的自身引用进去。

构造器是隐式的静态，只有编译器才能调用，但是在构造器里面可以this调用前面位置的构造器，并且只能调用一个，所以其他地方不能调用构造器。

## 3 类加载器

完成第一阶段加载工作的代码模块就是类加载器（ClassLoader）。同一个类加载器只能加载相同的一个类，不同的类加载器却可以加载相同的类，但是这两个类的实例对象却不是相同的类来的，使用instanceof的时候会返回false。

类加载器分为三种：  
1.启动类加载器(Bootstrap ClassLoader)：它是C++语言实现的，负责加载<JAVA_HOME>/lib目录下的所有jar包。它是顶层的根类加载器。其他的加载器都是用java实现的，而且是实现抽象类ClassLoader。 
 
2.扩展类加载器(Extension ClassLoader)：它负责加载<JAVA_HOME>/lib/ext目录，或者java.ext.dirs环境变量指定的目录的类。  

3.应用程序加载器(Application ClassLoader)：调用ClassLoader.getSystemClassLoader()就可以获取得到这个加载器，它负责将ClassPath上指定的类库加载。如果没有自定义的类加载器，我们的应用默认就是使用这个了。

上面的3种加载器是继承关系的，所以加载的过程应该是先调用父类来加载，如果加载失败再自己进行加载。

在2.6说了如果new一个类的话就会触发初始化，然后会把类加载进内存；如果我们只是想要加载一个类，有两种方法：

>ClassLoader.getSystemClassLoader().loadClass("org.zhangge.testclass.TestClass");

当然可以不使用getSystemClassLoader，而使用随便一个对象.getClass().getClassLoader()就可以了，这样就使用了同一个ClassLoader了。这里也可以知道其实每一个Class类都有相应的ClassLoader的引用。

第二种方法：

>Class.forName("org.zhangge.testclass.TestClass");

很熟悉，这就是我们学校JDBC的时候用的方法。也就是说JDBC那里把驱动加载进去内存并完成了类的初始化。

这个方法还有一个多参数的重载方法：

>public static Class<?> forName(String name, boolean initialize, ClassLoader loader) throws ClassNotFoundException;

这里的initialize参数是很重要的，即被加载同时是否完成初始化的工作，单参数版本的forName方法默认是完成初始化的：

>Class.forName(className, true, currentLoader)。

### 3.1 ClassLoader类

ClassLoader有下面一些重要的方法：

1.loadCass方法：以类的全限定名加载类。  
2.defineClass方法：这个方法把类class文件的字节数组转换成Class对象。字节数组可以是从本地文件系统或网络装入的数据。它把字节码分析成运行时数据结构、校验有效性等等。当我们自定义ClassLoader的时候需要调用这个方法。  
3.findSystemClass方法：从本地文件系统装入Java字节码。它在本地文件系统中寻找类文件，如果存在，就使用defineClass将字节数组转换成Class对象。当运行Java应用程序时,这是JVM正常装入类的缺省机制。  
4.resolveClass方法：解析装入的类，如果该类已经被解析过那么将不做处理。当调用loadClass方法时，通过它的resolve参数决定是否要进行解析。  
5.findLoadedClass方法：当调用loadClass方法装入类时，调用findLoadedClass方法来查看ClassLoader是否已装入这个类，如果已装入，那么返回Class对象，否则返回NULL。如果强行装载已存在的类，将会抛出链接错误。

什么时候需要自定义ClassLoader？

例如我们要加密class文件不让别人反编译的时候，那么我们就需要自定义ClassLoader了，先按照自己的加密方式解密得到原来的class文件，再loader进去。

### 3.2 Class类与反射

从上面我们了解到了每个对象都有Class对象，也就是说，如果java的Class对象很强大的话，我们就可以在运行时通过Class对象做很多事情了，事实就是这样的，java还有专门的reflect包，专门是搞反射的。让我们可以通过class对象任意获取对象的属性和调用对象的方法，尽管是private的。

一些学习测试的代码在：JavaTestBox/org.zhangge.reflection

一些常用的反射：

1.获取对象的属性值：

{% highlight java %}

Class ownerClass = owner.getClass();
Field field = ownerClass.getField(fieldName);
Object property = field.get(owner);

{% endhighlight %}

2.获取类的静态属性值：

{% highlight java %}

Class ownerClass = owner.getClass();
Field field = ownerClass.getField(fieldName);
Object property = field.get(ownerClass);//区别在这里的参数

{% endhighlight %}

3.执行对象的方法：

{% highlight java %}

Method method = ownerClass.getMethod(methodName, argsClass);//argsClass是参数的类型数组
method.invoke(owner, args);//args是参数的Object数组

//下面是执行静态方法，第一个参数传null
//method.invoke(null, args);//args是参数的Object数组

{% endhighlight %}

4.新建实例：

{% highlight java %}

Class newoneClass = Class.forName(className);

//如果调用没有参数的构造方法：
newoneClass.newInstance();

//如果调用有参数的构造方法：
Constructor cons = newoneClass.getConstructor(argsClass);//argsClass是参数的类型数组
cons.newInstance(args);

{% endhighlight %}

当然反射还能做更多的事情，这里就不列举了。以后用到再补充。

## 4.Java的内存结构

Java内存可以大概理解为运行前java虚拟机向操作系统申请初始化内存，然后运行中java应用程序在jvm管理的逻辑内存下运行，如果jvm内存不够就继续向操作系统申请，jvm可以设置参数调优，有初始大小和最大大小。如果超出的话就会发生溢出。

具体来说，一个类在JVM的内存这样存放：类的结构信息，常量，静态常量，静态方法,class类和classloader的引用存放在方法区，然后类的属性信息，包括变量属性，引用属性等存放在堆，对于局部变量存放在栈，而且对已这个对象的引用也是存放在栈的。方法的调用，不管传值还是传引用值，都是压入栈的。

### 4.1 Java内存运行时数据区域

Java内存模型简单分为方法区，堆和栈，但是其实还可以划分得更细。一般来说记住简单的就可以了。详细的运行时数据区域包括：`程序计数器`，`Java虚拟机栈`，`本地方法栈`，`Java堆`，`方法区`，`本机直接内存`。

程序计数器：每个线程都有一个PC用于记录当前执行指令（VM的原语），可以看作是当前线程所执行的字节码的行号指示器。该区域没有规定OutOfMemoryError情况。

JVM栈：生命周期与线程相同，是线程私有的。这里存放8种标准类型，引用值，方法返回地址等。如果线程请求的栈深度大于虚拟机所允许的深度，就抛出StackOverflowError异常；如果JVM栈可以动态扩展，但是扩展无法申请足够内存就抛出OutOfMemoryError异常。

本地方法栈：作用与JVM栈一样，不过的是为native方法服务。一般两者和在一起讨论。

Java堆Heap：这是线程共享的，栈是每个线程都有的。按照回收机制，堆可以分为两个区,新生代和老年代。对象（包括数组）是存放在这里的。GC在这里回收。这里还可能划分出多个线程私有的分配缓冲区TLAB。

方法区：存放类的结构信息，常量池，字段描述，方法描述，静态变量，静态方法，class类的引用，classloader类的引用，这两个引用用于反射机制。

运行时常量池：它属于方法区的一部分。静态常量和非静态常量都存放在这里，静态常量的初始化只有在类加载的时候进行一次。在运行时调用intern()方法可以把变量加入常量池。

本机直接内存：不再JVM管理范围，但是存放一些资源，IO缓冲等如果没有手动释放会产生内存泄漏。
  


还有疑惑：
1.虚拟机是什么？进程？常驻的服务？唯一？还是每跑一个java代码就启动一个虚拟机？