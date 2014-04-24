Objective-C的命名空间
===

** 1.Why the hell is everything NS-whatever? **
** 2.为什么很多命名都是NS开头的呢？**

1.You'll hear that within the first minute of introducing someone to Objective-C. Guaranteed.
2.我保证在第一次介绍给别人Objective-C的时候肯定会听到这段话。

1. Like a parent faced with the task of explaining the concept of death or the non-existence of Santa, you do your best to be forthcoming with facts, so that they might arrive at a conclusion themselves.
2. 就像父母要向孩子解释什么是死亡或者圣诞老人是不存在的问题一样，他们寄希望与未来，希望将来孩子能够自己找到答案。

** 1. Why, Jimmy, NS stands for NeXTSTEP (well, actually, NeXTSTEP/Sun, but we'll cover that with "the birds & the bees" talk), and it's used to...**
** 2. 你既然这么问了，实际上NS代表了NeXTSTEP （好吧，其实是代表NeXTSTEP/Sun，我们只是做个简单的介绍），它被用于**

1. ...but by the time the words have left your mouth, you can already sense the disappointment in their face. Their innocence has been lost, and with an audible sigh of resignation, they start to ask uncomfortable questions about @
2. 但是随着时间的推移，你可以从他们的脸上看到失望两个字，他们不在只是随便问问了，他们开始问一些你更难解释的问题--什么是[@](http://nshipster.com/at-compiler-directives/)

---

1. Namespacing is the preeminent bugbear of Objective-C. A cosmetic quirk with global implications, the language's lack of identifier containers remains a source of prodigious quantities of caremad for armchair language critics.
2. 命名一直是Objective-C的一个硬伤，

1. This is all to say: unlike many other languages that are popular today, Objective-C does not provide a module-like mechanism for avoiding class and method name collisions.
2. 他们总是说：Objective-C不像如今其他流行的语言一样提供模块化的机制来避免类名和方法名的冲突。

1. Instead, Objective-C relies on prefixes to ensure that functionality in one part of the app doesn't interfere with similarly named code somewhere else.
2. 相反地，Objective-C 依靠前缀来确保APP中的一些地方的方法名不会影响其他的地方的相似名字的代码。

1. We'll jump into those right after a quick digression into type systems:
2. 插入一个关于类型系统的题外话之后我们会继续进入关于命名的讨论。

---

##Types in C & Objective-C
##C和Objective-C中的类型

1. As noted many times in this publication, Objective-C is built directly on top of the C language. One consequence of this is that Objective-C and C share a type system, requiring that identifiers are globally unique.
2. 我曾在这本书上多次提过，Objective-C是直接建立在C语言之上的，一个重要的原因是Objective-C和C语言共用一个类型系统，他们都要求标识符是全局唯一的。

---

1. You can see this for yourself—try defining a new static variable with the same name as an existing @interface, and the compiler will generate an error:
2. 你可以通过尝试定义一个和@interface同名的静态变量，编译之后你会得到一个错误：

{% highlight objc %}
@interface XXObject : NSObject
@end

||st|ati|c char * XXObject;  // Redefinition of "XXObject" as different kind |of |symbol
//将“XXObject”重新定义为不同的符号

{% endhighlight %}

1. That said, the Objective-C runtime creates a layer of abstraction on top of the C type system, allowing the following code to compile without even a snicker:
2. 也就是说，Objective-C的runtime在C语言的类型系统上又创建了一个抽象层，它甚至可以允许下面这段代码被编译:

{% highlight objc %}

@protocol Foo
@end

@interface Foo : NSObject <Foo> {
||  |  ||id Fo|o;||
}

@property id Foo;
+ (id)Foo;
- (id)Foo;
@end

@interface Foo (Foo)
@end

@implementation Foo
@synthesize Foo;

+ (id)Foo {
||  |  ||id Fo|o = @"Foo";||
||  |  ||retur|n Foo;||
}
@end

{% endhighlight %}

1. Within the context of the Objective-C runtime, a program is able to differentiate between a class, a protocol, a category, an instance variable, an instance method, and a class method all having the same name.
2. 通过Objective-C的上下文，程序能区别所有相同名字的类，协议，类别，实例变量，实例方法和类方法。

** 1. That a variable can reappropriate the name of an existing method is a consequence of the C type system (which similarly allows for a variable to shadow the name of its containing function) **
** 2. 一个变量能重命名一个已经存在的方法也是得益与C语言的类型系统（这个有点像一个变量能够隐藏它的内含函数）**

##Prefixes
##前缀

---

1. All classes in an Objective-C application must be globally unique. Since many different frameworks are likely have some conceptual overlap—and therefore an overlap in names (users, views, requests / responses, etc.)—convention dictates that class names use 2 or 3 letter prefix.
2. 在Objective-C应用中的所有类名都必须是全局唯一的。由于很多不同的框架中会有一些相似的功能，所以在名字上也可能会有重复（users， views， requests / responses 等等），所以[苹果官方文档](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)规定类名需要有2-3个字母作为前缀。

###Class Prefixes
###类前缀

1. Apple recommends that 2-letter prefixes be reserved for first-party libraries and frameworks, while third-party developers (that's us) opt for 3 letters or more.
2. 苹果官方[建议](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)两个字母作为前缀的类名是为官方的库和框架准备的，而作为第三方的开发者（也就是我们）建议使用3个或者更多的字母去命名我们的类。

1. A veteran Mac or iOS developer will have likely memorized most if not all of the following abbreviated identifiers:
2. 一个资深的Mac或iOS开发者可能会记得下面大部分的缩写标识符：

|Prefix|Frameworks|
|---|:---:|
||AB|  |AddressBook / AddressBookUI||
||AC|   |Accounts||
||AD|   |iAd||
||AL|   |AssetsLibrary||
||AU|   |AudioUnit||
||AV|   |AVFoundation||
||CA|   |CoreAnimation||
||CB|   |CoreBluetooth||
||CF|   |CoreFoundation / CFNetwork||
||CG|   |CoreGraphics / QuartzCore / ImageIO||
||CI|   |CoreImage||
||CL|   |CoreLocation||
||CM|   |CoreMedia / CoreMotion||
||CV|   |CoreVideo||
||EA|   |ExternalAccessory||
||EK|   |EventKit / EventKitUI||
||GC|   |GameController||
|GLK* | GLKit|
||JS|   |JavaScriptCore||
||MA|   |MediaAccessibility||
||MC|   |MultipeerConnectivity||
||MF|   |MessageUI*||
||MI|DI*|  CoreMIDI||
||MK|   |MapKit||
||MP|   |MediaPlayer||
||NK|   |NewsstandKit||
||NS|   |Foundation, AppKit, CoreData||
||PK|   |PassKit||
||QL|   |QuickLook||
||SC|   |SystemConfiguration||
||Se|c* |  Security*||
||SK|   |StoreKit / SpriteKit||
||SL|   |Social||
||SS|   |Safari Services||
||TW|   |Twitter||
||UI|   |UIKit||
||UT|   |MobileCoreServices||

####3rd-Party Class Prefixes
####第三方类前缀
----

1. Until recently, with the advent of CocoaPods and a surge of new iOS developers, the distribution of open source, 3rd-party code had been largely a non-issue for Apple and the rest of the Objective-C community. Apple's naming guidelines came about recently enough that the advice to adopt 3-letter prefixes is only just becoming accepted practice.
2. 知道最近，由于[CocoaPods](http://cocoapods.org/)的出现和大量新的iOS开发者的涌现，开源代码的分布，第三方代码在很大程度上对苹果和其余的Objective-C开发社区来说已经不是问题了。就在最近苹果官方的命名指南也发生了变化，它将三个字母作为前缀的建议只是作为一个习惯做法。

1. Because of this, many established libraries still use 2-letter prefixes. Consider some of these most-starred Objective-C repositories on GitHub.
2. 正因为这样，那些已经存在的第三方库依然使用2个字母作为前缀，你可以参考一些那些在GitHub上得到很多[start的Objective-C的仓库](https://github.com/search?l=Objective-C&q=stars%3A%3E1&s=stars&type=Repositories)。

|Prefix|Frameworks|
|---|:---:|
|AF | AFNetworking ("Alamofire")|
|RK | RestKit|
|GPU|  GPUImage|
|SD | SDWebImage|
|MB | MBProgressHUD|
|FB | Facebook SDK|
|FM | FMDB ("Flying Meat")|
|JK | JSONKit|
|FUI|  FlatUI|
|NI | Nimbus|
|RAC|  Reactive Cocoa|

1. Seeing as how we're already seeing prefix overlap among 3rd-party libraries, make sure that you follow a 3+-letter convention in your own code.
2. 我们已经看到在第三方库中很多前缀已经一样了，所以要在你的代码中遵守要[三个字母以上的作为类前缀的规定](https://github.com/AshFurrow/AFTabledCollectionView)。

** 1. For especially future-focused library authors, consider using @compatibility_alias to provide a seamless migration path for existing users in your next major upgrade.**
** 2. 对于那些针对特殊功能而写的第三方库的作者，可以考虑在下一次主要升级时使用@compatibility_alias来为那些使用者们提供一个天衣无缝的转移路径。**

##Method Prefixes
##方法前缀

1. It's not just classes that are prone to naming collisions: selectors suffer from this too—in ways that are even more problematic than classes.
2. 不仅是类容易造成命名冲突，selectors也很容易造成命名冲突，甚至有比类更多的问题。

1. Consider the category:
2. 思考一下这个类别：

{% highlight objc %}
@interface NSString (PigLatin)
- (NSString *)pigLatinString;
@end

{% endhighlight %}

1. If -pigLatinString were implemented by another category (or added to the NSString class in a future version of iOS or Mac OS X), any calls to that method would result in undefined behavior, since no guarantee is made as to the order in which methods are defined by the runtime.
2. 如果 `-pigLatinString`方法被另一个类别实现了（或者以后版本的iOS或者Mac OS X 在NSString类中也添加了同样名字的方法），那么调用这个方法就会得到未定义的行为错误，因为runtime中，我们不能保证哪个方法会先被定义。

1. This can be guarded against by prefixing the method name, just like the class name (prefixing the category name isn't a bad idea, either):
2. 我们可以通过在方法名前加前缀来避免这个问题，就像这个类名（在类别名前加前缀也是个好办法）：

{% highlight objc %}
@interface NSString (XXXPigLatin)
- (NSString *)xxx_pigLatinString;
@end

{% endhighlight %}

1.Apple's recommendation that all category methods use prefixes is even less widely known or accepted than its policy on class prefixing.
2.苹果官方建议[所有分类方法使使用前缀](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html#//apple_ref/doc/uid/TP40011210-CH6-SW4)，这个建议比类名需要加前缀的规定更加广为人知和接受。

1. There are many outspoken developers who will passionately argue one side or another. However, weighing the risk of collision against its likelihood, the cost/benefit analysis is not entirely clear-cut:
2. 现在有很多直言不讳的开发者会非常热烈地讨论这个建议的一方面或另一方面。然而，通过花费或者效益分析来衡量命名冲突风险的可能性是不全面的:

1. The main feature of categories is coating useful functionality with syntactic sugar. Any category method could alternatively be implemented as a function taking an explicit argument in place of the implicit self of a method.
2. 类别的主要功能是通过语法糖将一些有用的功能包裹进原来的类中。任何一个类别方法都可以被选择性实现，你也可以把他当做是self的一个隐型的功能方法。

1. Collisions can be detected at compile time by setting the OBJC_PRINT_REPLACED_METHODS environment variable to YES. In practice, collisions are extremely rare, and when they do occur, they're usually an indicator of functionality that is needlessly duplicated across dependencies. Although the worst-case scenario is a runtime exception, it's entirely likely that two methods named the same thing will actually do the same thing, and result in no change in behavior. All of those Swiss Army Knife categories that defined NSArray -firstObject continued to march on once the method was officially added.
2. 当我在编译器的环境参数中将`OBJC_PRINT_REPLACED_METHODS`这个参数设置为YES，那我们就能在编译的时候检测方法名的冲突。实际上，方法名的冲突是很少发生的，而且在发生的时候，他们通常会显示一个通过依赖关系不需要复制的功能性指示。即使发生最坏的情况，程序在运行是出现异常，那么很可能是两个方法名一样，那么他们做的事情也是一样的，所以结果也不会有什么变化。就像Swiss Army Knife写了一个类别，他定义了`NSArray`中的` -firstObject`这个方法，那么只要苹果官方没有在`NSArray`中加这个方法的话，那么这个类别方法一直有效的。

1. Just as with constitutional scholarship, there will be strict and loose interpretations of Apple's programming guidelines. Those that see it as a living document would point out that... actually, you know what? If you've read this far and are still undecided, just prefix your damn category methods. If you choose not to, just be mindful that it could bite you in the ass.
2. 就像宪法奖学金一样，对于苹果官方的编程指南有严肃又松散的解释。这里没有固定的文档，他们可能总会一直变化。看到这里，如果你还是悬而未决，那么你只需要把的类别方法名加上前缀，如果你还是选择不去做任何改变，那么你就等着自食其果吧。

###Swizzing
###混合
---

1. The one case where method prefixing (or suffixing) is absolutely necessary is when doing method replacement, as discussed in last week's article on swizzling.
2. 在方法混合时，方法名加前缀或者后缀也是非常有必要的，这个我在上周关于swizzling的文章中提到过。

{% highlight objc %}
@implementation UIViewController (Swizzling)

	- (void)xxx_viewDidLoad {
		||- |   | [self xxx_viewDidLoad];||
		-
			||- |   |     // Swizzled implementation||
			||- |   |     }||
			-
{% endhighlight %}

##Do We Really Need Namespaces?
##我们真的需要命名空间么？

1. With all of the recent talk about replacing / reinventing / reimagining Objective-C, it's almost taken as a given that namespacing would be an obvious feature. But what does that actually get us?
2. 在最近关于Objective-C替换、改造、重塑的讨论中，我可以明显地发现命名空间是未来的一个趋势。但是它到底给我们带来了什么呢？

1. Aesthetics? Aside from IETF members and military personnel, nobody likes the visual aesthetic of CLAs. But would ::, /, or an extra . really make matters better? Do we really want to start calling NSArray "Foundation Array"? (And what would I do with NSHipster.com ?!)
2. 美学？除了IETE成员和军事人员，我想没有人会喜欢CLAs的视觉审美，但是用::，/或者另外的.这些符号真的能让我们觉得更好么？你真的想要以后把`NSArray`叫做`Foundation Array`？（那我这个NSHipster.com这个博客不是也得改名字了?!）

1. Semantics? Start to look closely at any other language, and how they actually use namespaces, and you'll realize that namespaces don't magically solve all matters of ambiguity. If anything, the additional context makes things worse.
2. 语义学？我比较一下其他的语言，看看他们是怎么用命名空间的，那么你就会意识到命名控件不能解决所有不明确的问题。可能在某些额外环境的情况下，那些命名空间会出现更多问题。

1. Not to create a straw man, but an imagined implementation of Objective-C namespaces probably look a lot like this:
2. 先不要这么做好决定，你只要想一下Objective-C的命名空间的实现可能会像这个样子:

{% highlight objc %}

@namespace XX
    @implementation Object

    @using F: Foundation;

    - (void)foo {
        F:Array *array = @[@1,@2, @3];

        // ...
    }

    @end
@end

{% endhighlight %}

1. What we have currently—warts and all—has the notable advantage of non-ambiguity. There is no mistaking NSString for anything other than what it is, either by the compiler or when we talk about it as developers. There are no special contextual considerations to consider when reading through code to understand what actors are at play. And best of all: class names are exceedingly easy to search for.
2. 我们有繁琐的代码但也有容易理解的明显优势。我们作为开发者去讨论NSString的时候，我们不会把它理解成别的意思，编译器也是一样。当我们在阅读代码时，我们不需要过多地去考虑这些代码是什么作用的。并且最重要的是，类名非常[容易就可以找到](http://lmgtfy.com/?q=NSString)。

---

不管怎样，如果你对这个讨论感兴趣的话，我强烈建议你看一下[Kyle Sluder](http://optshiftk.com/)的[《 this namespace feature proposal 》](http://optshiftk.com/2012/04/draft-proposal-for-namespaces-in-objective-c/)。非常值得一看。
