
###Real-World Testing with XCTest
###使用XCTest进行真实测试

Almost four months ago, our team (Marco, Arne, and Daniel) set out to write the model layer of a new app. We wanted to use testing as part of our development process. After some discussion, we chose XCTest as our testing framework.

差不多四个月以前，我们团队(Marco, arne和Daniel)开始着手为我们的新应用写模型层。我们想在开发过程中使用测试，经过一些讨论之后，我们选择[XCTest](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/Introduction/Introduction.html)作为我们的测试框架。

Our code base (excluding tests) has since grown to 544 kilobytes of source code spread across 190 files and 18,000 lines. Our tests click in at about 1,200 kilobytes, roughly twice that of the code they’re testing. We’re not quite done, yet, but closing in. We wanted to share some of what we have learned, both about testing in general, and about testing with XCTest.

现在我们的编码基数已经从原来的544字节增长到190个文件和18,000行代码。我们测试部分的代码现在差不多有1,200字节，是被测试代码的字节数的两倍。虽然我们还没有完全写好测试代码，但是已经接近尾声。我们在这里想和大家分享在这过程中我们所学到的东西，包括平常的测试和如何用XCTest来做测试。

Note that some of the model classes and methods in this article have been renamed, because the project is not in the App Store, yet.

这里需要注意的是在文章中提到的一些模型类和方法已经被重命名了，因为这个项目还没有在App Store中上线。

We chose XCTest for its simplicity and its tight integration into the Xcode IDE. With this article, we hope to shed some light on when XCTest is a good option, and when you might want to pick something else.

我们选择XCTest作为我们的测试框架是因为它非常简单并且Xcode的IDE中直接集成。通过这篇文章，我们想表达XCTest是测试框架中一个很好的选择，当然你们可能会选择其他的。

We tried to link to the other articles in this issue where it makes sense.

