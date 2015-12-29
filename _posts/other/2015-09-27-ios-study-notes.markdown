---
layout: post
title:  "IOS学习笔记"
date:   2015-09-27 11:00:03
categories: other
type: other
---

## 0. 关于CocoaPods

cocoapods和maven一样，是用来管理依赖的，它是ruby编写的，所以还是有必要学习一下ruby的。

## 1. 关于Object-C

程序入口其实是main.mm，.mm的意思是兼容c，c++和oc混编。这个文件里面就一个c的main函数，被@autoreleasepool包围着，意思是交给了arc来进行内存管理，然后这个函数调用了UIApplicationMain方法，可以看到这里初始化了AppDelegate，也创建了UIApplication对象实例。更多的东西以后慢慢学习。

### 1.1 AppDelegate

它就像android里面的application，或者说activity，参考这边blog比较他们的[生命周期](http://seniorzhai.github.io/2014/12/11/Android%E3%80%81iOS%E5%A4%A7%E4%B8%8D%E5%90%8C%E2%80%94%E2%80%94%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/ "生命周期")


实际上AppDelegate是UIApplication的一个委托，即UIApplicationDelegate协议的实现，同时，它也是继承了UIResponder的。

比较重要的一个方法是：

`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`

它就像android里面的onCreate一样，一般我们都在这里编写主要的初始化代码，其他的方法，以后再来总结。

### 1.2 类class

.h是头文件，即公共api，.m是实现文件，包含了公共api的实现和私有api。在.m里面用@interface Foo() @end来定义私有api。

#### 1.2.1 属性

在头文件里面，定义它的实例变量还需要｛｝，即在@interface后面用大括号包围变量，实例变量的规范是_开头。但是调用这些变量的时候必须通过编写setter和getter方法。规范是setter方法是setXXX，而getter方法是直接的属性名去掉_，因为通过点符号是这样规范使用的。

@property定义的属性，会自动生成getter和setter方法的了，属性设置strong，nonatomic，getter，setter等。strong是强引用；nonatomic是非原子性，即不是线程安全；getter和setter是用来修改方法名字的。例如：

@property (strong, nonatomic, getter=isChosen) BOOL chosen;

通过点的方式就能调用setter和getter方法了。点方法只用于setter和getter方法，其他方法都不会用。

#### 1.2.2 方法

在头文件里面定义方法，以减号-开始，紧跟着是括号，里面方法的返回类型。oc里面的方法名不再是一个字符串了，而是多个字符串组成，一个字符串对应一个方法参数。如下：

`- (NSString *)generateXXXNames:(NSString *)test withPrefix:(NSString *)aaa;`

如果是以+号开始，则表示是类方法，就是java的静态方法。

#### 1.2.3 数据类型

NSNumber是封装的对象类型，类似于java那边的Integer，Double这些。

NSString是字符串类型，用@来标记，而其实NSNumber也是可以使用@的。

NSArray，NSMutableArray，NSSet，NSMutableSet，NSDictionary，NSMutableDictionary是常用的数据结构，Mutable的意思是可变的，即数据结构的长度是可以动态改变的。

常量数组一般是@加中括号，如：@[a,b,c];

NSNull是nil的对象类型，不要在数据结构使用nil，应该使用NSNull，因为在c里面，nil是代表结尾。

NSUserDefaults就像android里面的Preference一样。

### 1.3 协议与委托

协议可以理解为接口，而委托就是协议的实现。因为每一个类都会有头文件，这个头文件不能完全理解为java那边的接口，因为协议也有重合的意思了。

定义协议一般在头文件里面定义，只要是@protocol XXXName<YYYName> @end包围的即可，后面尖括号表示XXXName协议包含所有YYYName协议，相当于继承。之后一般这个头文件都会有一个这个协议的属性，即有一个委托。这就好像java里面一个类里面定义了一个listener接口，然后这个类会有这个listener的引用属性。

一个类实现了某个协议叫做委托，头文件@interface后面用尖括号包围这个协议即可。

### 1.4 控制器UIViewController

生命周期：

init—>loadView—>viewDidLoad—>viewWillApper—>viewDidApper—>viewWillDisapper—>viewDidDisapper—>viewWillUnload->viewDidUnload—>dealloc

在stroyboard里面对controller的simulated metrics的bottom bar设置为none，不然挡住UI。

数据传递：

在同一个NavigationController里面，传递数据是直接访问controller的属性来赋值的。不同NavigController的时候，是通过segue来获取controller，然后还是访问属性来读取赋值的。

页面跳转：

### 1.5 UIView

awakeFromNib:

当一个nib文件对应两个类，File's Owner的class为XXXViewController，Objects下的View对应的为XXXView时：

awakeFromNib：在XXXView.m文件中有效，即只有写在这个类文件中才会调用，写在XXXViewController.m文件中时，不会被调用。
viewDidLoad：写于XXXViewController.m文件中，作用同awakeFromNib。
 
当.nib文件被加载的时候，会发送一个awakeFromNib的消息到.nib文件中的每个对象，每个对象都可以定义自己的 awakeFromNib函数来响应这个消息，执行一些必要的操作。也就是说通过nib文件创建view对象是执行awakeFromNib 

viewDidLoad 
当view对象被加载到内存是就会执行viewDidLoad，所以不管通过nib文件还是代码的方式创建对象都会执行viewDidLoad 

## Category

这个是oc针对区别继承的一种特性，继承是在拥有父类的功能的前提下，还可以进行重载，修改父类的功能。而category是对类的一种功能扩展，不可以重载，只能增加新的功能。

它的特点的是不需要访问原生类的代码，也就是说不需要知道父类，本质实现是？

## Protocol

协议

## xib界面

一个xib是一个界面，其实就是xml，所以是xml interface builder的意思。选择File’s Owner，在右边检查器面板的custom class可以设置ViewController，在controller的loadView方法的时候就会加载一个xib。xib文件编译以后就是nib文件了。

## storyboard界面

一个stroyboard包括了多个界面，即多个viewcontroller，如果没有设置custom controller，应该是有默认的。

视图控制器选定后，选取“Editor”>“Embed In”>“Navigation Controller”，可以装配到导航控制器。

在storyboard里面，如果需要为点击某个按钮跳转到下一个界面，只要按住control键，拖动那个按钮指向下一个界面，然后在弹出来的窗口选择push即可。如果需要点击某个按钮退出当前界面回到上一个界面，如果都是在NavigationController里面，会自动有返回键的，但是如果是从一个NavigationController返回到另一个NavigationController的话，需要要按住control键，拖动那个按钮指向Exit项即可，需要在上一个界面定义IBAction操作，如下：

- (IBAction)unwindBack:(UIStoryboardSegue *)segue;

## 关于NSBundle

## xcode

在Deployment Info里面的Main Interface可以设置选择主要的storyboard界面。选择一个storyboard以后，在右边的custom class可以设置ViewController。还需要在检查器面板那里设置这个ViewController为初始化控制器，即勾上Is initial View Controller。

### 快捷键

<table>
	<tr>
		<td>command+0,1,2..</td>
		<td>工程导航器的显示和隐藏，并对应上面的每一个tab</td>
	</tr>
	<tr>
		<td>command+option+0,1,2..</td>
		<td>工具面板（检查器面板）的显示和隐藏，并对应上面的每一个tab</td>
	</tr>
	<tr>
		<td>option+左键点击</td>
		<td>打开Assistant Editor</td>
	</tr>
	<tr>
		<td>Command+Shift+F</td>
		<td>全局搜索</td>
	</tr>
	<tr>
		<td>Command+F</td>
		<td>本页面搜索</td>
	</tr>
	<tr>
		<td>Control+6（键入方法/变量名+Enter跳转）</td>
		<td>相当于idea上面ctrl+F12快速定位到某个方法</td>
	</tr>
	<tr>
		<td>Command + Shift + O</td>
		<td>快速打开</td>
	</tr>
	<tr>
		<td>command+control+上下键</td>
		<td>打开Assistant Editor</td>
	</tr>
	<tr>
		<td>Command + Shift + 0 (Zero)</td>
		<td>文档和参考</td>
	</tr>
	<tr>
		<td>在类或者方法名上执行Option + 左键</td>
		<td>内联帮助</td>
	</tr>
	<tr>
		<td>command+shift+j</td>
		<td>快速定位到打开的文件</td>
	</tr>
	<tr>
		<td>control+1</td>
		<td>打开Show Related Items弹出菜单，非常方便查看很多东西，如哪里调用这个方法了</td>
	</tr>
	<tr>
		<td>shift+command+y</td>
		<td>hide or show debug area</td>
	</tr>
	<tr>
		<td>ctrl+command+y</td>
		<td>debug的时候play</td>
	</tr>
	<tr>
		<td>F6</td>
		<td>debug的时候step over</td>
	</tr>	
	<tr>
		<td>F7</td>
		<td>debug的时候step into</td>
	</tr>
	<tr>
		<td>option+command+左右箭头</td>
		<td>把括号里面的代码收缩起来</td>
	</tr>
	<tr>
		<td>command+[或者]</td>
		<td>相当于tab或者shift+tab的效果，缩进</td>
	</tr>

</table>

## 其他

1.	organization identifier相当于包名
2.	新建一个OC文件的时候，选择的是cocoa touch class是什么意思？
3.	New group相当于new一个package吗？
4.	如果直接new一个oc文件的时候，可用选择的那三种filetype是什么意思
5.	button没法设置背景颜色
7.	怎么修改屏幕尺寸？
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