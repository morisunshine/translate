Objective-C的命名空间
===

1.Why the hell is everything NS-whatever?
2.为什么很多命名都是NS开头的呢？

---

1.You'll hear that within the first minute of introducing someone to Objective-C. Guaranteed.
2.我保证在第一次介绍给别人Objective-C的时候肯定会听到这段话。

---

1. Like a parent faced with the task of explaining the concept of death or the non-existence of Santa, you do your best to be forthcoming with facts, so that they might arrive at a conclusion themselves.
2. 就像父母要向孩子解释什么是死亡或者圣诞老人是不存在的问题一样，他们寄希望与未来，希望将来孩子能够自己找到答案。

---

11. Why, Jimmy, NS stands for NeXTSTEP (well, actually, NeXTSTEP/Sun, but we'll cover that with "the birds & the bees" talk), and it's used to...
2. 你既然这么问了，实际上NS代表了NeXTSTEP （好吧，其实是代表NeXTSTEP/Sun，我们只是做个简单的介绍），它被用于

---

1. ...but by the time the words have left your mouth, you can already sense the disappointment in their face. Their innocence has been lost, and with an audible sigh of resignation, they start to ask uncomfortable questions about @
2. 但是随着时间的推移，你可以从他们的脸上看到失望两个字，他们不在只是随便问问了，他们开始问一些你更难解释的问题--什么是@

---

1. Namespacing is the preeminent bugbear of Objective-C. A cosmetic quirk with global implications, the language's lack of identifier containers remains a source of prodigious quantities of caremad for armchair language critics.
2. 命名一直是Objective-C的一个硬伤，

---

1. This is all to say: unlike many other languages that are popular today, Objective-C does not provide a module-like mechanism for avoiding class and method name collisions.
2. 他们总是说：Objective-C不像如今其他流行的语言一样提供模块化的机制来避免类名和方法名的冲突。

---

1. Instead, Objective-C relies on prefixes to ensure that functionality in one part of the app doesn't interfere with similarly named code somewhere else.
2. 相反地，Objective-C 依靠前缀来确保APP中的一些地方的方法名不会影响其他的地方的相似名字的代码。

---
1. We'll jump into those right after a quick digression into type systems:
2. 插入一个关于类型系统的题外话之后我们会继续进入关于命名的讨论。

---

##Types in C & Objective-C
##C和Objective-C中的类型

---

1. As noted many times in this publication, Objective-C is built directly on top of the C language. One consequence of this is that Objective-C and C share a type system, requiring that identifiers are globally unique.
2. 我曾在这本书上多次提过，Objective-C是直接建立在C语言之上的，一个重要的原因是Objective-C和C语言共用一个类型系统，他们都要求标识符是全局唯一的。

---

1. You can see this for yourself—try defining a new static variable with the same name as an existing @interface, and the compiler will generate an error:
2. 你可以通过尝试定义一个和@interface同名的静态变量，编译之后你会得到一个错误：

```
@interface XXObject : NSObject
@end

static char * XXObject;  // Redefinition of "XXObject" as different kind of symbol
//将“XXObject”重新定义为不同的符号

```

也就是说，Objective-C的runtime在C语言的类型系统上又创建了一个抽象层，它甚至可以允许下面这段代码被编译:

```
@protocol Foo
@end

@interface Foo : NSObject <Foo> {
    id Foo;
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
    id Foo = @"Foo";
    return Foo;
}
@end

```

---

1. Within the context of the Objective-C runtime, a program is able to differentiate between a class, a protocol, a category, an instance variable, an instance method, and a class method all having the same name.
2. 通过Objective-C的上下文，程序能区别所有相同名字的类，协议，类别，实例变量，实例方法和类方法。

---

1. That a variable can reappropriate the name of an existing method is a consequence of the C type system (which similarly allows for a variable to shadow the name of its containing function)
2. 一个变量能重命名一个已经存在的方法也是得益与C语言的类型系统（这个有点像一个变量能够隐藏它的内含函数）

##Prefixes
##前缀

---

1. All classes in an Objective-C application must be globally unique. Since many different frameworks are likely have some conceptual overlap—and therefore an overlap in names (users, views, requests / responses, etc.)—convention dictates that class names use 2 or 3 letter prefix.
2. 在Objective-C应用中的所有类名都必须是全局唯一的。由于很多不同的框架中会有一些相似的功能，所以在名字上也可能会有重复（users， views， requests / responses 等等），所以苹果官方文档规定类名需要有2-3个字母作为前缀。

###Class Prefixes
###类前缀

1. Apple recommends that 2-letter prefixes be reserved for first-party libraries and frameworks, while third-party developers (that's us) opt for 3 letters or more.
2. 苹果官方[建议](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html)两个字母作为前缀的类名是为官方的库和框架准备的，而作为第三方的开发者（也就是我们）建议使用3个或者更多的字母去命名我们的类。

---

1. A veteran Mac or iOS developer will have likely memorized most if not all of the following abbreviated identifiers:
2. 一个资深的Mac或iOS开发者可能会记得下面大部分的缩写标识符：

..
####3rd-Party Class Prefixes
####第三方类前缀

1. Until recently, with the advent of CocoaPods and a surge of new iOS developers, the distribution of open source, 3rd-party code had been largely a non-issue for Apple and the rest of the Objective-C community. Apple's naming guidelines came about recently enough that the advice to adopt 3-letter prefixes is only just becoming accepted practice.
2. 知道最近，由于CocoaPods的出现和大量新的iOS开发者的涌现，开源代码的分布，第三方代码在很大程度上对苹果和其余的Objective-C开发社区来说已经不是问题了。就在最近苹果官方的命名指南也发生了变化，它将三个字母作为前缀的建议只是作为一个习惯做法。

---

1. Because of this, many established libraries still use 2-letter prefixes. Consider some of these most-starred Objective-C repositories on GitHub.
2. 正因为这样，那些已经存在的第三方库依然使用2个字母作为前缀，你可以参考一些那些在GitHub上得到很多start的Objective-C的仓库。

...

1. Seeing as how we're already seeing prefix overlap among 3rd-party libraries, make sure that you follow a 3+-letter convention in your own code.
2. 我们已经看到在第三方库中很多前缀已经一样了，所以要在你的代码中遵守要三个字母以上的作为类前缀的规定。

---

1. For especially future-focused library authors, consider using @compatibility_alias to provide a seamless migration path for existing users in your next major upgrade.
2. 对于那些针对特殊功能而写的第三方库的作者，可以考虑在下一次主要升级时使用@compatibility_alias来为那些使用者们提供一个天衣无缝的转移路径。

##Method Prefixes
##方法前缀

1. It's not just classes that are prone to naming collisions: selectors suffer from this too—in ways that are even more problematic than classes.
2. 不仅是类容易造成命名冲突，selectors也很容易造成命名冲突，甚至有比类更多的问题。

---

1. Consider the category:
2. 思考一下这个类别：

```
@interface NSString (PigLatin)
- (NSString *)pigLatinString;
@end

```

1. If -pigLatinString were implemented by another category (or added to the NSString class in a future version of iOS or Mac OS X), any calls to that method would result in undefined behavior, since no guarantee is made as to the order in which methods are defined by the runtime.
2. 如果 `-pigLatinString`方法被另一个类别实现了（或者以后版本的iOS或者Mac OS X 在NSString类中也添加了同样名字的方法），那么调用这个方法就会得到未定义的行为错误，因为runtime中，我们不能保证哪个方法会先被定义。

---

1. This can be guarded against by prefixing the method name, just like the class name (prefixing the category name isn't a bad idea, either):
2. 我们可以通过在方法名前加前缀来避免这个问题，就像这个类名（在类别名前加前缀也是个好办法）：

```
@interface NSString (XXXPigLatin)
- (NSString *)xxx_pigLatinString;
@end

```

