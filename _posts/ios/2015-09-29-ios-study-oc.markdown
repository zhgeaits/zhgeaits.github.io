---
layout: post
title:  "iOS学习笔记之Objective-C"
date:   2015-09-29 11:00:03
categories: ios
supertype: career
type: ios
---

学习iOS，入门核心便是学习它的语言，只要学会了它的语言，基本上在这个平台的开发就已经掌握了基础。而语言无非就是由语法组成，学会了语法也就学会了表达，实际上，学习一门语言就是学习它的表达式即可，当然，里面会有很多语言的整体设计思想，例如，设计到内存，就会有地址，即C语言的指针，由于Objective-C是面向对象的，这也就包括面向对象的思想，重要的还有内存模型，内存管理，垃圾回收，等等。既然我是学习C语言起步的，学习过C++，Pascal，Basic等，又是非常的熟悉Java语言，那么掌握OC就是相对容易的事情了。

## 1.编译

既然OC是从改过来的，实际上和C就是同一个家族的了，编译环境就很类似的了，编译C和C++都可以使用GCC，但是编译OC就只能用clang了。而其实也有过IDE的，那就是xcode，这个工具也是非常复杂的，特别是开发界面的时候。在mac上应该自带了clang编译器的，如果没有，只要安装一下xcode就有了。

oc也是有头文件的，但是实现文件是以.m结尾的了，这个m的意思是module的意思。clang的用法和gcc很相似的，不同的是要引入oc的framework Foundation，例如下面编译HelloWorld：

>% clang -o helloworld HelloWorld.m -framework Foundation

实际上，这个framework就像是oc的SDK，和JDK一样，提供了很多的类和函数调用，都封装在一起，叫Cocoa环境，iOS里面的是Cocoa Touch环境。只要引入一个头文件即可：

>#import <Foundation/NSObject.h>

另外，如果一个源码文件里面混合了c，c++和oc的话，那么文件后缀就是.mm了。clang的语法和gcc是非常的相似，慢慢了解即可，用了IDE以后也很少接触clang的了。

## 2.对象和消息

虽然说Java是非常纯的面向对象语言，但是，里面对象之间的交互通信依然是叫做方法调用，其实就是函数调用，每个对象的动作函数叫做方法，要触发对象执行动作就需要调用该对象的方法，这一点并没有让人感觉到面向对象的不同。而在OC里面就彻底改变了这个概念，不再使用调用函数的概念了，而是发送消息，如果一个对象要触发另外一个对象的动作，那么就需要给该对象发送一个消息，消息内容包括了方法名字和参数，方法名就是消息名，这个概念下就不会存在空指针异常了，因为允许没对象接收消息，就像广播一样。这是一个非常好的概念，一切对象的通信都是通过消息，而且在语法上也发生了改变，采用中括号[]的表达式，而消息名是由多个字符串组成，一个字符串对应一个参数，不再像函数一样了，一个字符串然后括号里面多个参数。例如：

[obj msg1:param1 withSecondParam:param2 withThirdParam:param3]

上面就是给obj发送了一个消息，消息名/方法名就是msg1:withSecondParam:withThirdParam，然后是传了三个参数的。当然如果十几个参数的话，那么这个名字就有点长了，不过，这个概念脱离了数学上的函数，因为毕竟函数还是很抽象的一个概念，而是更加接近了面向对象。

中括号还是会和数组有混淆的，代码读起来也不会那么顺畅，挺乱的，不过熟悉以后可读性还是不错的，因为比较容易理解这个消息的意义。

java里面可以链式的调用方法，就是不挺的点下去，发送消息也是可以不断的嵌套发送，不过，嵌套太多的中扩展，可读性就会非常低了，完全不如点的链式。

消息名还有一个名字，叫做选择器；要注意的是，如果选择器同名，区别在于有没参数，那么这个选择器的名字区别就在于有没有冒号了。

## 3.类与实例

