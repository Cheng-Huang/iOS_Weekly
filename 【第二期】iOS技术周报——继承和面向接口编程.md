# 【第二期】iOS技术周报——继承和面向接口编程

在写项目的时候，我发现项目中有继承导致的不合理的耦合问题，想找到相关的解决方案并尝试改进。在网上看了几篇相关博客和文章，了解了如何在保证代码复用性的同时，去除代码的耦合。本期技术周报我将整合继承和面向接口编程的相关资料进行分享。

在开篇之前，引用[继承和面向接口（iOS架构思想篇）](https://www.jianshu.com/p/39e6a8409476)的几个问题来引出本期的内容。

> 1. 继承最大的缺点是什么？
> 2. 为什么说耦合也可能是一种需求？
> 3. 有哪些场景不适合使用继承？
> 4. 继承本身就具有高耦合性，但却可以实现代码复用，有哪些替代方案可以去除高耦合性并实现代码的复用
> 5. iOS 开发中有否有必要同一派生 ViewController？
> 6. 什么是面向切面编程思想？
> 7. 为什么Swift着力宣传面向协议的思想，而OC 中面向协议的思想为什么不能像Swift那样得以普及？

# 一、继承

[**继承**](https://zh.wikipedia.org/zh-hans/继承_%28计算机科学%29)（英语：inheritance）是面向对象软件技术当中的一个概念。 如果一个类別B「**继承**自」另一个类別A，就把这个B称为「A的子类」，而把A称为「B的父类別」也可以称「A是B的超类」。 **继承**可以使得子类具有父类別的各种属性和方法，而不需要再次编写相同的代码。

## 为什么使用继承

1. 代码复用
2. 制定规范（规则）
3. 为了多态

## 继承的局限性

> 继承从代码复用的角度来说，特别好用，也特别容易被滥用和被错用。不恰当地使用继承导致的最大的一个缺陷特征就是**高耦合**。  
> 在这里我要补充一点，耦合是一个**特征**，虽然大部分情况是缺陷的特征，但是**当耦合成为需求的时候，耦合就不是缺陷**了。

以上引用了Casa提出的观点，他在文章内用具体的例子讲述了继承的局限性，具体可移步阅读[跳出面向对象思想\(一\) 继承](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-yi-ji-cheng.html)。

## 继承的观点

本节我引用了[iOS架构师之路：慎用继承](https://www.jianshu.com/p/ee8c6fcc1746)对继承适用场景的叙述。

### 适用继承的场景

Chris Eidhof的文章[《subclassing》](https://link.jianshu.com?t=https://www.objc.io/issues/13-architecture/subclassing/)提到需要自定义UITableViewCell等View视图的时候我们可以使用继承来创建自定义View,这些代码放入子类更合理，不光代码得到更好的封装，也能得到一个可以在工程中重用的组件。Chris Edihof还提到model可以继承来实现了 `isEqual:`、`hash`、 `copyWithZone:` 和 `description` 等方法的类。  
 Chris Eidhof说的只是继承的几个应用场景，他没有说使用继承的界限。

Casa在[跳出面向对象思想\(一\) 继承](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-yi-ji-cheng.html)中提到是否使用继承需要考虑三个点：

1. 父类只是给子类提供服务，并不涉及子类的业务逻辑
2. 层级关系明显，功能划分清晰，父类和子类各做各的。
3. 父类的所有变化，都需要在子类中体现，也就是说此时耦合已经成为需求
    　　在我看来一个很重要的原则就是我们不能脱离cocoa框架开发，所以我们可以继承cocoa的类，以达到快速开发的目的，但是如果没有特殊原因我们写的代码要控制在继承链不增加两层。

### 不适用继承的场景

Casa的观点：当你发现你的继承超过2层的时候，你就要好好考虑是否这个继承的方案了，第三层继承正是滥用的开端。确定有必要之后，再进行更多层次的继承。

Chris Eidhof也有类似的观点：`In a lot of projects that I’ve worked on, I’ve seen deep hierarchies of subclasses. I am guilty of doing this as well. Unless the hierarchies are very shallow, you very quickly tend to hit limits.`（ 在我工作的许多项目中看到过一些深度继承的项目。当我也这么干的时候，总会感到内疚。除非继承的层次非常浅，否则你会很快发现它的局限性。）

## 替代继承的方式

### 协议

假设原本已经开发了一个继承 NSObject 的音频播放器 VoicePlayer ，但此时想支持 OGG 格式的音频。而实际上之前的 VoicePlayer 和现在想要开发的音频播放器类，只是对外提供的API类似，内部实现代码却差别很大。这里简单说明一下 OGG 格式音频在游戏开发中用的比较普遍，相比于其他音频而言，OGG 最大的特点是体积更小。一段音频中，没有声音的那一部分将不暂用任何体积，而类似 MP3 格式则不同，即使是没声音，依然会存在体积占用。参照上面关于继承的使用原则可知，此时继承并不适合这种场景。解决方案通过协议提供相同的接口，代码结构如下：

```objective-c
@protocol VoicePlayerProtocol <NSObject>
- (void)play;
- (void)pause;
@end

@class NormalVoicePlayer : NSObject <VideoPlayerProtocol>
@end

@class OGGVoicePlayer : NSObject <VideoPlayerProtocol>
@end
```

### 组合或聚合

以下对继承、组合和聚合的定义参考了[面向对象的三大特性: 封装， 继承， 多态](http://www.cnblogs.com/bossren/p/6428475.html)

#### 类与类之间常见的三种关系：继承、组合、聚合

1. 如果两个类之间拥有`is a`关系,这两个类应该是**继承**关系。

   狗是动物  Dog is a Animal.

   Animal是**父类**， Dog是**子类**

2. 如果两个类之间拥有`has a`关系，应该用**组合**或**聚合**

   计算机中有一个CPU  Computer has a CPU

   **组合**和**聚合**是另一种类与类之间的关系

#### 组合

表示两个对象之间是整体和部分的强关系，是“contains\(包含\) a”关系，要求两个类同生共死。生命周期完全一致，同生共死。部分的生命周期不能超越整体！

例如：一个窗口内有按钮、标签，当窗口关闭时，窗口与按钮、标签同时销毁。

##### 优点：

1）当前对象只能通过所包含的那个对象去调用其方法，所以所包含的对象的内部细节对当前对象是不可见的。

2）当前对象与包含的对象是一个低耦合关系，如果修改包含对象的类中代码，不需要修改当前对象类的代码。

3）当前对象可以在运行时动态的绑定所包含的对象。可以通过set方法给所包含对象赋值。

##### 缺点：

容易产生过多的对象

为了能组合多个对象，必须仔细对接口进行定义。

#### 聚合

表示两个对象之间是整体和部分的弱关系，是”has a”关系，不要求两个类同生共死。生命周期不一致，一个类无法控制另一个类的生死。部分的生命周期可以超越整体。

例如：电脑和鼠标，电脑被销毁时，鼠标可以留下在其他电脑上继续使用。

##### 优点：

1）被包含的对象通过包含它们的类来访问

2）很好的封装

3）被包含的对象内部细节不可见

4）可以在运行时动态定义聚合的方式

##### 缺点：

1）系统可能会包含太多对象

2）当使用不同的对象时，必须小心定义的接口。

##### 组合与聚合的区别：

组合和聚合的生命周期不一样，组合是同生共死（关系紧密）；聚合没有特别的关系。

##### 适用组合或聚合的情况：

假如：A界面有个输入框，会根据服务器上用户的输入历史来自动补全，叫`AutoCompleteTextField`。后来某天来了个需求，在另外一个界面中，也用到这个输入框，除了根据输入历史补全，增加一个自动补全邮箱的功能，就是用户输入@后，我们自动补全一些域名。这个功能很简单，结构如下：

```objective-c
@interface AutoCompleteTextField : UITextField
- (void)autoCompleteWithUserInfo;
@end

@interface AutoCompleteMailTextField : AutoCompleteTextField
- (void)autoCompleteWithMail;
@end
```

过两天，产品经理希望有个本地输入框能够根据本地用户信息来补全，而不是根据服务器的信息来自动补全，我们可以轻松通过覆盖来实现：

```objective-c
@interface AutoCompleteLocalTextField : AutoCompleteTextField
- (void) autoCompleteWithUserInfo;
@end
```

app上线一段时间之后，UED不知哪根筋搭错了，决定要修改搜索框的UI,于是添加个初始化函数`initWithStyle`得以解决。

重点来了，但是有一天，隔壁项目组的哥们想把我们的本地补全输入框`AutoCompleteLocalTextField`移植到他们的项目中。这个可就麻烦了，因为使用`AutoCompleteLocalTextField`要引入`AutoCompleteTextField`，而`AutoCompleteTextField`本身也带着API相关的对象，同时还有数据解析的对象。 也就是说，要想给另外一个TEAM，差不多整个网络层框架都要移植过去。

上面这个问题总结来说是两种类型问题：第一种类型问题是改了一处，其他都要改，但是勉强还能接受；第二种类型就是代码复用的时候，要把所有相关依赖都拷贝过去才能解决问题；两种类型的问题都说明了继承的高耦合性，牵一而动全身的特性。

关于上述问题最佳的解决方案，我认为是通过组合的形式，区分不同的模块来处理，输入框本身的UI可以作为一个模块，本地搜索提示和服务器搜索提示可以作为不同的模块分别处理。实际使用中可以通过不同的模块组合，实现不同的功能。

#### 总结

如果想重用已有的代码而不想共享同样的接口，组合或聚合便是首选。

### 类别

有时可能会想在一个对象的基础上增加额外的功能形成另外一个对象，继承是一种很容易想到的方法。还有另外一种比较好的方案是通过类别。为该对象扩展方法，按需调用，比如为NSArray增加一个移除第一个元素的方法：

```objective-c
@interface NSArray (OBJExtras)
- (void)removingFirstObject;
@end
```

另外，类似无网络或数据的提示视图，也可以借助Category实现，在无法避免使用属性的情况下，可以借助运行时添加属性。可以完全同控制器解耦。

### 配置对象

假设某个app中有主题切换，其中每种主题都对应`backgroundColor` 和 `font` 两个属性。按照继承的思路我们很有可能会先写一个父类，为这个父类实现一个空的setupStyle方法，然后各种不同风格的主题分别是一个子类，重写父类的`setupStyle`方法。

其实大可不必这样做，完全可以创建一个`ThemeConfiguration`的类，该类中具有 `backgroundColor`和 `fontSize` 属性。可以事先创建几种主题， Theme 在其初始化函数中获取一个配置类 ThemeConfiguration 的值即可。相比继承而言，就不用创建那么多文件，以及父类中还要写一个 `setupStyle`空方法。

# 二、ViewController是否应统一继承

## 不统一继承的理由

如果ViewController统一继承了父类控制器，首先可能会涉及到上面说到的高耦合的一个项目，缺点；除此之外，还会涉及上手接受成本问题，新手接受需要对父类控制器的使用有一定的了解；另外，如果涉及项目迁移问题，在迁移子类控制器的同时还要将父类控制器也迁移出去。最后一个理由是，即使不通过继承，同样能达到对项目控制器进行统一配置。

## 面向切面\(AOP\)思想简介

上面也说了几种替代继承的方法，如果ViewController不通过继承的方式实现，那么首选的替代方式是什么？这里我们可以采用面向切面的编程思想和分类结合的方式替代控制器的继承。  
 首先简单说下面向切面的编程思想（AOP），听起来很高大上，实际上很多iOS开发者应该都用过，在iOS中最直接的体现就是借助 Method Swizzling 实现方法替换。一般，主要的功能是日志记录，性能统计，安全控制，事务处理，异常处理等等。主要的意图是：将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改 变这些行为的时候不影响业务逻辑的代码。可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。**假设把应用程序想成一个立体结构的话，OOP的利刃是纵向切入系统，把系统划分为很多个模块（如：用户模块，文章模块等等），而AOP的利刃是横向切入系统，提取各个模块可能都要重复操作的部分（如：权限检查，日志记录等等）。**

## 方案实现

面向切面的思想可以实现系统资源的统一配置，iOS 中的`Method Swizzling`替换系统方法可达到同样的效果。这里更为推荐使用第三方开源库[Aspects](https://github.com/steipete/Aspects)去拦截系统方法。

我们可以创建一个叫做ViewControllerConfigure的类，实现如下代码。

```objective-c
//.h文件
@interface ViewControllerConfigure : NSObject
@end

//.m文件
#import "ViewControllerConfigure.h"
#import <Aspects/Aspects.h>
#import <UIKit/UIKit.h>
@implementation ViewControllerConfigure
+ (void)load
{
    [super load];
    [ViewControllerConfigure sharedInstance];
}

+ (instancetype)sharedInstance
{
    static dispatch_once_t onceToken;
    static ViewControllerConfigure *sharedInstance;
    dispatch_once(&onceToken, ^{
        sharedInstance = [[ViewControllerConfigure alloc] init];
    });
    return sharedInstance;
}

- (instancetype)init
{
    self = [super init];
    if (self) {
        /* 在这里做好方法拦截 */
        [UIViewController aspect_hookSelector:@selector(loadView) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo>aspectInfo){
            [self loadView:[aspectInfo instance]];
        } error:NULL];

        [UIViewController aspect_hookSelector:@selector(viewWillAppear:) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> aspectInfo, BOOL animated){
            [self viewWillAppear:animated viewController:[aspectInfo instance]];
        } error:NULL];
    }
    return self;
}

/*
   下面的这些方法中就可以做到自动拦截了。
    所以在你原来的架构中，大部分封装UIViewController的基类或者其他的什么基类，都可以使用这种方法让这些基类消失。
 */
#pragma mark - fake methods
- (void)loadView:(UIViewController *)viewController
{
    NSLog(@"loadView");
}

- (void)viewWillAppear:(BOOL)animated viewController:(UIViewController *)viewController
{
    /* 你可以使用这个方法进行打日志，初始化基础业务相关的内容 */
    NSLog(@"viewWillAppear");
}
@end
```

关于上面的代码主要说三点：

> 1、借助 load 方法，实现代码无任何入性型。  
>  当类被引用进项目的时候就会执行`load`函数\(在main函数开始执行之前）,与这个类是否被用到无关,每个类的`load`函数只会自动调用一次。除了这个案列，在实际开发中笔者曾这么用过load方法，将app启动后的广告逻辑相关代码全部放到一个类中的load方法，实现广告模块对项目的无入侵性。`initialize`在类或者其子类的第一个方法被调用前调用。即使类文件被引用进项目,但是没有使用,initialize不会被调用。由于是系统自动调用，也不需要再调用 \[super initialize\] ，否则父类的initialize会被多次执行。  
>  2、不单单可以替换`loadView`和`viewWillAppear`方法，还可以替换控制器其他生命周期相关方法，在这些方法中实现对控制器的统一配置。如view背景颜色、统计事件等。  
>  3、控制器中避免不了还会拓展一些方法，如无网络数据提示图相关方法，此时可以借助`Category`实现，在无法避免使用属性的情况下，可以借助运行时添加属性。

关于控制器的集成问题就先说到这，接下来看看面向接口的思想。

# 三、面向接口的思想

对于接口这一概念的支持，不同语言的实现形式不同。Java中，由于不支持多重继承，因此提供了一个Interface关键词。而在C++中，通常是通过定义抽象基类的方式来实现接口定义的。Objective-C既不支持多重继承，也没有使用Interface关键词作为接口的实现（Interface作为类的声明来使用），而是通过抽象基类和协议（protocol）来共同实现接口的。OC中接口可以理解为Protocol，面向接口编程可以理解为面向协议编程。先看如下两端代码：

```objective-c
ASIHTTPRequest *request = [ASIHTTPRequest requestWithURL:url];
[request setDidFinishSelector:@selector(requestDone:)];
[request setDidFailSelector:@selector(requestWrong:)];
[request startAsynchronous];
AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
[manager GET:@"www.olinone.com" parameters:nil success:^(AFHTTPRequestOperation *operation, id responseObject) {

} failure:^(AFHTTPRequestOperation *operation, NSError *error) {

}];
```

观察上述两段代码，是否发现第二段网络请求代码相比第一段更容易使用。因为第二段代码只需初始化对象，然后调用方法传参即可，而第一段代码要先初始化，然后设置一堆属性，最终才能发起网络请求。如果让一个新手上手，毫无疑问更喜欢采用第二种方式调用方法，因为无需对AFN掌握太多，仅记住这一个方法便可发起网络请求，而反观 ASI 要先了解并设置各种属性参数，最终才能发起网络请求。上面的两端代码并不是为了说明ASI和AFN熟好熟劣，只是想借此引出面向接口的思想。

所以，通过接口的定义，调用者可以忽略对象的属性，聚焦于其提供的接口和功能上。程序猿在首次接触陌生的某个对象时，接口往往比属性更加直观明了，抽象接口往往比定义属性更能描述想做的事情。

相比于OC，Swift 可以做到协议方法的具体实现，而 OC 则不行。面向对象编程和面向协议编程最明显的区别在于程序设计过程中对数据类型的抽取（抽象）上，面向对象编程使用类和继承的手段，数据类型是引用类型；而面向协议编程使用的是遵守协议的手段，数据类型是值类型（Swift中的结构体或枚举）。看一个简单的swift版面向协议范例，加入想为若干个继承自UIView的控件扩展一个抖动动画方法，可以按照如下代码实现：

```swift
//  Shakeable.swift
import UIKit
protocol Shakeable { }
extension Shakeable where Self: UIView {
    func shake() {
        // implementation code
    }
}
```

如果想实现这个shake动画，相关控件只要遵守这个协议就可以了。

```swift
class CustomImageView: UIImageView, Shakeable {

}
class CustomButton: UIButton, Shakeable {

}
```

可能有的人就会问了，直接通过 `extension`实现不就可以了，这种方案是可以的。但是，如果使用`extension`方式对于 `CustomImageView` 和 `CustomButton`，根本看不出来任何抖动的意图，整个类里面没有任何东西能告诉你它需要抖动。相反，通过协议可以很直白的看出抖动的意图。这仅仅是面向协议的一个小小好处，除此之外在Swift中还有很多巧妙的用法。

```swift
import UIKit
extension UIView {
    func shake() {
    }
}
```

# 四、多态和面向接口

## 多态

[**多态**](https://baike.baidu.com/item/多态)（Polymorphism）按字面的意思就是“**多**种状态”。 在面向对象语言中，接口的**多**种不同的实现方式即为**多态**。对于一个引用变量，可以指向任何类的对象 （一个对外接口，多个内在实现）

### 多态的四个问题

引用[跳出面向对象思想\(二\) 多态](https://casatwy.com/tiao-chu-mian-xiang-dui-xiang-si-xiang-er-duo-tai.html)中Casa对多态四种情况的描述：

> 一般来说我们采用多态的场景还是很多的，有些在设计的时候就是用于继承的父类，希望子类覆盖自己的某些方法，然后才能够使程序正常运行下去。比如：
>
> ```
> BaseController需要它的子类去覆盖loadView等方法来执行view的显示逻辑
> BaseApiManager需要它的子类去覆盖methodName等方法来执行具体的API请求
> ```
>
> 以上是我列举的应用多态的几个场景，在基于上面提到的需求，以及站在代码观感的立场，我们在实际采用多态的时候会有下面四种情况：
>
> 1. 父类有部分public的方法是不需要，也不允许子类覆重
> 2. 父类有一些特别的方法是必须要子类去覆重的，在父类的方法其实是个空方法
> 3. 父类有一些方法是可选覆重的，一旦覆重，则以子类为准
> 4. 父类有一些方法即便被覆重，父类原方法还是要执行的

## 多态和面向接口的对比

以一个文件解析类为例，文件解析的过程中主要有两个步骤：读取文件和解析文件。假如实际中可能会有一些格式十分特殊的文件，所用到的文件读取方式和解析方式不同于常规方式。通常按照继承的写法可能会是下面这样。

```objective-c
//.h文件
@interface FileParseTool : NSObject
- (void)parse;
- (void)analyze;
@end
//.m文件
@implementation FileParseTool
- (void)parse {
    [self readFile];
    [self analyze];
}
- (void)readFile {
    //实现代码
    ....
}
- (void)analyze {
    //子类要重写该方法
}
@end
```

如果想实现对特殊格式文件的解析，此时可能会重写父类的`analyze`方法。

```objective-c
@interface SpecialFileParseTool: FileParseTool

@end

@implementation SpecialFileParseToll
- (void)analyze {
    NSLog(@"%@:%s", NSStringFromClass([self class]), __FUNCTION__);
}
@end
```

按照继承的写法，会存在以下问题：

* 父类中的`analyze`会有空方法挂在那里，对于父类而言没有任何实际意义。
* 如果架构工程师写父类，业务工程师实现子类。那么业务工程师很可能不清楚：哪些方法需要被覆盖重载，哪些不需要。如果子类没有覆重方法，而父类提供的只是空方法，就很容易出问题。如果子类在覆重的时候引入了其他不相关逻辑，那么子类对象就显得不够单纯，角色复杂了。

使用面向接口的方式实现代码如下：

```objective-c
//父类.h文件
@protocol FileParseProtocol <NSObject>
- (void)readFile;
- (void)analyze;
@end

@interface FileParseTool : NSObject
@property (nonatomic, weak) id<FileParseProtocol> assistant;
- (void)parse;
@end

//  FileParseToolt.m
@implementation FileParseTool
- (void)parse {
    [self.assistant readFile];
    [self.assistant analyze];
}
@end
```

```objective-c
//  SpecialFileParseTool.h
@interface SpecialFileParseTool: FileParseTool <FileParseProtocol>
@end

//SpecialFileParseTool.m
@implementation SpecialFileParseTool
- (instancetype)init {
    self = [super init];
    if (self) {
        self.assistant = self;
    }
    return self;
}
- (void)analyze {
    NSLog(@"analyze special  file");
}
- (void)readFile {
    NSLog(@"read special file");
}
@end
```

相比较于继承的写法，面向接口的写法恰好能弥补上述三个缺陷：

* 父类中将不会再用`analyze`空方法挂在那里。
* 原本需要覆盖重载的方法，不放在父类的声明中，而是放在接口中去实现。基于此，公司内部可以规定:`不允许覆盖重载父类中的方法`、`子类需要实现接口协议中的方法`，可以避免继承上带来的困惑。子类中如果引入了父类的外部逻辑，此时通过协议的控制，原本引入了不相关的逻辑也很容易被剥离。

现在可以回答本节开始引出Casa的四个问题：

> 1. 父类有部分public的方法是不需要，也不允许子类覆重
> 2. 父类有一些特别的方法是必须要子类去覆重的，在父类的方法其实是个空方法
> 3. 父类有一些方法是可选覆重的，一旦覆重，则以子类为准
> 4. 父类有一些方法即便被覆重，父类原方法还是要执行的

关于第一个问题，在利用面向接口的方案中，公司内部可以规定:`不允许覆盖重载父类中的方法`、`子类需要实现接口协议中的方法`。

关于第二个问题，第二个方案中父类`FileParseTool`的`.m`文件中不再存在空的`analyze`方法。

关于第三个问题，显然能在解答第一个问题中找到答案。

关于第四个问题，可能需要再补充一些代码来解决这个问题。主要思路是：通过在接口中设置哪些方法是必须要实现，哪些方法是可选实现的来处理对应的问题，由子类根据具体情况进行覆重。代码如下：

```objective-c
//父类.h文件
//流程管理相关接口，该协议可以定义子类必须实现的方法
@protocol FileParseProtocol <NSObject>
- (void)readFile;
- (void)analyze;
@end

//拦截相关接口，该协议可以定义可选的方法，子类可以根据实现情况选择是否重载父类方法
@protocol InterceptorProtocol <NSObject>
- (void)willBeginAnalyze;
- (void)didFinishAnalyze;
@end

@interface FileParseTool : NSObject
@property (nonatomic, weak) id<FileParseProtocol> assistant;
@property (nonatomic, weak) id<InterceptorProtocol> interceptor;
- (void)parse;
@end

//  FileParseToolt.m
@implementation FileParseTool
- (void)parse {
    [self.assistant readFile];
    if ([self.interceptor respondsToSelector:@selector(willBeginAnalyze)]) {
        [self.interceptor willBeginAnalyze];
    }
    [self.assistant analyze];
    if ([self.interceptor respondsToSelector:@selector(didFinishAnalyze)]) {
        [self.interceptor didFinishAnalyze];
    }
}
@end
```

```objective-c
//  SpecialFileParseTool.h
@interface SpecialFileParseTool: FileParseTool<FileParseProtocol,InterceptorProtocol>
@end

//SpecialFileParseTool.m
@implementation SpecialFileParseTool
- (instancetype)init {
    self = [super init];
    if (self) {
        self.assistant = self;
        self.interceptor = self;
    }
    return self;
}
- (void)analyze {
    NSLog(@"analyze special  file");
}
- (void)readFile {
    NSLog(@"read special file");
}
@end
```

## 何时使用多态

1. 如果在子类中可能被外界使用到，则应该采用多态的形式，对外提供接口；如果只是子类私有要更改的方法，则应该采用IOP更为合理。

2. 如果引入多态之后导致对象角色不够单纯，那就不应当引入多态，如果引入多态之后依旧是单纯角色，那就可以引入多态。

# 五、总结

文章的第一部分首先说了继承的代码复用性和高耦合性，然后总结了继承应当在何时使用，最后有说了四种替代继承的方案\(**协议**、**组合**、**类别**、**配置对象**\)；第二部分利用面向切面的思想，解决了iOS开发中关于ViewController继承的问题；第三部分简单介绍了面向接口的思想，以及和面向对象思想的比较；第四部分涉及多态和面向接口的抉择问题。

---

> 编辑：黄成
>
> 日期：2019.1.22



