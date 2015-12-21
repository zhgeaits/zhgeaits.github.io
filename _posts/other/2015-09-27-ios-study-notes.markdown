---
layout: post
title:  "IOS学习笔记"
date:   2015-09-27 11:00:03
categories: other
type: other
---

## 关于CocoaPods

cocoapods和maven一样，是用来管理依赖的，它是ruby编写的，所以还是有必要学习一下ruby的。

## 关于Object-C

程序入口其实是main.mm，.mm的意思是兼容c，c++和oc混编。这个文件里面就一个c的main函数，被@autoreleasepool包围着，意思是交给了arc来进行内存管理，然后这个函数调用了UIApplicationMain方法，可以看到这里初始化了AppDelegate，也创建了UIApplication对象实例。更多的东西以后慢慢学习。

### AppDelegate

它就像android里面的application，或者说activity，参考这边blog比较他们的生命周期：http://seniorzhai.github.io/2014/12/11/Android%E3%80%81iOS%E5%A4%A7%E4%B8%8D%E5%90%8C%E2%80%94%E2%80%94%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/

实际上AppDelegate是UIApplication的一个委托，即UIApplicationDelegate协议的实现，同时，它也是继承了UIResponder的。

比较重要的一个方法是：

`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`

它就像android里面的onCreate一样，一般我们都在这里编写主要的初始化代码，其他的方法，以后再来总结。

### class

.h是头文件，即公共api，.m是实现文件，包含了公共api的实现和私有api。在.m里面用@interface Foo() @end来定义私有api。

#### 属性

在头文件里面，定义它的实例变量还需要｛｝，即在@interface后面用大括号包围变量，实例变量的规范是_开头。但是调用这些变量的时候必须通过编写setter和getter方法。规范是setter方法是setXXX，而getter方法是直接的属性名去掉_，因为通过点符号是这样规范使用的。

@property定义的属性，会自动生成getter和setter方法的了，属性设置strong，nonatomic，getter，setter等。strong是强引用；nonatomic是非原子性，即不是线程安全；getter和setter是用来修改方法名字的。例如：

@property (strong, nonatomic, getter=isChosen) BOOL chosen;

通过点的方式就能调用setter和getter方法了。点方法只用于setter和getter方法，其他方法都不会用。

#### 方法

在头文件里面定义方法，以减号-开始，紧跟着是括号，里面方法的返回类型。oc里面的方法名不再是一个字符串了，而是多个字符串组成，一个字符串对应一个方法参数。如下：

`- (NSString *)generateXXXNames:(NSString *)test withPrefix:(NSString *)aaa;`

如果是以+号开始，则表示是类方法，就是java的静态方法。

### 协议与委托

协议可以理解为接口，而委托就是协议的实现。因为每一个类都会有头文件，这个头文件不能完全理解为java那边的接口，因为协议也有重合的意思了。

定义协议一般在头文件里面定义，只要是@protocol XXXName<YYYName> @end包围的即可，后面尖括号表示XXXName协议包含所有YYYName协议，相当于继承。之后一般这个头文件都会有一个这个协议的属性，即有一个委托。这就好像java里面一个类里面定义了一个listener接口，然后这个类会有这个listener的引用属性。

一个类实现了某个协议叫做委托，头文件@interface后面用尖括号包围这个协议即可。

### ViewController

生命周期：

init—>loadView—>viewDidLoad—>viewWillApper—>viewDidApper—>viewWillDisapper—>viewDidDisapper—>viewWillUnload->viewDidUnload—>dealloc

## xcode

## 其他

1.	organization identifier相当于包名
2.	新建一个OC文件的时候，选择的是cocoa touch class是什么意思？
3.	New group相当于new一个package吗？
4.	如果直接new一个oc文件的时候，可用选择的那三种filetype是什么意思
5.	button没法设置背景颜色
6.	ViewController是怎么和xib建立联系的？为什么File’s Owner有东西？好像是custom class那里建立联系的
7.	怎么修改屏幕尺寸？
8.	感觉Delegate就是android那边的application，UIViewController就是Activity之类的。
9.	为什么初始化的是调用了super init后返回值直接赋值给self？所以构造方法返回的是instanceType，而不是某个类的指针地址，虽然调用的是父类的init方法，实际上是子类的对象实例。
10.	子类也能调用父类的静态方法？java好像不行？
11.	Cocoa Touch框架感觉就像是jdk一样，提供了OC类的集合。
12.	OC里面使用指针指向对象，而不是引用或者句柄。
13.	OC的alloc方法是类方法，和java的类初始化方法一样<clinit>，注意，不是实例的构造方法哦，oc的init才是对应构造方法。
14.	OC里面调用方法称为发送消息：接收方，选择器，实参。接收方就是类或者对象，选择器则是方法，实参就是参数了。发送一个消息的形式是中括号。这种方式比较符合面向对象，因为对象之间是需要通信的，而不是方法的调用。
15.	OC的所有对象都是继承NSObject，里面的description方法相当于java的toString方法。
16.	NSString,NSArray,NSMutableArray,NSLog都是以NS开头的，NS就是nextstep的意思，你懂得！
17.	只有调用NSLog才和java比较像，还有就是NSString的一些类方法也是比较相似，例如，initWithFormat，因为是不定参数的原因吧。其他的方法调用都是以发送消息形式的。打印的时候，基本上和c相同，就字符串是％@，实际上是调用对象的description方法。
18.	OC相对于C新增的关键字基本上都是以@开头的，但是也有一些不是，例如：instancetype。还有，一个字符串是用@开头的。
22.	id类型相当于c里面的void＊，指向任意对象的指针。
23.	为什么在重载构造方法的时候可用调用self，这个时候self不是nil吗？
25.	@的神奇作用。字符串，关键字，NSArray初始值，import
26.	调用一个对象的时候，也就是发送消息的时候，在编译期间不能知道这个对象有没有这个方法，只有在运行期才会发生错误。
27.	属性使用copy的时候，当给它赋值的时候就会copy一份再赋值。
28.	synthesized不懂怎么用。
29.	这个调用的是Core Graphics的上下文[[UIColor lightGrayColor] setStroke];还是需要好好学习了解。
30.	图片资源文件添加在哪里？
31.	UITabBarController不能预览？
32.	方法前面的括号除了是返回类型，还是表示这个方法是什么，例如IBAction
33.	[[UIApplication sharedApplication] scheduledLocalNotifications:note];编译不通过。
34.	controller传递参数的时候，@Class到底什么作用，还是没明白。
36. viewDidLoad