在java里面，较好的规范是一个文件一个类，而类的方法有范围的修饰符，如果是public的修饰，那么就是公共的api了，对别的类来说的可见的。而在c语言的规范里面，存在头文件，通常在头文件里面声明了各个函数，然后在实现文件里面实现函数，这样别的地方通过include这个头文件就可以使用这些函数接口了。oc也是一样的，存在头文件，因此，规范的做法是每一个类都需要编写一个头文件，在头文件里面编写类的接口声明。这一点上，java是合并在一个文件里面，实际上是不好的做法，混淆了接口的概念，还增加了另外一个“接口interface”的概念，在oc里面使用的是“协议”这个概念。

oc的类的声明语法如下：

	@interface 类名:父类名 {
		实例变量的定义；
		...
	}
	消息声明;
	...
	@end

可以看到以@开头的指令，实际上这就是oc区别于c的编译指令。实例变量的声明是在花括号里面的，如果是实例的声明定义也采用了@指令的形式，即@property，那么就不需要在花括号里面了，而通常我们开发iOS都是这样做的。为什么这样，因为是property熟悉隐含的帮我们声明了getter和setter的方法。

类的实现单独编写在.m文件，语法如下：

	@implementation 类名
	方法实现
	...
	@end

在java里面编写好类以后，通过new关键字就可以实例化类来使用了，只需要理解存在一个跟类名一样的构造方法就可以了，实际上里面隐含了很多虚拟机的阶段，从分配内存到真正实例化一个类分了好几个阶段的，虽然便于了开发，但是也区别普通和高级程序员。而在oc里面就不一样了，必须要理解两个步骤，一个是分配内存，另外一个是初始化。所以实例化一个类需要调用两个方法，如下：

>[[类名 alloc] init]

这两个消息都是默认的名字，所有的类都有的，我们可以重载或者重写一个init方法，但是必须调用原来的init方法，而且init方法是有返回值的，不像java里面的构造方法没有返回值。

java里面所有的类都是Object的子类，根据多态的思想，所有的引用都可以是Object的引用。但是oc里面分了三个不同的情况，首先，所有的类都是NSObject的子类，其次，指向任意类的指针并不是NSObject，而是id，这个id就像c里面的void*一样，最后，init消息返回的并不是当前类的指针，而是instancetype。所有创建实例返回的方法都是intancetype。

由于init这些初始化方法并不是像java里面的特殊的构造方法，它是像普通方法一样的方法，所以不能存在方法名相同，而返回类型不同，这点和java是一样的，所以init方法就不能返回当前类的指针了，否则会导致父类和子类的init方法冲突，java不存在这个问题是因为使用构造方法，oc的解决方案就是init都是返回同一个叫instancetype的类型，那么就实现了方法的重载。因此在调用子类的init方法里面必须调用父类的init方法。

在java里面本类的实现的引用是this，而oc里面是叫self，父类的引用，都是叫super。

### 3.1 属性

普通实例变量的定义：

>int _number;

由于oc里面的对象通信只能通过消息的形式，所以oc里面没有像java那样点的方式来调用，即使是属性也是，这样面向对象的概念封装得更好，所有的属性变量都是私有的，访问这些属性变量的时候必须通过编写setter和getter方法。如果没有编写属性的setter和getter方法，只能在本类里面直接通过属性名来访问。

可以看到实例变量的规范是_开头，setter方法是setXXX，而getter方法是直接的属性名去掉_，不是getXXX了。为什么？因为这是一个坑，开发人员发现对于属性来说，不通过点来调用的话开发是很不方便的，于是还是让它支持了点的调用语法，但是实际上触发的就是getter和setter方法，其他方法还是必须发消息调用。这会让我们不熟悉的人踩坑，总以为点属性的语法只是简单的取值和设置值，没想到的是调用了两个方法，里面可能还有更多的逻辑。于是乎，通过点调用语法的时候就成了self.number,而不是self._number了，oc自动帮我们优化了，按照这个规则编写即可。