我们还在文章中添加了关于[测试这个问题](http://www.objc.io/issue-15/)的其他文章的链接，使这篇文章更加有意义。

##Why We Are Testing
##为什么我们需要测试？

As the [article about bad practices in testing](http://www.objc.io/issue-15/bad-testing-practices.html) mentions, many people believe that “the only time a test will give value back is when we want to change our code.” There’s obviously more to it, and you should read that article. But it is also important to remember that even when writing the first version of any code, most of the time will be spent changing code—as the project evolves, more and more features get added, and we realize that behavior needs to change slightly here and there. So even though you’re not yet working on version 1.1 or 2.0, you’ll still do a lot of changing, and tests do provide invaluable help at that point in time.

就像[这篇关于糟糕的测试的文章](http://www.objc.io/issue-15/bad-testing-practices.html)中提到的那样，很多人都认为“只有当我们的改变代码时，才需要测试返回的值是否于预期的一样。” 显然这篇文章中有很多例子，你需要仔细读这篇文章。但是这里也有一点非常重要，就是虽然我们在写最早版本的代码，但我们还是会将大部分时间花在修改代码上--随着项目的发展，越来越多的功能会被加进来，我们会发现很多地方都需要稍微改一下。所以即使你还没有在做1.1或2.0版本，但你还是要做大量的修改，而测试就是在时间上为我们提供了非常重要的帮助。

We are still finishing the initial version of our framework, and have just recently passed 1,000 test cases with more than 10 man months of effort spent on the project. Even though we had a clear vision of the project’s architecture, we’ve still had to change and adapt the code along the way. The ever-growing set of test cases have helped us do this.

我们依然还在完成我们框架的最初版本，在超过10个[人月](https://en.wikipedia.org/wiki/Man-hour)的努力下，我们通过了将近1,000个测试。现在已经有一个比较清楚的架构，但是我们仍然需要沿着这个方向，去修改和调整我们的代码。这套不断增长的测试用例会帮我们做到这一点。

They have given us a level of comfort with the quality of our code, and at the same time given us the ability to refactor and change the code while knowing that we didn’t break things. And we have been able to actually run our code from day one; we didn’t have to wait for all the pieces to be in place.

测试用例使我们的代码变得舒适，也使我们又得到一个新的技能，就是在我们重构或者修改代码时，我们知道我们的修改没有破坏其他东西。而且我们可以在项目开始的第一天就能运行我们的程序，而不用等到万事俱备。

##How XCTest Works
##XCTest如何运行

Apple has some decent documentation on using XCTest. Tests are grouped into classes that subclass from XCTestCase. Each method that has a name starting with test is a test.

苹果提供了一些关于[如何使用XCTest](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/Introduction/Introduction.html)的官方文档。测试用例被分配到不同的类，他们都是继承XCTestCase。每个以`test`为开头的方法都是一个测试用例。

Because tests are simply classes and methods, we can add @property and helper methods to the class, as we see fit.

因为测试是简单的类和方法，所以我们可以适当地添加一些`@property`和辅助方法。

In order to be able reuse code, we have a common superclass, TestCase, for all our test classes. This class subclasses from XCTestCase. All our test classes subclass from our TestCase.

考虑到代码的重用性，我们的所有测试用例类都有一个共同的父类，也就是`TestCase`，它也是`XCTestCase`的子类，所有的测试类都是我们的`TestCase`类的子类。

We then put shared helper methods into the TestCase class. And we even have properties on it that get pre-populated for each test.

然后我们把一些公用的辅助方法放在`TestCase`类中，并且加了一些property作为每个类的预置property。

##Naming
##命名

Since a test is simply a method that begins with the word test, a typical test method would look like this:

因为测试用例仅仅只是一个以`test`为开头的方法，所以典型的测试方法看起来就像这样：

- (void)testThatItDoesURLEncoding
{
  // test code
}

All our tests start with the words “testThatIt”. Another frequently used way of naming tests is to use the method or class being tested, as in testHTTPRequest. But which class and method is being tested should be obvious by simply looking at the test.

我们的所有测试用例中都是以“testThatIt”为开头。而另外一个用的比较多的命名方式是“test+要测试的方法和类名”，比如像`testHTTPRequest`。但是这些被测试的类和方法需要在测试中很容易被找到。

The “testThatIt” style shifts the focus to the desired outcome, which, in most cases, is more difficult to understand at first glance.

“testThatIt”这类命名方式将重点转移到期望结果上，但是大多数情况下，这很难让我们一眼就能理解这个测试用例的意思。

There is a test case class for each production code class, and one is named after the other, e.g. HTTPRequest and HTTPRequestTests. If a class gets a bit bigger, we use categories to split up the test by topics.

这里一个测试用例类对应一个产品代码类，而且测试用例类的名字是根据被测试代码的名字，比如，`HTTPRequest` 和 `HTTPRequestTests`。如果一个类变得有点大，我们就可以根据主题来给它分类。

If we ever need to disable a test, we simply prefix the name with DISABLED:

比如我们想要禁止一个测试用例，我们只需要在方法名字前加`DISABLED`:

- (void)DISABLED_testThatItDoesURLEncoding

It’s easy to search for this string, and since the method name no longer starts with test, XCTest will skip this method.

我们很容易就能找到这个方法，并且因为这个方法不再是以test为开头，所以XCTest在运行时也会跳过这个测试。

##Given / When / Then
##Given / When / Then

We structure our tests by using the Given-When-Then pattern—every test is split into three parts.
我们可以根据`Given-When-Then`模式来组织我们的测试用例，将测试用例拆分成三个部分。

The given section sets up the environment for the test by creating model objects or bringing the system under test to a certain state. The when section contains the code we want to test. In most cases, this is only one method call. In the then section, we check the result of our action: Did we get the desired output? Was the object changed? This section consists mainly of assertions.

`given`这部分是建立测试环境，我们可以通过创建模型对象或将被测试的系统设置到指定的状态。`when`这部分包含了我们要测试的代码。在大部分情况，一个测试用例中只有一个方法会被调用。在`then`这部分中
，我们需要检查我们行为的结果--是否得到了我们期望的结果？对象是否有改变？这部分会有很多断言。

In a fairly simple test, it looks like this:
就像下面这个简单的测试用例一样：

- (void)testThatItDoesURLEncoding
{
    // given
    NSString *searchQuery = @"$&?@";
    HTTPRequest *request = [HTTPRequest requestWithURL:@"/search?q=%@", searchQuery];

    // when
    NSString *encodedURL = request.URL;

    // then
    XCTAssertEqualObjects(encodedURL, @"/search?q=%24%26%3F%40");
}

This simple pattern makes it easier to write and understand tests, as they all follow the same basic pattern. For faster visual scanning, we even put the words “given,” “when,” and “then” as comments on top of their respective sections. This way, the method being tested immediately sticks out.

这种简单的模式使我们能够更容易地写和理解这些测试用例，因为他们都遵循了同样的模式。为了更快的视觉扫描，我们会在每个部分的代码上写“given”，“when”，“then”的注释。通过这种方式，这个方法就能很快被理解。

##Reusable Code
##可重用代码

Over time, we noticed that we repeated some code again and again in our tests, like waiting for an asynchronous operation to finish or setting up an in-memory Core Data stack. To avoid code duplication, we began to gather all these useful snippets into a common base class for all our tests.
随着时间的流逝，我们注意到在我们的测试用例中有越来越多的重复代码，比如等待异步才能完成的操作，或者设置一个内存中的Core Data堆栈。为了避免代码重复，我们开始整理所有有用的代码片段，并将它们加入到一个公共类中，为所有的测试用例服务。

It turned out that this is not only useful as a collection of utility methods. The test base class can run its own -setUp and -tearDown methods to set up the environment. We use this mostly to initialize Core Data stacks for testing, to reseed our deterministic NSUUID (which is one of those small things that makes debugging a lot easier), and to set up background magic to simplify asynchronous testing.
结果证明这个公共类是一个非常实用的方法的集合。这个测试基础类能够运行自己的`-setUp`和`-tearDown`方法来配置环境。我们大部分情况用它来初始化测试用的Core Data堆栈，来重新设置我们的确定的`NSUUID`(这是可以让调试更简单的类)，并且设置background magic来简化异步测试。

Another useful pattern we started using recently is to implement delegate protocols directly in our XCTestCase classes. This way, we don’t have to awkwardly mock the delegate. Instead, we can interact with the tested class in a fairly direct way.
另外一个我们最近开始用的模式也很有用，它是在`XCTestCase`类中直接实现delegate协议。通过这个方式，我们不用必须笨拙地mock这个delegate。相反的，我们可以相当直接地与被测试的类互动。

##Mocking
##Mocking

Our mocking framework is OCMock. As described in the article about mocking in this issue, a mock is an object that returns a prepared answer for method calls.
我们使用的mock框架是[OCMock](http://ocmock.org/)。就像在这篇关于[mocking](http://www.objc.io/issue-15/mocking-stubbing.html)的文章中描述的那样，mock是一个在方法调用时返回标准答案的对象。

We use mocks for all dependencies of an object. This way, we can test the behavior of a class in isolation. The obvious disadvantage is that changes in a class don’t automatically lead to failing unit tests for other classes that depend on it. But this is remedied by the fact that we also have integration tests, which test all the classes together.
我们用mock来管理一个对象的所有依赖项。通过这个方式，我们可以测试这个类在隔离情况下的行为。但是这里有个明显的缺点，那就是当我们修改了一个类后，其他依赖与这个类的类不能自动地导致单元测试失败。但是关于这一点我们可以通过集成测试来补求，因为它可以测试所有的类。

It is important not to ‘over-mock,’ which is the habit of mocking every object except for the one being tested. When we started, we got into this habit at times, and we even mocked rather simple objects that were used as input to methods. Now we use many objects just the way they are, without mocking them.
不要‘over-mock’这一点很重要，就是我们要mock每个对象但是被测试的对象除外。当我们刚开始的时候，我们有时会进入这个习惯，我们甚至会mock那些简单到可以作为方法参数的对象。先我们用本身的方式来使用这些对象，而不用去mock他们。

As part of our common superclass for all test classes, we also added a
作为我们所有测试类的公共父类的一部分，我们还加入这个方法

- (void)verifyMockLater:(id)mock;

method. It makes sure that the mocks get verified at the end of that method / test. This makes using mocks even more convenient. We can specify that a mock should be verified right at the point where we create that mock:

它可以确保这个mock会在这个方法结束的时候被审查，这样使用mock就会更加方便。我们可以指定一个mock在创建这个mock的时候才会被审查。

- (void)testThatItRollsBackWhenSaveFails;
{
    // given
    id contextMock = [OCMockObject partialMockForObject:self.uiMOC];
    [self verifyMockLater:contextMock];
    NSError *error = [NSError errorWithDomain:NSCocoaErrorDomain code:NSManagedObjectValidationError userInfo:nil];
    [(NSManagedObjectContext *)[[contextMock stub] andReturnValue:@NO] save:[OCMArg setTo:error]];

    // expect
    [[contextMock expect] rollback];

    // when
    [ZMUser insertNewObjectInManagedObjectContext:self.uiMOC];
    [self.uiMOC saveOrRollback];
}

##State and Statelessness
##状态性和无状态性

A lot has been said about stateless code during the last few years. But at the end of the day, our apps need state. Without state, most apps would be pretty pointless. But managing state is the source of a lot of bugs, because managing state is very complex.
无状态性的代码在过去几年中一直被提起。但是在今天结束的时候，我们的app还是需要状态。如果没有状态，大部分app就会变得没有意义。但是状态的管理又会引起很多bug，因为管理状态非常复杂。

We made our code a lot easier to work on by isolating the state. A few classes contain state, while most are stateless. In addition to the code, testing got a whole lot easier, too.
我们通过隔离这些状态来使我们的代码能更容易工作。一些类中包含状态，而大部分则是无状态的。除了代码，测试也变得更加简单。

For example, we have a class, EventSync, that is responsible for sending local changes to our server. It needs to keep track of which local objects have changes that need to get pushed to the server, and which changes are currently being pushed. We can send multiple changes at once, but we don’t want to send the same change twice.
比如说，我们有一个叫`EventSync`的类，它是负责发送本地变化到我们的服务器。它需要跟踪哪些本地对象有过改变需要上传到服务器中，哪些改变现在正在被上传到服务器。我们需要一次发送多个变化，但是我们不想发送重复的变化。

We also had interdependencies between objects to keep track of. If A has a relationship to B, and B has local changes, we need to wait for those local changes to be pushed first, before we can send the changes of A.
我们也有跟踪对象之间的依赖关系。当**A**和**B**有依赖关系，我们需要在发送**A**的变化前先发送**B**的本地变化。

We have a UserSyncStrategy that has a -nextRequest method that generates the next request. This request will send local changes to the server. The class itself is stateless, though. Or rather: All its state is encapsulated inside an UpstreamObjectSync class, which keeps track of all the user objects that have local changes, and for which we have running requests. There is no state outside this class.
我们有一个`UserSyncStrategy`类，它有个一个`-nextRequest`方法可以生成下一次请求。这个请求将会发送本地改变到服务器中。虽然，这个类本身是无状态的。更精确地说，所有它的状态都被封装在一个叫`UpstreamObjcetSync`的类中，它负责跟踪那些有本地改变的用户对象，还有那些我们正在运行的请求。除了这个类之外都是没有状态的。

This way, it was easy for us to have one set of tests for the UpstreamObjectSync. They check that this class manages the state correctly. For the UserSyncStrategy, we were mocking the UpstreamObjectSync and didn’t have to worry about any state inside the UserSyncStrategy itself. That reduced the test complexity a lot, even more so because we were syncing many different kind of objects, and our different classes were all stateless and able to re-use the UpstreamObjectSync class.
通过这个方式，我们可以很容易得到测试`UpstremObjectSync`的集合。他们检查这个类正确地管理状态。对于`UserSyncStrategy`，当我们正在`mockUpstremObjectSync`的时候，不用担心在`UserSyncStrategy`本身的一些状态。这也减少了测试的复杂度，更如此因为我们正在同步很多不同的对象，并且我们不同的类都是无状态的，还可以重用`UpstreamObjectSync`类。

##Core Data
##Core Data

Our code relies heavily on Core Data. Since we need our tests to be isolated from one another, we have to bring up a clean Core Data stack for each test case, and tear it down afterward. We need to be sure that we don’t reuse the store from one test case for the next test.
我们的代码有点过于依赖于[Core Data](https://developer.apple.com/technologies/mac/data-management.html)。因为我们需要我们的测试是相互隔离的，这样我们就必须为每个测试用例创建一个**clean**的Core Data堆栈，然后再销毁它。我们需要确保我们在这个测试用例到下个测试用例中没有重复使用一个存储。

All of our code is centered around two managed object contexts: one that the user-interface uses and that is tied to the main queue, and one that we use for synchronization and that has its own private queue.
我们的所有代码都是以两个managed object context为中心：一个是用户界面时要使用的，它需要放在主队列上，而另一个是我们同步时要使用的，它被放在自己的私有队列上。

We didn’t want to repeat creating managed object contexts inside every test that needs them. Hence, inside our shared TestCase superclass’ -setUp method, we create this set of two managed object contexts. This makes each individual test a lot easier to read.
我们不想在我们需要managed object context的时候，都要重复创建它们。所以我们在共享的`TestCase`父类的`-setUp`方法中加入了创建两个managed object context的集合。这使每个独立的测试用例更易读。

A test that needs a managed object context can simply call self.managedObjectContext or self.syncManagedObjectContext, like so:
一个测试用例需要managed object context时可以见很方便地调用`self.managedObjectContext`或者 `self.syncManagedObjectContext`，就像这样：

- (void)testThatItDoesNotCrashWithInvalidFields
{
    // given
    NSDictionary *payload =     // expected JSON response
    @{
      @"status": @"foo",
      @"from": @"eeeee",
      @"to": @44,
      @"last_update": @[],
    };

    // when
    ZMConnection *connection = [ZMConnection connectionFromTransportData:payload
                                                    managedObjectContext:self.managedObjectContext];

    // then
    XCTAssertNil(connection);
}
We are using NSMainQueueConcurrencyType and NSPrivateQueueConcurrencyType to make the code consistent. But we implemented our own -performGroupedBlock: on top of -performBlock:, in order to solve the isolation problem. More about this under our section about testing asynchronous code.
我们使用`NSMainQueueConcurrencyType`和`NSPrivateQueueConcurrencyType`来保持代码的一致性。但是我们`-performBlock:`之上实现了我们自己的`-performGroupedBlock:`来解决隔离的问题。关于这一点，在下面[关于测试同步代码这节](http://www.objc.io/issue-15/xctest.html#async-testing)中会讲到。

##Merging Multiple Contexts
##合并多个Context

We have two contexts in our code. In production, we rely heavily on being able to merge from one context to another by means of -mergeChangesFromContextDidSaveNotification:. At the same time, we are using a separate persistent store coordinator for each context. Both contexts can then access the same SQLite store with minimal contention.
在我们的代码中有两个context。在生产中，我们依赖于能够通过`-mergeChangesFromContextDidSaveNotification:`来将一个context合并到另一个context。同时，每个context使用一个单独的persistent store coordinator。两个context能以最小的资源冲突来访问SQLite。

But for testing, we had to change this. We want to be able to use an in-memory store.
但是对于测试来说，我们必须改变这一点，我们只想要使用一个内存存储器。

Using an on-disk SQLite store for testing does not work, because there are race conditions associated with deleting the store on disk. This would break isolation between tests. In-memory stores are a lot faster, which is good for testing.
使用磁盘上的SQLite存储器对于测试来说并不管用，因为这里有与从磁盘中删除存储的竞态条件相关。它会打破测试用例之间相互隔离的局面。内存存储器更加快速，这有利于测试。

We use factory methods to create all our NSManagedObjectContext instances. The base test class alters the behavior of these factory methods slightly, so that all contexts share the same NSPersistentStoreCoordinator. At the end of each test, we discard that shared persistent store coordinator to make sure that the next test uses a new one, and a new store.
我们使用工厂方法来创建我们的`NSManagedObjectContext`实例。基础测试类略微地改变了工厂方法的行为么，来实现所有的context能够公用同样的NSPersistentStoreCoordinator。在每个测试的结束时，我们都要抛弃公用的persistent store coordinator来确保下个测试用例能够新的和新的存储器。

##Testing Asynchronous Code
##测试异步代码
Asynchronous code can be tricky. Most testing frameworks offer some basic helpers for asynchronous code.
异步代码是非常狡猾。大多数测试框架都有提供一些对于异步代码的基础辅助工具。

Let’s say we have an asynchronous message on NSString:
假设我们有一个关于NSString的异步消息：

- (void)appendString:(NSString *)other resultHandler:(void(^)(NSString *result))handler;

With XCTest, we could test it like this:
使用XCTest，我们可以这样测试它：

- (void)testThatItAppendsAString;
{
    NSString *s1 = @"Foo";
    XCTestExpectation *expectation = [self expectationWithDescription:@"Handler called"];
    [s1 appendString:@"Bar" resultHandler:^(NSString *result){
        [expectation fulfill];
        XCTAssertEqualObjects(result, @"FooBar");
    }]
    [self waitForExpectationsWithTimeout:0.1 handler::nil];
}
Most testing frameworks have something like this.
大部分的测试框架都是这样。

But the main problem with asynchronous testing is isolation. Isolation is the “I“ in FIRST, as mentioned by the article about testing practices.
但是异步代码的测试的主要问题是隔离。隔离在英语中是Isolation，也就是以“I”为先，这在[这篇关于测试练习的文章](http://www.objc.io/issue-15/bad-testing-practices.html)中被提到过。

With asynchronous code, it can be very tricky to ensure that code from one test has completely stopped running on all threads and queues before the next test starts.
由于测试异步代码很难确定在下一个测试开始之前已经在所有的线程或队列中都已经运行结束了。

We found that the best solution to this problem is to consistently use groups, namely dispatch_group_t.
我发现对于这样的问题的最好解决方法就是一贯地使用组，也就是`dispatch_group_t`。

##Don’t Be Alone, Join a Group
##不要孤单，加入一个组

Some of our classes need to use a dispatch_queue_t internally. Some of our classes enqueue blocks onto an NSManagedObjectContext’s private queue.
我们的一些类中需要在内部使用`dispatch_queue_t`，一些则在`NSManagedObjectContext`的私有队列中排队block。

Inside our -tearDown method, we need to wait for all such asynchronous work to be completed. To achieve that, we had to do multiple things, as shown below.
在我们的`-tearDown`方法中，我们需要等所有的异步工作结束。为了实现这样的方式，我们必须做很多事情，就像下面提到的。

Our test classes have a property:
我们的测试类中有一个property：

@property (nonatomic) dispatch_group_t;

We define this once in our common superclass and set it there.
我们在我们的公共父类中定义并且设置它。

Next, we can inject this group into any class that we’re using that uses a dispatch_queue or similar, e.g. instead of calling dispatch_async(), we’re consistently using dispatch_group_async().
接下来，我们可以将这个组放入到那些使用`dispatch_queue`或类似的类中，比如，我们用`dispatch_group_async()`来替换`dispatch_async()`。

Since we’re relying heavily on Core Data, we added a method to NSManagedObjectContext called
因为我们非常依赖于CoreData，所以我们为`NSManagedObjectContext`的调用增加一个方法。

- (void)performGroupedBlock:(dispatch_block_t)block ZM_NON_NULL(1);
{
    dispatch_group_enter(self.dispatchGroup);
    [self performBlock:^{
        block();
        dispatch_group_leave(self.dispatchGroup);
    }];
}

and added a new dispatchGroup property to all managed object contexts. We then exclusively used -performGroupedBlock: is all our code.
并且为所有managed object contexts添加一个新的property名为dispatchGroup。然后我们在所有的代码中仅仅使用`-performGroupedBlock:`就可以了。

With this, we could wait for all asynchronous work to be done inside our tearDown method:
通过这样的方式，我们可以在我们`tearDown`方法中实现等待所有异步工作结束。

- (void)tearDown
{
    [self waitForGroup];
    [super tearDown];
}

- (void)waitForGroup;
{
    __block BOOL didComplete = NO;
    dispatch_group_notify(self.requestGroup, dispatch_get_main_queue(), ^{
        didComplete = YES;
    });
    NSDate *end = [NSDate dateWithTimeIntervalSinceNow:timeout];
    while (! didComplete) {
        NSTimeInterval const interval = 0.002;
        if (! [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate dateWithTimeIntervalSinceNow:interval]]) {
            [NSThread sleepForTimeInterval:interval];
        }
    }
}
This works, because -tearDown gets called on the main loop. We spun the main loop to make sure that any code that might get enqueued onto the main queue got to run. The above code will hang indefinitely if the group never empties. In our case, we adapted it slightly, so that we had a timeout.
这样是有用的，因为`-tearDown`是在main loop中被调用的。我们在main loop调用它就可以确保一些代码会进入主队列中等待运行。如果这个组永远不空的话，上面这个代码可能会被挂起，在我们情况里，我们稍微调整了一下代码，以确保它有个超时机制。

### Waiting for All Work to Be Done
### 等待所有任务结束

With this in place, a lot of our other tests became a lot easier, too. We created a `WaitForAllGroupsToBeEmpty()` helper, which we use like this:

这个要求具备后，我们很多其他的测试用例也会变得简单一些。我们创建了一个`WaitForAllGroupsToBeEmpty()`辅助方法，我们可以如下使用它：

    - (void)testThatItDoesNotAskForNextRequestIfThereAreNoChangesWithinASave
    {
        // expect
        [[self.transportSession reject] attemptToEnqueueSyncRequestWithGenerator:OCMOCK_ANY];
        [[self.syncStrategy reject] processSaveWithInsertedObjects:OCMOCK_ANY updateObjects:OCMOCK_ANY];
        [self verifyMockLater:self.transportSession];

        // when
        NSError *error;
        XCTAssertTrue([self.testMOC save:&error]);
        WaitForAllGroupsToBeEmpty(0.1);
    }

The last line waits for all asynchronous work to be done, i.e. the test makes sure that even asynchronous blocks enqueuing additional asynchronous work are all done, and that none of them trigger any of the rejected methods.

最后一行代码是等待所有的异步任务都执行完，比如，这个测试用例保证，即使是在异步 blocks 中又入队的其他的异步任务也都被执行完毕了，并且所有的都不会再触发 rejected 相关的方法调用。

We implemented this with a simple macro
我们通过一个简单的宏来实现这种需求

    #define WaitForAllGroupsToBeEmpty(timeout) \
        do { \
            if (! [self waitForGroupToBeEmptyWithTimeout:timeout]) { \
                XCTFail(@"Timed out waiting for groups to empty."); \
            } \
        } while (0)

which, in turn, uses a method on our shared test case superclass:
在其中，又使用了一个我们共享在父类中的方法：

    - (BOOL)waitForGroupToBeEmptyWithTimeout:(NSTimeInterval)timeout;
    {
        NSDate * const end = [[NSDate date] dateByAddingTimeInterval:timeout];

        __block BOOL didComplete = NO;
        dispatch_group_notify(self.requestGroup, dispatch_get_main_queue(), ^{
            didComplete = YES;
        });
        while ((! didComplete) && (0. < [end timeIntervalSinceNow])) {
            if (! [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate dateWithTimeIntervalSinceNow:0.002]]) {
                [NSThread sleepForTimeInterval:0.002];
            }
        }
        return didComplete;
    }


<a name="custom-expectations"> </a>

### Custom Expectations
### 自定义 Expectations

At the [beginning of this section](#async-testing), we mentioned how

        XCTestExpectation *expectation = [self expectationWithDescription:@"Handler called"];

and

        [self waitForExpectationsWithTimeout:0.1 handler::nil];

are some basic building blocks for asynchronous tests.

在[本节的开始](#async-testing)，我们提到了

        XCTestExpectation *expectation = [self expectationWithDescription:@"Handler called"];
        
和

        [self waitForExpectationsWithTimeout:0.1 handler::nil];

是异步测试的一些基本的构建代码块 (building blocks)。


XCTest has some convenience stuff for `NSNotification` and key-value observing, both of which are built on top of these building blocks.
对于`NSNotification`和 key-value observing，XCTest 提供了一些便利的方式，这些都是建立在这些构建代码块的基础上的。

Sometimes, though, we found ourselves using the same patters in multiple places, e.g. if we're asynchronously expecting a managed object context to be saved, we may have code like this:

但是很多时候，我们发现自己会在很多地方使用相同格局的代码，比如，如果我们异步expecting一个受管对象上下文(a managed object context)被保存，我们可能会写出如下代码：

    // expect
    [self expectationForNotification:NSManagedObjectContextDidSaveNotification
                              object:self.syncManagedObjectContext
                             handler:nil];

We simplify this code by having a single, shared method
我们可以独立出一个共享的方法来简化这个代码

    - (XCTestExpectation *)expectationForSaveOfContext:(NSManagedObjectContext *)moc;
    {
        return [self expectationForNotification:NSManagedObjectContextDidSaveNotification
                              object:moc
                             handler:nil];
    }

and then use
然后再在测试用例中使用如下方法调用

    // expect
    [self expectationForSaveOfContext:self.syncManagedObjectContext];

inside our tests. This is easier to read. Along this pattern it is possible to add custom methods for other situations, too.
这更容易阅读。类似于这种模式，也可以给其他情况增加自定义的方法。



<a name="fake-transport-session"> </a>

## The Ol’ Switcheroo—Faking the Transport Layer
## IO 切换器—欺骗传输层

One important question in testing an application is how to test the interaction with the server. The most ideal solution would be to quickly spin up a local copy of the real server, to provision it with fake data, and to run tests directly against it over http.

在测试时，一个很重要的问题就是如何测试与服务器之间的交互。最理想的解决方案是快速的在真实服务器上取一块本地拷贝，给它填充上假数据，然后通过 http 直接针对它运行测试用例。

We are, in fact, working on this solution. It gives us a very realistic test setup. But the sad reality is that it is also a very slow setup. Clearing the database of the server between each tests is slow. We have 1,000 tests. Even if only 30 of them depend on having a real server, if clearing the database and bringing up a 'clean' server instance takes 5 seconds, that would be 2.5 minutes of our test spent on waiting for that to happen. And we also needed to be able to test a server API before that API had been implemented. We needed something else.

实际上，我们就是使用的这种解决方案。它为我们提供了一个非常现实的测试配置。但是一个不好的现实影响是，这种方案是非常慢的。在每次测试之间清理服务器的数据库是非常慢的。我们有1000个测试用例。即使只有其中30个测试用例需要依赖真实的服务器，如果清理数据库，并且提供一个干净的服务器实例需要5秒钟的时间，那么我们的测试过程就需要有2.5分钟的时间是在等待清理工作的完成。我们也需要能够在服务器 API 真正实现之前对其进行测试。我们还需要其他一些东西。

This alternative solution is our 'fake server.' From the get-go, we structured our code so that all our communication with the server is channeled through a single class, the `TransportSession`, which is similar in style to `NSURLSession`, but also handles JSON conversion.

替代的解决方案是‘虚假服务器’。从一开始，我们把所有和服务器交互的代码全部都组织在 `TransportSession`这个类中，这个类类似于`NSURLSession`，但是也处理 JSON 转换。


We have a set of tests that use the API we provide to the UI, and all the interaction with the server is then channeled through a *fake* implementation of the `TransportSession`. This transport session mimics both the behavior of the real `TransportSession` and the behavior of the server. The fake session implements the entire protocol of the `TransportSession` and adds a few methods that allow us to set up its state.

我们有一些列的测试用例是使用我们提供给 UI 层的 API 的，并且所有的这些和服务器的交互都被引导到一个假的实现`TransportSession`中去。这个 transport session 即模仿一个真实的`TransportSession`的行为，也模仿服务器的行为。这个虚假的 session 实现了整个 `TransportSession`协议，并且也提供了一些允许我们改变其状态的方法。

Having a custom class here has several advantages over mocking the server in each test using OCMock. For one, we can create more complex scenarios than what, realistically, would be possible with a mock. We can simulate edge cases that are hard to trigger with a real server.

相比在每个测试用例中使用OCMock来模拟服务器，提供一个自定义的类有很多优势。我们可以创建比使用 mock 更复杂的场景。我们可以模拟一些在真实服务器上很难触发的边缘情况。

Also, the fake server has tests of its own, so its answers are more precisely defined. If we ever need to change the server's reaction to a request, we only have to do so in one place. This makes all the tests that depend on the fake server much more stable, and we can more easily find parts in our code that do not play well with the new behavior.

 并且，这个假服务器也有对其自身的测试用例，所以它的结果是更精确定义的。如果我们想改变服务器的反应到一个请求上去，我们只需要在一个地方改动即可。这使我们所有依赖于假服务器的的测试用例更稳定，并且我们也能更容易的发现我们代码中和新的行为配合不好的地方。

The implementation of our `FakeTransportSession` is simple. An `HTTPRequest` object encapsulates the relative URL, method, and optional payload of a request. The `FakeTransportSession` maps all endpoints to internal methods, which then generate responses. It even has its own in-memory Core Data stack to keep track of the objects it knows about. This way, a GET can return a resource that a previous operation added with a PUT.

`FakeTransportSession`的实现很简单。使用一个`HTTPRequest`对象来封装请求相关的 URL、method 和其他一些可选的参数。`FakeTransportSession` 把所有的请求映射为内部的方法，这些内部方法产生相应的响应。它甚至有一个内存的 Core Data 栈来跟踪相应的对象。使用这种方式，一个 GET 请求可以返回一个之前使用 PUT 请求添加的资源。


All of this may sound like a hard-to-justify time investment. But the fake server is actually quite simple: it is not a real server; we cut a lot of corners. The fake server can only serve a single client, and we do not have to worry about performance / scalability. We also did not implement everything in one huge effort but wrote the parts we needed while developing and testing.

所有的这些听起来需要很多的时间投入。但是，这个假服务器实际上是很简单的：它不是一个真正的服务器；我们削减了大量的细节。这个假服务器只能够为一个客户端提供服务，并且我们也不需要担心性能和扩展性。我们也不需要一次实现所有的功能，我们只需要实现在开发和测试中所需要的功能即可。

One thing worked in our favor, though: the server API was already quite stable and well defined when we started.
尽管这样，有一件事情对我们是有利的：在我们开始做这件事时，我们的服务器 API 已经非常稳定和很好的定义了。

## Custom Assert Macros
## 自定义断言宏

With the Xcode Test framework, one uses XCTAssert macros to do the actual checks:
使用 Xcode Test 框架，人们使用XCTAssert宏来做实际的检查：

    XCTAssertNil(request1);
    XCTAssertNotNil(request2);
    XCTAssertEqualObjects(request2.path, @"/assets");

There's a full list of "Assertions Listed by Category" in Apple’s ["Writing Test Classes and Methods"](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/testing_3_writing_test_classes/testing_3_writing_test_classes.html) article.

在苹果的["编写测试类和方法"](https://developer.apple.com/library/prerelease/ios/documentation/DeveloperTools/Conceptual/testing_with_xcode/testing_3_writing_test_classes/testing_3_writing_test_classes.html)这篇文章里，有一个全面的按照类别排列的断言列表。

But we found ourselves often using very domain-specific checks, such as:
但是我们发现自己经常使用具体的检查，比如：

    XCTAssertTrue([string isKindOfClass:[NSString class]] && ([[NSUUID alloc] initWithUUIDString:string] != nil),
                  @"'%@' is not a valid UUID string", string);

That's very verbose and hard to read. And we didn't like the code duplication. We fixed that by writing our own simple assert macro:
这么写非常的啰嗦，难以阅读。并且我们也不喜欢代码重复。我们通过编写自己的断言宏来解决这个问题：


    #define AssertIsValidUUIDString(a1) \
        do { \
            NSUUID *_u = ([a1 isKindOfClass:[NSString class]] ? [[NSUUID alloc] initWithUUIDString:(a1)] : nil); \
            if (_u == nil) { \
                XCTFail(@"'%@' is not a valid UUID string", a1); \
            } \
        } while (0)

Inside our tests, we then simply use:
在我们的测试用例中，我们只需要简单的如下使用即可：

    AssertIsValidUUIDString(string);

This approach can make a huge difference in making the tests readable.
这种方式让代码更具有可读性。

### One Step Further
###更进一步

But we all know it: [C preprocessor macros](https://en.wikipedia.org/wiki/C_preprocessor#Macro_definition_and_expansion) are beast to dance with.

我们都知道，使用 [C 的预处理宏](https://en.wikipedia.org/wiki/C_preprocessor#Macro_definition_and_expansion) 就是在和野兽跳舞。

For some things, they're unavoidable, and it's all about limiting the pain. We need to use macros in this case in order for the test framework to know on which line and in which file the assertion failed. `XCTFail()` is itself a macro and relies on  `__FILE__` and `__LINE__` to be set.

对于一些事情，它们是无法避免的，只能是如何减少这种痛苦。我们需要在使测试框架知道这个断言是在那个文件的那行代码失败的。`XCTFail()` 本事就是一个宏，而且它还依赖于 `__FILE__` and `__LINE__`。

For more complex asserts and checks, we implemented a simple helper class called `FailureRecorder`:
对于更复杂的断言和检查，我们实现了一个简单的辅助类 `FailureRecorder`：

    @interface FailureRecorder : NSObject

    - (instancetype)initWithTestCase:(XCTestCase *)testCase filePath:(char const *)filePath lineNumber:(NSUInteger)lineNumber;

    @property (nonatomic, readonly) XCTestCase *testCase;
    @property (nonatomic, readonly, copy) NSString *filePath;
    @property (nonatomic, readonly) NSUInteger lineNumber;

    - (void)recordFailure:(NSString *)format, ... NS_FORMAT_FUNCTION(1,2);

    @end


    #define NewFailureRecorder() \
        [[FailureRecorder alloc] initWithTestCase:self filePath:__FILE__ lineNumber:__LINE__]


In our code, we had quite a few places where we wanted to check that two dictionaries are equal to one another. `XCTAssertEqualObjects()` can do that, but when it fails, the output is not very useful.
在我们的代码中，我们有一些地方我们想检查两个字典是不是相等：`XCTAssertEqualObjects()`可以做这样的事情，但是当不相等时，它的输出却不是那么的有意义。

We wanted something like this
我们希望像下面这样使用

    NSDictionary *payload = @{@"a": @2, @"b": @2};
    NSDictionary *expected = @{@"a": @2, @"b": @5};
    AssertEqualDictionaries(payload, expected);

to output
检查失败时，像下面这样输出

    Value for 'b' in 'payload' does not match 'expected'. 2 == 5

So we created
我们创建了一个如下的宏

    #define AssertEqualDictionaries(d1, d2) \
        do { \
            [self assertDictionary:d1 isEqualToDictionary:d2 name1:#d1 name2:#d2 failureRecorder:NewFailureRecorder()]; \
        } while (0)

which forwards into our method
这个宏中调用了如下的方法

    - (void)assertDictionary:(NSDictionary *)d1 isEqualToDictionary:(NSDictionary *)d2 name1:(char const *)name1 name2:(char const *)name2 failureRecorder:(FailureRecorder *)failureRecorder;
    {
        NSSet *keys1 = [NSSet setWithArray:d1.allKeys];
        NSSet *keys2 = [NSSet setWithArray:d2.allKeys];
        if (! [keys1 isEqualToSet:keys2]) {
            XCTFail(@"Keys don't match for %s and %s", name1, name2);
            NSMutableSet *missingKeys = [keys1 mutableCopy];
            [missingKeys minusSet:keys2];
            if (0 < missingKeys.count) {
                [failureRecorder recordFailure:@"%s is missing keys: '%@'",
                 name1, [[missingKeys allObjects] componentsJoinedByString:@"', '"]];
            }
            NSMutableSet *additionalKeys = [keys2 mutableCopy];
            [additionalKeys minusSet:keys1];
            if (0 < additionalKeys.count) {
                [failureRecorder recordFailure:@"%s has additional keys: '%@'",
                 name1, [[additionalKeys allObjects] componentsJoinedByString:@"', '"]];
            }
        }
        for (id key in keys1) {
            if (! [d1[key] isEqual:d2[key]]) {
                [failureRecorder recordFailure:@"Value for '%@' in '%s' does not match '%s'. %@ == %@",
                 key, name1, name2, d1[key], d2[key]];
            }
        }
    }

The trick is that the `FailureRecorder` captures `__FILE__`, `__LINE__`, and the test case. Inside its `-recordFailure:` method, it simply forwards the string to the test case:
这里的技巧是，`FailureRecorder`捕获了 `__FILE__`, `__LINE__`和 test case。在 `-recordFailure:` 方法内部，它简单的把字符串传递给 test case:

    - (void)recordFailure:(NSString *)format, ...;
    {
        va_list ap;
        va_start(ap, format);
        NSString *d = [[NSString alloc] initWithFormat:format arguments:ap];
        va_end(ap);
        [self.testCase recordFailureWithDescription:d inFile:self.filePath atLine:self.lineNumber expected:YES];
    }


<a name="integration-with-xcode"> </a>

## Integration with Xcode and Xcode Server
## 与 Xcode 和 Xcode Server 集成
The best part about XCTest is that it integrates extremely well with the [Xcode IDE](https://developer.apple.com/xcode/). With Xcode 6 and the Xcode 6 Server, this has been improved even more. This tight integration was very helpful and improved our productivity.

XCTest 最好的优点就是它可以和[Xcode IDE](https://developer.apple.com/xcode/)集成的非常好。使用 Xode 6 和 Xcode 6 Server，这方面的优点更被加强了。这种紧密集成是非常有用的，并且能提高我们的效率。

### Focus
###专注

While working on a single test or a set of tests inside a test class, the little diamond on the left-hand side gutter, next to the line numbers, lets us run that specific test or set of tests:
当运行一个单一的测试用例或者在一个测试类中运行一些列测试用例是，在左手边栏上、靠近行数的小菱形，使我们可以运行特定的一个或者一系列测试用例：

![Diamond in Xcode Gutter]({{ site.images_path }}/issue-15/xctest-diamond-in-gutter@2x.png)
![Diamond in Xcode Gutter](http://img.objccn.io/issue-15/xctest-diamond-in-gutter@2x.png)

If the test fails, it turns red:
如果测试失败，它会变成红色：

![Failing Test]({{ site.images_path }}/issue-15/xctest-red-diamon-in-gutter@2x.png)
![Failing Test](http://img.objccn.io/issue-15/xctest-red-diamon-in-gutter@2x.png)


If it passes, it turns green:
如果测试通过，它会变成绿色：

![Passing Test]({{ site.images_path }}/issue-15/xctest-green-diamon-in-gutter@2x.png)
![Passing Test](http://img.objccn.io/issue-15/xctest-green-diamon-in-gutter@2x.png)


One of our favorite keyboard shortcuts is ^⌥⌘G, which runs the last test or set of tests again. After clicking on the diamond in the gutter, we can change the test and then simply rerun it without having to take our hands off the keyboard. When debugging tests, that is extremely helpful.

我们最喜欢的一个键盘快捷键是^⌥⌘G，它可以再一次的运行之前运行的最后一个或者最后一系列测试用例。当点击了边栏上的小菱形后，我们可以改变测试代码，并且简单的再去运行它们而不需要我们的手离开键盘。当调试测试用例时，这是非常有用的。

### Navigator
### 导航

The navigators (on the left-hand side of the Xcode window) have a **Test Navigator**, which shows all tests grouped by their classes:
在 Xcode 左侧的导航栏中有一列**测试导航**，这里是按照所属类分组展示的测试用例：

![Test Navigator]({{ site.images_path }}/issue-15/xctest-test-navigator@2x.png)
![Test Navigator](http://img.objccn.io/issue-15/xctest-test-navigator@2x.png)


Groups of tests and individual tests can also be started from this UI. What's more useful, though, is we can filter this list to only show failing tests by enabling the third little icon at the bottom of the navigator:
也可以从这里开始运行某一个单一的测试用例或者是某一组测试用例。更有用的是，我们可以使用导航栏底部的第三个小图标来过滤所有失败的测试用例。

![Test Navigator with failing tests]({{ site.images_path }}/issue-15/xctest-test-navigator-failing@2x.png)


### Continuous Integration
### 持续集成

OS X Server has a feature called [Xcode Server](https://www.apple.com/osx/server/features/#xcode-server), which is a [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration) server based on Xcode. We have been using this.

OS X Server 有一个叫做 [Xcode Server](https://www.apple.com/osx/server/features/#xcode-server)的特性，它是一个基于 Xcode 的持续集成服务器。我们使用过它。

Our Xcode Server automatically checks out the project from GitHub whenever there's a new commit. We have it configured to run the static analyzer, run all our tests on an iPod touch and a few iOS simulators, and finally create an Xcode archive that can be downloaded.
只要有新的提交，我们的 Xcode Server 就会自动从 Github 上 check out 我们的工程。我们配置它，让他运行静态分析器，在 iPod touch 和 一些 iOS 模拟器上运行所有的测试用例，并且最后自动打包成 Xcode 存档以供下载使用。

With Xcode 6, the feature set of Xcode Server should be rather good, even for complex projects.
在 Xcode 6 中，这些Xcode Server的特性支持的更好了，即使是对复杂的工程。

We have a custom *trigger* that runs as part of the build on Xcode Server on our release branch. This *trigger* script uploads the generated Xcode archive to a file server. This way, we have versioned archives. The UI team can then download a specific version of the precompiled framework from the file server.

我们有一个运行在release分支上的 Xcode Server 上的自定义的“触发器”。这个“触发器”脚本把生成好的 Xcode 存档上传到文件服务器上。这样一来，我们就有了基于版本控制的存档。 UI 小组就可以从文件服务器上下载指定版本的预编译好的框架。

<a name="behaviour-driven-development"> </a>

## BDD and XCTest
## BDD and XCTest

If you are familiar with [behavior-driven development](/issue-15/behavior-driven-development.html), you might have noticed earlier that our style of naming is heavily inspired by this way of testing. Some of us have used [Kiwi](https://github.com/kiwi-bdd/Kiwi) as a testing library before, so it felt natural to concentrate on a method or class behavior. But does this mean that XCTest can replace a good BDD library? The answer is: not quite.
如果你熟悉[行为驱动开发为驱动开发/issue-15/behavior-driven-development.html)，你会发现我们的命名风格在很大程度上受这种测试方式的影响。之前，我们中有些人使用过[Kiwi](https://github.com/kiwi-bdd/Kiwi)作为测试库，所以很自然的集中在一个方法或者一个类的行为上。但是，这是不是意味着 XCTest 可以取代 BDD 库呢？答案是：并不能完全取代。

Both the advantage and the disadvantage of XCTest is that it is very simple. There is not much to it: you create a class, prefix methods that hold tests with the word "test," and you're good to go. The excellent integration into XCode speaks in XCTest's favor as well. You can press the diamond in the gutter to run just one test, you can easily display a list of failed tests, and you can quickly jump to a test by clicking on it in the list of tests.
XCTest 的优势和确定都是由于它太简单了。你只需要创建一个类，使用“test”作为测试函数名的前缀，只需要这样就可以了，不需要再做些什么。和 Xcode 极好的集成性也是 XCTest 获得青睐的原因。你可以点击边栏上的小菱形来运行测试用例，你可以很容易的查看所有失败的测试用例，也可以通过在测试用例列表中点击而快速的跳转到某一个测试用例。

Unfortunately, that is also pretty much all you get. We did not hit any roadblocks while developing and testing our app with XCTest, but often, a bit more comfort would have been nice. XCTest classes look like plain classes, while the structure of a BDD test suite, with its nested contexts, is very visible. And this possibility to create nested contexts for tests is missing most. Nested contexts allow us to create more and more specific scenarios while keeping the individual tests very simple. This is, of course, possible in XCTest too, for example by calling custom setup methods for some of the tests. It is just not as convenient.

不幸的是，这也几乎是所有你能得到的。在开发和测试中，在使用 XCTest 时我们没有碰到任何的障碍，但是很多时候如果能更方便一些会是更好的。XCTest 类看起来就像普通的类，而一个 BDD 测试套件的结构和其嵌套的上下文是很显而易见的。并且这种为测试创建嵌套上下文的可能性也是最缺失的。嵌套的上下文允许我们创建越来越具体的场景，并且使独立的测试相对简单。当然，在 XCTest 中这也是可以的，比如为一些测试用例调用自定义的 setup 方法。只是不那么的方便。

How important the additional features of BDD frameworks are depends on the size of the project. Our conclusion is that XCTest can certainly be a good choice for small- to medium-sized projects. But for larger projects, it might pay off to take a closer look at BDD frameworks like [Kiwi](https://github.com/kiwi-bdd/Kiwi) or [Specta](https://github.com/specta/specta).

使用多么重要的BDD框架的附加功能是取决于项目的大小。我们的结论是，XCTest 对中小型的工程来说是一个很好的选择，但是对于更大型的工程，仔细考察一下像 [Kiwi](https://github.com/kiwi-bdd/Kiwi) or [Specta](https://github.com/specta/specta)这样的 BDD 框架是有必要的。

<a name="the-project"> </a>

##Summary
##总结

Is XCTest the right choice? You will have to judge based on the project at hand. We chose XCTest as part of [KISS](https://en.wikipedia.org/wiki/Keep_it_simple_stupid)—and have a wish list for things we'd like to be different. XCTest has served us well, even though we had to make tradeoffs. With another testing framework, the tradeoff would have been something else.

XCTest 是不是正确的选择呢？你必须根据手头的项目判断。我们选择使用 XCTest 作为[KISS](https://en.wikipedia.org/wiki/Keep_it_simple_stupid)的一部分，对于我们希望能有所改进的地方我们有一个愿望清单。尽管我们不得不做一些取舍，但是 XCTest 对我们来说工作的很好。对于其他的测试框架，这些取舍将会是另外一些事情。