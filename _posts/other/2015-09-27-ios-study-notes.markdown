---
layout: post
title:  "IOS学习笔记"
date:   2015-09-27 11:00:03
categories: other
type: other
---

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
19.	@autoreleasepool是什么意思？
20.	@interface关键字相当于java的class，它还要对应@end。在头文件里面，定义它的实例变量还需要｛｝，实例变量的规范是_开头，在.m文件里面，m的意思是implements的意思。定义方法的时候，参数类型用括号包起来。
21.	如果不使用get和set方法，直接使用点语法，它跟java的直接点一个变量不同，它点了以后需要去掉_。
22.	id类型相当于c里面的void＊，指向任意对象的指针。
23.	为什么在重载构造方法的时候可用调用self，这个时候self不是nil吗？
24.	定义方法的时候-表示的是实例方法，+表示的是类方法。
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