每个属性都必须这样写两个方法，如果没有编写setter和getter方法的话，即使在本类的实现里面也不能通过self点属性名来访问该属性。

非常的繁琐，于是oc又发明了@property的指令，自动帮我们生成setter和getter方法，而且支持更多的配置，非常的强大。

@property的变量声明：

>@property (strong, nonatomic, getter=isChosen) BOOL chosen;

@property定义的属性，属性设置strong，nonatomic，getter，setter等。strong是强引用；nonatomic是非原子性，即不是线程安全；getter和setter是用来修改方法名字的。另外除了strong，还有assign，weak，copy，readonly等配置。

**私有属性**

在头文件里面定义的都是公开的属性，因为头文件就是api嘛，若想定义本类使用的私有属性，就需要在.m的实现文件里面编写了。在@implementation上面定义，语法如下：

	@interface 类名 () {
		普通属性;
		...
	}
	@property属性;
	...
	@end
	@implementation
	...
	@end

可以看到的区别是类名后面多了一个括号。

### 3.2 方法/函数/消息/选择器

在头文件里面定义方法，以减号-开始，紧跟着是括号，里面方法的返回类型。例如：

`- (NSString *)generateXXXNames:(NSString *)test withPrefix:(NSString *)aaa;`

方法就是一条消息，前面明白了消息的发送，这里的声明也非常容易理解，特别的是以减号-开始，然后是括号，里面是返回值的类型。那么如果是以加号+开始的是什么？那是java里面的类方法，也就是静态方法。如果声明的是静态方法，那么发送消息的时候，消息的接收者就是类了，而不是实例了。另外，声明私有方法的时候和私有属性一样，在同样的地方声明即可。

还要注意的是，没有返回值的消息是void，返回任意指针的是id，而非NSObject*，如果返回本实例引用的消息是instancetype。

一切的对象通信都是通过消息，即使属性的点语法也是发送了setter和getter消息，因此就弱化了函数调用的概念，即使在本类里面，一个方法调用另外一个方法，也只能通过对self进行发送消息的形式来调用。除非是混合编译的情况，就是oc的代码里面混合c的代码，比如说，引入了stdio.h头文件的时候，进行一些系统调用，那么就可以使用函数调用的语法了，源码的文件后缀也是.mm了。

### 3.3 引入

c里面如果引入其他的模块，使用的是#include 头文件的形式，java里面是直接import，而oc里面也是使用了#import，虽然兼容c，但是区别c的include在于不会重复引入头文件，除非我们混合c编程，否则都不会使用到include指令的。虽然c里面使用#ifndef和#define这些指令解决重复引入问题，不过就比较麻烦，还容易出错。

## 4.数据类型

和java一样，oc可以使用基本类型，如int，long，float，double，char，byte，short，布尔类型是BOOL。还有一堆对象类型，以NS开头的，NSNumber是封装的对象类型，类似于java那边的Integer，Double这些。NSString是字符串类型。当使用对象类型的时候只能使用指针了。和java一样，基本类型通常不用指针，对象类型使用指针，就是java的引用。

当给对象类型初始化值的时候，使用@的指令，如：

	NSNumber *number = @123;
	NSString *name = @"zhangge";
	NSArray *array = @[@"helloworld", @27];

NSArray，NSMutableArray，NSSet，NSMutableSet，NSDictionary，NSMutableDictionary是常用的数据结构，Mutable的意思是可变的，即数据结构的长度是可以动态改变的。这些数据结构里面没有泛型，所以可以混合放任意类型的对象指针。

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

### UIWindow

一个应用启动的时候有一个主的UIWindow，但是我们也可以自己创建很多UIWindow来进行显示，创建以后，调用makeKeyAndVisible就可以了。

如果屏幕进行了旋转，则需要对UIWindow进行设置transform属性，代码如下：

> float angle = ([UIDevice currentDevice].orientation == UIDeviceOrientationLandscapeLeft)?90:-90;  
> 
> view.transform = CGAffineTransformMakeRotation(degreesToRadian(angle));

另外，还需要对frame进行调整。

## Category

这个是oc针对区别继承的一种特性，继承是在拥有父类的功能的前提下，还可以进行重载，修改父类的功能。而category是对类的一种功能扩展，不可以重载，只能增加新的功能。

它的特点的是不需要访问原生类的代码，也就是说不需要知道父类，本质实现是？

## Protocol

协议

## 约束constraints

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

		<td>command+y</td>

		<td>断点生效或无效</td>

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

	<tr>

		<td>command+option+[或者]</td>

		<td>上移或者下移一行代码，跟java那边的一样，非常方便</td>

	</tr>

	<tr>

		<td>command+option+左键点击</td>

		<td>垂直打开文本编辑</td>

	</tr>

</table>

## 其他

translatesAutoresizingMaskIntoConstraints applyAutoResizingMaskWithOldSuperviewSize

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

35.如果用了property属性，会自动生成一个实例？为毛要这样干？oc不是没有空指针异常了么？这样还浪费内存了

36.马蛋，oc里面自动生成set方法有点恶心，虽然方便了，但是通过点来赋值的时候，会产生误导。很多人经常在set方法里面来做很多事情，然而通过点等号来赋值并不是仅仅的赋值，而是去调用了那个set方法。。。

1. viewDidLoad



objective-c 计算md5和sha1


{% highlight objective-c %}

+ (NSString *)fileMD5:(NSString *)filePath {
    NSFileHandle *handle = [NSFileHandle fileHandleForReadingAtPath:filePath];
  
    if(!handle)
  
    {
  
    return nil;
  
    }
  
    CC_MD5_CTX md5;
  
    CC_MD5_Init(&md5);
  
    BOOL done = NO;
  
    while (!done)
  
    {
    NSData *fileData = [handle readDataOfLength:256];
    CC_MD5_Update(&md5, [fileData bytes], [@([fileData length]) unsignedIntValue]);
    if([fileData length] == 0)
        done = YES;

  
    }
  
    unsigned char digest[CC_MD5_DIGEST_LENGTH];
  
    CC_MD5_Final(digest, &md5);
  
    NSString *result = [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
  
                    digest[0], digest[1],
                    digest[2], digest[3],
                    digest[4], digest[5],
                    digest[6], digest[7],
                    digest[8], digest[9],
                    digest[10], digest[11],
                    digest[12], digest[13],
                    digest[14], digest[15]];
  
    return result;
  
  }
  
+ (NSString *)fileSha1:(NSString *)filePath
  
  {
  
    NSFileHandle *handle = [NSFileHandle fileHandleForReadingAtPath:filePath];
  
    if(!handle)
  
    {
  
    return nil;
  
  
    }
  
    CC_SHA1_CTX sha1;
  
    CC_SHA1_Init(&sha1);
  
    BOOL done = NO;
  
    while (!done)
  
    {
  
    NSData *fileData = [handle readDataOfLength:256];
    CC_SHA1_Update(&sha1, [fileData bytes], [@([fileData length]) unsignedIntValue]);
    if([fileData length] == 0)
        done = YES;
  
    }
  
    unsigned char digest[CC_SHA1_DIGEST_LENGTH];
  
    CC_SHA1_Final(digest, &sha1);
  
    NSString *result = [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
  
                    digest[0], digest[1],
                    digest[2], digest[3],
                    digest[4], digest[5],
                    digest[6], digest[7],
                    digest[8], digest[9],
                    digest[10], digest[11],
                    digest[12], digest[13],
                    digest[14], digest[15],
                    digest[16], digest[17],
                    digest[18], digest[19]
                    ];
  
    return result;
  
  }

{% endhighlight %}
