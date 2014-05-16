当斯蒂芬乔布斯在2007第一次介绍iPhone的时候，iPhone的触摸屏交互简直就像是一种魔力。最好的例子就是他[第一次展示滑动TableView的时候](https://www.youtube.com/watch?v=t4OEsI0Sc_s&t=16m9s)。你可以感受到当时观众的反应是多么印象深刻，但是对于我们来说早已习以为常。在展示的后面一部分，he underlined this point by quoting somebody he had given a demo to before: [“You got me at scrolling”](https://www.youtube.com/watch?v=t4OEsI0Sc_s&t=16m9s).

What was it about scrolling that created this ‘wow’ effect?

是什么样的滑动能让有‘哇哦’的效果呢？

Scrolling was a perfect example of direct manipulation through capacitive touch displays. The scroll view obeyed the movements of your finger so closely, and it continued the motion seamlessly after you let go. From there, it decelerated in a natural way, and even exhibited a nice bounce when it hit its boundaries. Scrolling was responsive at any time and behaved just like an object from the real world.

滑动是通过电容触摸显示来直接操作最完美的例子。滚动试图非常忠诚地听命于你的手指，当你放开你的手指时它有能自然地继续滑动直到该停止为止。它用自然的方式减速，甚至在快到界限的时候又表现出细腻的弹力。

##State of Animations
##动画的状态

Most animations in iOS still don’t live up to the standard that scrolling set on the original iPhone. They are fire-and-forget animations, which cannot be interacted with once they’re running (for example the unlock animation, the animations opening and closing groups on the home screen, and the navigation controller animations, to name just a few).

在iOS中的大部分动画仍然没有按照最初iPhone指定的滑动标准执行。这里有很多一旦他们运行就不能交互的动画（比如说解锁动画，主界面中打开文件夹和关闭文件夹的动画，和导航栏切换之间的动画，还有很多）


However, there are some apps out there that bring that aspect of always in control, direct manipulation to all animations they use. It’s a big difference in how these apps feel compared to the rest. Prominent examples of such apps are the original Twitter iPad app and the current Facebook Paper app. But for the time being, apps that fully embrace direct manipulation and always interruptible animations are still rare. This creates an opportunity for apps that do this well, as they have a very different, high-quality feel to them.

然而现在有一些apps给我一种一直在控制的体验，直接操作那些我在用的动画。当我们和其他的apps比较时我们能感觉到很大的区别。这样的app中最优秀的app是最初的Twitter iPad app， 和现在的facebook paper。但是目前，仍然是以直接操作为主而可中断动画仍然较少。这就给我们一个将app做的更好的机会，让它有更不同的，更好质量的体验。

##Challenges of Truly Interactive Animations
##真实交互动画的挑战

Using UIView or CAAnimation animations has two big problems when it comes to interactive animations: those animations separate what you see on the screen from what the actual spacial properties are on the layer, and they directly manipulate the spacial properties.

当我们用UIView 或者 CAAnimation 来实现交互式动画时会有两个大问题: 这些动画会将屏幕上所展示和layer上真实空间内容分隔开来，并且他们直接操作这些真实空间内容。

###Separation of Model and Presentation
###分离模型和表示

Core Animation is designed in a way that it decouples the layer’s model properties from what you see on the screen (the presentation layer). This makes it more difficult to create animations you can interact with at any time, because those two representations do not match. It’s up to you to do the manual work to get them in sync before you change the animation:

Core Animation 是通过分离layer的模型内容和屏幕上的内容(表示层)的方式来实现，这就导致我们很难去创建一个我们可以在任何时候能交互的动画，因为在动画时，内容模型和表示层已经不能匹配了。这时，我们不得不通过手动的方式来同步这两个的状态，来达到改变动画的效果:


    view.layer.center = view.layer.presentationLayer.center;
    [view.layer removeAnimationForKey:@"animation"];
    // add new animation...



###Direct vs. Indirect Control
###直接控制 vs 间接控制

The bigger problem with CAAnimation animations is that they directly operate on the spatial properties of a layer. This means, for example, that you specify that a layer should animate from position (100, 100) to position (300, 300). If you want to stop this animation halfway and to animate the layer back to where it came from, things get very complicated. If you simply remove the current animation and add a new animation, then the layer’s velocity would be discontinuous.

更大的问题是CAAnimation 是直接在空间特性层上操作的。这意味着什么呢？比如我们想指定一个layer从坐标为（100，100）的位置运动到（300，300）的位置，但是我突然在它运动到中间的时候，我们想它停下来并且让它回到它原来的位置，那么这个如果用CAAnimation来实现的话就是非常复杂了。如果你只是简单地删除当前的动画然后再添加一个新的，那么这个layer的速率就会不连续。


![image](http://www.objc.io/images/issue-12/abrupt.png)

What we want to have, though, is a nice, smooth deceleration and acceleration.

我们想要的只不过是一个漂亮的，流畅地减速和加速

![image](http://www.objc.io/images/issue-12/smooth.png)

This only becomes feasible once you start controlling animations indirectly, i.e. through simulated forces acting on the view. The new animation needs to take the layer’s current velocity vector as input in order to produce a smooth result.

只有通过间接操作动画才能达到上面的效果，比如通过模拟力在界面上的表现。新的动画需要用layer的当前速率矢量作为参数传入来达到顺畅的效果。

Looking at the UIView animation API for spring animations (animateWithDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:), you’ll notice that the velocity is a CGFloat. So while you can give the animation an initial velocity in the direction the animation moves the view, you cannot tell the animation that the view is, for example, currently moving at a certain velocity perpendicular to the new animation direction. In order to enable this, the velocity needs to be expressed as a vector.

看一下UIView 中关于spring animations 的动画API （animateWithDuration:delay:usingSpringWithDamping:initialSpringVelocity:options:animations:completion:），有会注意到这个速率是个CGFloat。所以当我们给一个动画运动的方向上加一个初始的速率，这个动画根本不知道这个界面在哪里，就好像我们现在沿着新动画方向的垂直方向移动。为了使这个成为可能，这个速率需要用矩形来表示。

##Solutions
##解决方案

So let’s take a look at how we can correctly implement interactive and interruptible animations. To do this, we’re going to build something like the Control Center panel:

所以让我们看一下我们怎样来实现一个可交互并且可中断的动画。为了实现这个效果，我们将要做一个类似于控制中心板的东西:

<video controls="1" style="display:block;max-width:100%;height:auto;border:0;">
  <source src="/images/issue-12/interactive-animation.mov">
</video>

The panel has two states: opened and closed. You can toggle the states by tapping it, or dragging it up and down. The challenge is to make everything interactive, even while animating. For example, if you tap the panel while it’s animating to the opened state, it should animate back to the closed state from its current position. In a lot of apps that use default animation APIs, you’ll have to wait until the animation is finished before you can do anything. Or, if you don’t have to wait, the animation exhibits a discontinuous velocity curve. We want to work around this.

这个控制板有两个状态：打开和关闭。你可以通过点击来切换这两个状态，或者通过上下拖动来调整音量。我要将每个东西都做的可以交互，甚至是在动画的过程中，这是有挑战性的。比如，当你在这个控制不切换到打开的动画时，你点击了它，那么这个控制板也应该从现在这个点回到关闭状态。在现在很多的apps中，大部分都是用默认的动画api，你必须要等一个动画结束之后你才能做自己想做的事情。或者，你不需要等，但是你会看到一个不连续的速率曲线。我们想要解决这个问题。

###UIKit Dynamics
###UIKit Dynamics

With iOS 7, Apple introduced the animation framework UIKit Dynamics (see WWDC 2013 sessions [206](https://developer.apple.com/videos/wwdc/2013/index.php?id=206) and [221](https://developer.apple.com/videos/wwdc/2013/index.php?id=221)). UIKit Dynamics is based on a pseudo-physics engine that can animate everything that implements the [UIDynamicItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDynamicItem_Protocol/Reference/Reference.html) protocol by adding specific behaviors to an animator object. This framework is very powerful and enables complex behaviors of many items like attachments and collisions. Take a look at the sample dynamics catalog to see what’s available.

随着iOS7的发布，苹果向我们介绍了一个动画框架UIKit Dynamics(可以参见WWDC 2013 sessions [206](https://developer.apple.com/videos/wwdc/2013/index.php?id=206) and [221](https://developer.apple.com/videos/wwdc/2013/index.php?id=221))。UIKit Dynamic是一个基于模拟物理引擎的框架，它能够实现很多动画只要添加指定的行为到你动画对象上来实现[UIDynamicItem](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDynamicItem_Protocol/Reference/Reference.html)协议。这个框架非常强大并且能够将很多物体像附着行为和碰撞行为一样结合起来，让我们看一下简单的[dynamic 目录](https://developer.apple.com/library/ios/samplecode/DynamicsCatalog/Introduction/Intro.html)，看看有什么我们可以得到启发的。

Since animations with UIKit Dynamics are driven indirectly, as we discussed above, this enables us to implement truly interactive animations that can be interrupted and that exhibit continuous acceleration behavior at any time. At the same time, the abstraction of UIKit Dynamics at the physics level can also seem overwhelming for the kind of animations that we generally need in user interfaces. In most cases, we’ll only use a very small subset of its capabilities.

因为UIKit Dynamic的动画是被间接驱动的，就想上面提到的，这就为我们实现真实的交互动画成为可能，它能在任何时候被中断并且可以展示连续加速的动画。同时，UIKit Dynamic在物理层的抽象是能完全胜任我们一般情况下在用户界面上的所需要的动画。在大部分情况下，我们只会用到其中的一部分功能。

####Defining Behaviors
####定义行为

In order to implement our sliding-panel behavior, we’ll make use of two different behaviors that come with UIKit Dynamics: UIAttachmentBehavior and UIDynamicItemBehavior. The attachment behavior fulfills the role of a spring, pulling our view toward its target point. The dynamic item behavior, on the other hand, defines intrinsic properties of the view, such as its friction coefficient.

为了实现我们的滑动板行为，我们将使用UIkit Dynamic的两个不同的行为:[UIAttachmentBehavior](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAttachmentBehavior_Class/Reference/Reference.html) 和 [UIDynamicItemBehavior](https://developer.apple.com/library/ios/documentation/uikit/reference/UIDynamicItemBehavior_Class/Reference/Reference.html)。这个连接行为用来实现弹簧行为，滑动我们的界面朝向它的目标点。在另一方面，这个动态item behvaior定义了这个view的本质属性，比如它的摩擦系数。

To package these two behaviors for our sliding panel, we’ll create our own behavior subclass:

我创建了一个行为子类，将这两个行为封装到我们的滑动板上:

    @interface PaneBehavior : UIDynamicBehavior

    @property (nonatomic) CGPoint targetPoint;
    @property (nonatomic) CGPoint velocity;

    - (instancetype)initWithItem:(id <UIDynamicItem>)item;

    @end

We initialize this behavior with one dynamic item and then can set its target point and velocity to whatever we want. Internally, we create the attachment behavior and the dynamic item behavior and add both as child behavior to our custom behavior:

我们通过一个dynamic item 来初始化这个行为，然后就可以设置他的目标点和我们想要的任何速率。再深入的话，我们创建了连接行为和dynamic item 行为，并且将这些行为添加到我们自定义的行为中:

    - (void)setup
    {
        UIAttachmentBehavior *attachmentBehavior = [[UIAttachmentBehavior alloc] initWithItem:self.item attachedToAnchor:CGPointZero];
        attachmentBehavior.frequency = 3.5;
        attachmentBehavior.damping = .4;
        attachmentBehavior.length = 0;
        [self addChildBehavior:attachmentBehavior];
        self.attachmentBehavior = attachmentBehavior;
        
        UIDynamicItemBehavior *itemBehavior = [[UIDynamicItemBehavior alloc] initWithItems:@[self.item]];
        itemBehavior.density = 100;
        itemBehavior.resistance = 10;
        [self addChildBehavior:itemBehavior];
        self.itemBehavior = itemBehavior;
    }

In order to make the targetPoint and velocity properties affect the item’s behavior, we overwrite their setters and modify the corresponding properties on the attachment and item behaviors, respectively. For the target point, this is very simple:

为了实现用`targetPoint` 和 `velocity`来影响这些item’的行为，我们需要重写他们的setters方法，并且在连接行为上修改到对应的property和还有item behaviors。我们对目标点的修改非常简单:

    - (void)setTargetPoint:(CGPoint)targetPoint
    {
        _targetPoint = targetPoint;
        self.attachmentBehavior.anchorPoint = targetPoint;
    }

For the velocity property, we have to jump through one more hoop, since the dynamic item behavior only allows relative changes in velocity. That means that in order to set the velocity to an absolute value, we first have to get its current velocity and then add the difference to the target velocity:

对于`velocity`这个property，我们需要多做一些工作，因为dynamic item behavior 只允许相对速度的改变。这就意味这为了设置这个`velocity`为绝对值，我们首先需要得到当前的速度，然后在添加差值来得到我们的目标速度。

    - (void)setVelocity:(CGPoint)velocity
    {
        _velocity = velocity;
        CGPoint currentVelocity = [self.itemBehavior linearVelocityForItem:self.item];
        CGPoint velocityDelta = CGPointMake(velocity.x - currentVelocity.x, velocity.y - currentVelocity.y);
        [self.itemBehavior addLinearVelocity:velocityDelta forItem:self.item];
    }

###Putting the Behavior to Use
###使用行为

Our sliding panel has three different states: it is at rest in one of its end positions, being dragged by the user, or animating without the user’s interaction toward one of its end points.

我们的滑动板有三个不同状态：在它的结束位置上它是静止的状态，被用户拖动的状态和在没有用户交互时运动到结束位置的动画状态。

At the transition from the direct manipulation state (the user dragging the panel) to the animation state, we have to do some extra work to make sure that the panel exhibits a smooth animation behavior. When the user stops dragging the panel, it sends a message to its delegate. Within this method, we decide toward what position the panel should animate and add our custom PaneBehavior with this endpoint and – very important – the initial velocity, in order to ensure a smooth transition from dragging to animation:

从直接操作状态（用户拖动这个滑动板）过渡到动画状态，我们还有很多其他的事要做才能将这个板的交互能顺畅地展示动画行为。但用于停止拖动这个板时，它会发送一个消息到它的delegate。根据这个方法，我们可以知道这个板应该朝哪个方向运动，然后在结束位置添加我们自定义的`PaneBehavior`，然后非常重要的是，为了确保从拖动到动画这个过程能够非常流畅，我们需要给它一个初始速度。

    - (void)draggableView:(DraggableView *)view draggingEndedWithVelocity:(CGPoint)velocity
    {
        PaneState targetState = velocity.y >= 0 ? PaneStateClosed : PaneStateOpen;
        [self animatePaneToState:targetState initialVelocity:velocity];
    }

    - (void)animatePaneToState:(PaneState)targetState initialVelocity:(CGPoint)velocity
    {
        if (!self.paneBehavior) {
            PaneBehavior *behavior = [[PaneBehavior alloc] initWithItem:self.pane];
            self.paneBehavior = behavior;
        }
        self.paneBehavior.targetPoint = [self targetPointForState:targetState];
        if (!CGPointEqualToPoint(velocity, CGPointZero)) {
            self.paneBehavior.velocity = velocity;
        }
        [self.animator addBehavior:self.paneBehavior];
        self.paneState = targetState;
    }

As soon as the user puts his or her finger down on the panel again, we have to remove the dynamic behavior from the animator, in order to not interfere with the pan gesture:

一旦用户用他的手指再次放到这滑动板上时，我必须要将所有的dynamic behavior从animator删除，这样这滑动板才能响应拖动手势:


    - (void)draggableViewBeganDragging:(DraggableView *)view
    {
        [self.animator removeAllBehaviors];
    }

We not only allow the panel to be dragged, but it can also be tapped to toggle from one position to the other. When a tap happens, we immediately adjust the panel’s target position. Since we don’t control the animation directly, but via spring and friction forces, the animation will proceed smoothly without abruptly reversing its movement:

我不仅仅允许这个滑动板被拖动，我们还允许它可以被点击，让它从一个位置跳转到另一个位置来达到开关的效果。一旦点击事件发生，我们就会立即调整这个滑动板的目标位置。因为我们不能直接控制动画，但是通过弹力和摩擦力，我们的动画会非常流畅地不突然的执行这个移动：

    - (void)didTap:(UITapGestureRecognizer *)tapRecognizer
    {
        PaneState targetState = self.paneState == PaneStateOpen ? PaneStateClosed : PaneStateOpen;
        [self animatePaneToState:targetState initialVelocity:CGPointZero];
    }

And that’s pretty much all there is to it. You can check out the whole example project on GitHub.

这样该完成的功能都已经实现了。你可以在[GitHub](https://github.com/objcio/issue-12-interactive-animations-uidynamics)上查看完整的例子。

To reiterate the crucial point: UIKit Dynamics allows us to drive the animation indirectly by simulating forces on the view (in our case, spring and friction forces). This indirection enables us to interact with the view at any time while maintaining a continuous velocity curve.

重申一点：UIKit Dynamics 可以通过在界面上模拟力来间接地驱动动画（我们的例子中，使用的是弹力和摩擦力）。这就使我们与界面互动成为可能，并且保持了动画的连续速度曲线。

Now that we have implemented this interaction with UIKit Dynamics, we’ll take a look behind the scenes. Animations like the one in our example only use a tiny fraction of UIKit Dynamic’s capabilities, and it’s surprisingly simple to implement them yourself. That’s a good exercise to understand what’s going on, but it can also be necessary if you either don’t have UIKit Dynamics available (e.g. on the Mac) or it’s not a good abstraction for your use case.

现在我们已经通过UIKit Dynamic来实现了交互，让我们看一下这个场景的背后。这个例子的动画中我们只是用了UIkit Dynamic中的一小部分功能，并且它的实现方式非常地简单。对于我们来说这是一个非常好的例子去理解发生了什么，但是如果我们使用的环境中没有UIKit Dynamic（比如说在Mac上）或者你的情况中不能很好的适用UIkit Dynamic。

##Driving Animations Yourself
##自己操作动画

As for the animations you’ll use most of the time in your apps, e.g. simple spring animations, it’s surprisingly not difficult to drive those yourself. It’s a good exercise to lift the lid of the huge black box of UIKit Dynamics and to see what it takes to implement simple interactive animations ‘manually.’ The idea is rather easy: we make sure to change the view’s frame 60 times per second. For each frame, we adjust the view’s frame based on the current velocity and the forces acting on the view.

关于在你的应用中大部分时间会用的动画，比如简单的弹力动画，我们控制它真的不难。举起一个使用UIKit Dynamic的巨大黑色箱子，然后看它是如何实现简单的手动交互的来做一个练习。这个想法非常简单：我们保证修改这个界面的frame60次每秒。每次frame的变化时，我们都要基于当前的速率和当前作用在界面上的力来调整这个界面的frame。

###The Physics
###物理原理

Let’s first take a look at some basic physics necessary to drive a spring animation like we created before using UIKit Dynamics. To simplify things, we’ll look at a purely one-dimensional case (as it is the case in our example), although introducing the second dimension is straightforward.

首先让我们看一些需要的基础物理知识，这样我们才能实现想刚才使用UIKit Dynamic创建的弹力动画效果。特别指出的是，我们将要先看纯一维世界（在我们的例子中就是这样的情况）因为直接介绍第二维太直白了。

The objective is to calculate the new position of the panel based on its current position and the time that has elapsed since the last animation tick. This can be expressed as:

这个对象是用来计算滑动板的新位置，而新位置是基于滑动板的当前位置和从上一次动画开始到现在的时间来决定的。我们可以把它表达成这样：

    y = y0 + Δy

The position delta is a function of the velocity and the time:

这个位置的偏移量可以通过速率和时间的函数来表达：

    Δy = v ⋅ Δt
The velocity can be calculated as the previous velocity plus the velocity delta, caused by the force acting on the view:

这个速率可以通过前一次的速率加上速率偏移量算出来，这个速率是由力在界面上的作用引起的。

    v = v0 + Δv
The change in velocity can be calculated by the impulse applied to the view:

速率的变化可以通过作用在这个界面上的推力计算出来：

    Δv = (F ⋅ Δt) / m
Now, let’s take a look at the force acting on the view. In order to get the spring effect, we have to combine a spring force with friction force:

现在，让我们看一下作用在这个界面上的力。为了得到弹力效果，我们必须要将摩擦力和弹力结合：

    F = F_spring + F_friction
The spring force comes straight from the textbook:

弹力的计算方法我们可以从任何一本教科书中得到：

    F_spring = k ⋅ x
where k is the spring constant and x is the distance of the view to its target end point (the length of the spring). Therefore, we can also write this as:

k是弹力系数，x是到目标结束位置的距离（也就是弹力的长度）。因此，我们可以把它写成这样：

    F_spring = k ⋅ abs(y_target - y0)
We calculate friction as being proportional to the view’s velocity:

摩擦力是界面速率的正比：

    F_friction = μ ⋅ v
μ is a simple friction constant. You could come up with other ways to calculate the friction force, but this works well to create the animation we want to have.

μ是一个简单的摩擦系数。你可以通过别的方式来计算摩擦力，但是这个方法能很好地做出我们想要的动画效果。

Putting this together, the force on the view is calculated as:

将上面的方法放在一起，我们就可以算出作用在界面上的力：

    F = k ⋅ abs(y_target - y0) + μ ⋅ v
To simplify things a bit more, we’ll set the view’s mass to 1, so that we can calculate the change in position as:

为了实现起来简单点，我们将view的质量设置为1，这样我们就能计算在位置上的变化：

    Δy = (v0 + (k ⋅ abs(y_target - y0) + μ ⋅ v) ⋅ Δt) ⋅ Δt

###Implementing the Animation
###实现动画

To implement this, we first create our own Animator class, which drives the animations. This class uses a CADisplayLink, which is a timer made specifically for drawing synchronously with the display’s refresh rate. In other words, if your animation is smooth, the timer calls your methods 60 times per second. Next, we implement a protocol Animation that works together with our Animator. This protocol has only one method, animationTick:finished:. This method gets called every time the screen is updated, and gets two parameters: the first parameter is the duration of the previous frame, while the second parameter is a pointer to a BOOL. By setting the value of the pointer to YES, we can communicate back to the Animator that we’re done animating:

为了实现动画，我们首先需要创建我们自己的`Animator`类，它将扮演驱动动画的功能。这个类使用了`CADisplayLink`，`CADisplayLink`是指定在渲染时和界面的刷新频率同步的定时器。换句话说，如果你的动画是流畅的，这个定时器就会每秒60次地调用你的方法。接下来，我们需要实现`Animation`协议来和我们的`Animator`一起工作。这个协议只有一个方法，`animationTick:finished:`。屏幕每次被刷洗时都会调用这个方法，并且我们会得到两个参数：第一个参数是前一个frame的持续时间，第二个参数是一个bool值，我们通过设置这个bool值为YES时，我们就可以又回到我们刚才已经结束动画的`Animator`:

    @protocol Animation <NSObject>
    - (void)animationTick:(CFTimeInterval)dt finished:(BOOL *)finished;
    @end

The method is implemented below. First, based on the time interval, we calculate a force, which is a combination of the spring force and the friction force. Then we update the velocity with this force, and adjust the view’s center accordingly. Finally, if the speed gets low and the view is at its goal, we stop the animation:

这个方法会在下面被实现。首先，根据时间间隔我们来计算由弹力和摩擦力结合的力。然后根据速率来更新这个力，再调整界面的中心位置。最后，如果这个界面的开始减速并且即将到达结束位置时，我们就停止这个动画：

    - (void)animationTick:(CFTimeInterval)dt finished:(BOOL *)finished
    {
        static const float frictionConstant = 20;
        static const float springConstant = 300;
        CGFloat time = (CGFloat) dt;

        // friction force = velocity * friction constant
        CGPoint frictionForce = CGPointMultiply(self.velocity, frictionConstant);
        // spring force = (target point - current position) * spring constant
        CGPoint springForce = CGPointMultiply(CGPointSubtract(self.targetPoint, self.view.center), springConstant);
        // force = spring force - friction force
        CGPoint force = CGPointSubtract(springForce, frictionForce);

        // velocity = current velocity + force * time / mass
        self.velocity = CGPointAdd(self.velocity, CGPointMultiply(force, time));
        // position = current position + velocity * time
        self.view.center = CGPointAdd(self.view.center, CGPointMultiply(self.velocity, time));

        CGFloat speed = CGPointLength(self.velocity);
        CGFloat distanceToGoal = CGPointLength(CGPointSubtract(self.targetPoint, self.view.center));
        if (speed < 0.05 && distanceToGoal < 1) {
            self.view.center = self.targetPoint;
            *finished = YES;
        }
    }

That’s all there is to it. We capsulated this method in a SpringAnimation object. The only other method in this object is the initializer, which takes the view to animate, the target point for the view’s center (in our case, it’s either the center point for the opened state, or the closed state), and the initial velocity.

这就是这个方法里的全部内容。我们把这个方法封装到一个SpringAnimation对象中。这个类中其他的方法还有是界面动画的初始化方法，这个界面中心的目标位置（在我们的例子中，就是打开状态的中心位置，或者关闭状态的中心位置）和初始的速率。

###Adding the Animation to the View
###将动画添加到界面上

Our view class is exactly the same as in the UIDynamics example: it has a pan recognizer and updates its center based on the pan gestures. It sends out the same two delegate methods, which we will implement to initialize our animation. First of all, when the user starts dragging, we cancel all animations:

我们的界面类刚好和UIDynamic例子中的一样：它有个拖动手势和根据拖动手势来更新中心位置。它也有两个同样的delegate 方法，这两个方法会实现我们的初始化。首先，一旦用户开始拖动时，我们就停止所有动画：

    - (void)draggableViewBeganDragging:(DraggableView *)view
    {
        [self cancelSpringAnimation];
    }

After the dragging ends, we start our animation with the last velocity value from the pan gesture. The target point is calculated from the paneState:

一旦这个拖动结束，我们就根据从拖动手势中的得到的最后一个速率值来开始我们的动画。动画的结束位置是从拖动状态中的计算出来的：

    - (void)draggableView:(DraggableView *)view draggingEndedWithVelocity:(CGPoint)velocity
    {
        PaneState targetState = velocity.y >= 0 ? PaneStateClosed : PaneStateOpen;
        self.paneState = targetState;
        [self startAnimatingView:view initialVelocity:velocity];
    }

    - (void)startAnimatingView:(DraggableView *)view initialVelocity:(CGPoint)velocity
    {
        [self cancelSpringAnimation];
        self.springAnimation = [UINTSpringAnimation animationWithView:view target:self.targetPoint velocity:velocity];
        [view.animator addAnimation:self.springAnimation];
    }

The only thing left to do is add the tap animation and that is relatively easy. We toggle the state and start animating. If there is a spring animation, we start with that velocity. If the spring animation is nil, the initial velocity will be CGPointZero. To understand why it still animates, look at the animationTick:finished: code. When the initial velocity is zero, the spring force will slowly keep increasing the velocity until the pane arrives at the target center point:

剩下来要做的只有添加点击动画了，这很简单。我们触发这个状态，并且开始动画。如果这里有个弹力动画，我们就用速率来启动它。如果这个弹力动画是nil，那么这个开始速率就是CGPointZero。想要知道为什么这个动画依然会有，就去看`animationTick:finished:`里的代码。当这个起始速率为0的时候，弹力就会缓慢地保持速率的增长直到拖动到目的中心位置：

    - (void)didTap:(UITapGestureRecognizer *)tapRecognizer
    {
        PaneState targetState = self.paneState == PaneStateOpen ? PaneStateClosed : PaneStateOpen;
        self.paneState = targetState;
        [self startAnimatingView:self.pane initialVelocity:self.springAnimation.velocity];
    }

###The Animation Driver
###动画驱动者

Finally, the last part we need is the Animator, which is the driver of the animations. The animator is a wrapper around the display link. Because each display link is coupled to a specific UIScreen, we initialize our animator with a specific screen. We set up a display link, and add it to the run loop. Because there are no animations yet, we start in a paused state:

最后，我们需要一个Animator，也就是动画的驱动者。Animator是displayLink的封装者。因为每个displayLink都是链接一个指定的`UIScreen`。我们根据这个指定的UIScreen来初始化我们的Animator。我们初始化一个DisplayLink，并且将他加入到RunLoop。因为现在还没有动画，我们是停止状态开始的：

    - (instancetype)initWithScreen:(UIScreen *)screen
    {
        self = [super init];
        if (self) {
            self.displayLink = [screen displayLinkWithTarget:self selector:@selector(animationTick:)];
            self.displayLink.paused = YES;
            [self.displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
            self.animations = [NSMutableSet new];
        }
        return self;
    }

Once we add the animation, we make sure that the display link is not paused anymore:

一旦我们添加了这个动画，我们就肯定这个DisplayLink已经不再是停止状态了：

    - (void)addAnimation:(id<Animation>)animation
    {
        [self.animations addObject:animation];
        if (self.animations.count == 1) {
            self.displayLink.paused = NO;
        }
    }

We set up the display link to call animationTick:, and on each tick we iterate over the animations, send them a message, and that’s it. If there are no animations left, we pause the display link:

我们创建了这个DisplayLink来调用`animationTick:`方法，在每个Tick中，我们都迭代它的动画，并且发送一个消息。如果这里已经没有动画了，我们就停止这个DisplayLink。

     - (void)animationTick:(CADisplayLink *)displayLink
     {
         CFTimeInterval dt = displayLink.duration;
         for (id<Animation> a in [self.animations copy]) {
             BOOL finished = NO;
             [a animationTick:dt finished:&finished];
             if (finished) {
                 [self.animations removeObject:a];
             }
         }
         if (self.animations.count == 0) {
             self.displayLink.paused = YES;
         }
     }

The entire project is available on GitHub.

完整的项目在GitHub中。

###Tradeoffs
###平衡

It’s important to keep in mind that driving animations via display links (as demonstrated above or by using UIKit Dynamics or something like Facebook’s POP framework) comes with a tradeoff. As Andy Matuschak pointed out UIView and CAAnimation animations are less likely to be affected by other tasks running on the system, because the render server runs at a higher priority than your app.

记住我们是通过DisplayLink来驱动动画这一点非常重要（就像我们刚才演示的，或者我们使用UIkit Dynamic来做的例子，或者像Facebook的Pop框架）都是经过平衡的过程。就像[Andy Matuschar指出](https://twitter.com/andy_matuschak/status/464790108072206337)UIView和CAAnimation动画和其他任务相比，更少受系统的影响，因为渲染在你的应用处于更高的优先级。

##Back to the Mac
##回到Mac

There’s nothing like UIKit Dynamics available on Mac at this time. If you want to create truly interactive animations here, you have to take the route of driving those animations yourself. Now that we’ve already shown how to implement this on iOS, it’s very simple to make the same example work on OS X; check out the full project on GitHub. These are the things that need to be changed:

现在在Mac中还没有UIKit Dynamic。如果你想在Mac中创建一个真实的交互动画，你必须自己去实现这些动画。我们已经想你展示如何在iOS中实现这些动画，所以在OS X中实现相似的功能也是非常简单的。你可以查看在GitHub中的[完整项目](https://github.com/objcio/issue-12-interactive-animations-osx)，如果你想要应用到OS X中，这里还有一些地方需要修改：

- The first thing to change is the Animator. On the Mac, there is no CADisplayLink, but instead, a CVDisplayLink, which has a C-based API. Setting it up is a bit more work, but just as straightforward.

- 第一个要修改的就是Animator。在Mac中没有`CADisplayLink`，但是取而代之的有`CVDisplayLink`，它是以C语言为基础的API。创建它需要做更多的工作，但也是很简单的。

- Our spring animation on iOS adjusts the center of the view. An NSView doesn’t have a center property, so instead we animate the frame’s origin.

- iOS中的弹力动画是基于调整界面的中心位置来实现的。而OS X中的NSView类没有中心点这个property，所以我们用fram中的origin来代替。

- On the Mac, there are no gesture recognizers. Instead, we have to implement mouseDown:, mouseUp:, and mouseDragged: in our custom view subclass.

- 在Mac中是没有手势的，所以我要在我们自定义的View子类中实现`mouseDown:`, `mouseUp:`, 和 `mouseDragged`


These are the only changes we need to make to port our animation code to the Mac. For a simple view like this, it works really well. For more complex things, you might not want to animate the frame, but use transform instead, which is the topic of a blogpost on OS X Animations by Jonathan Willing.

上面就是我们需要在Mac中使用我们的动画效果的代码所需要做的改变。对于想这样的简单界面，它能很好的胜任。但对于更复杂的动画，你可以不能想通过改变界面的frame来实现了，你可以用transform来代替，你可以阅读一下Jonathan Willing写的关于[OS X动画](http://jwilling.com/osx-animations)的以前博客。

###Facebook’s POP Framework
###Facebook的pop框架

There has been quite a bit of buzz in the last weeks around Facebook’s POP framework. This is the animation engine that powers its Paper app. It operates very similar to the example above of driving your own animations, but it comes in a neat package with a lot more flexibility.

上个星期围绕着Facebook的[Pop框架](https://github.com/facebook/pop)有很多的议论。这是Paper应用背后的动画引擎。它的操作非常像我们上面提到的驱动自己的动画的例子，但是它有一个非常灵活整洁的包。

So, let’s try to make our own manually driven animation work with POP instead. Since we already had our own spring animation packaged into its own class, the change is pretty trivial. All we have to do is instantiate a POP animation instead of our own one, and add this to the view:

所以让我们动手用Pop来驱动我们的动画。因为我们已经在我们自己的类中已经封装了我们的弹力动画了，这些改变和Pop相比已经非常微不足道了。我们所要做的就是做一个POP动画的示例来代替我们刚才自己做的例子，将下面这段代码加入到界面类中：



    - (void)animatePaneWithInitialVelocity:(CGPoint)initialVelocity
    {
        [self.pane pop_removeAllAnimations];
        POPSpringAnimation *animation = [POPSpringAnimation animationWithPropertyNamed:kPOPViewCenter];
        animation.velocity = [NSValue valueWithCGPoint:initialVelocity];
        animation.toValue = [NSValue valueWithCGPoint:self.targetPoint];
        animation.springSpeed = 15;
        animation.springBounciness = 6;
        [self.pane pop_addAnimation:animation forKey:@"animation"];
        self.animation = animation;
    }

You can find the full working example using POP on GitHub.

你可以在[GitHub](https://github.com/objcio/issue-12-interactive-animations-pop)中找到使用Pop的完整例子。

It’s super easy to get it to work, and it’s pretty straightforward to create more complex animations. But the real power of it lies in the fact that it enables you to create truly interactive and interruptible animations, as we have talked about before, because the animations it supports out of the box take the velocity as input. If you plan your interactions from the get-go to be interruptible at any time, a framework like POP helps you to implement this in a way that ensures animations always stay smooth.

使用它非常简单，并且通过它我们可以实现很多更复杂的动画。但是它真正强大的地方在于它能够实现真正的可交互和可中断的东湖，就想我们上面提到的那样，因为它支持以速率作为参数来满足一定的需求。如果你打算从一开始到被中断的任何时候都能交互，像POP这样的框架就能帮你实现这些动画，并且始终保持流畅。

If you need more than what POPSpringAnimation and POPDecayAnimation can do out of the box, POP also comes with a POPCustomAnimation class, which basically is a convenient wraparound display link to drive your own animation in a callback block that gets called on each animation tick.

如果你不满足于`POPSpringAnimation`和`POPDecayAnimation`来处理的话，POP来提供了`POPCustomAnimation`类，它是在每次动画Tick时会调用一个回掉block来实现封装的DisplayLink驱动我们的动画的转变。

##The Road Ahead
##展望未来

With iOS 7’s shift away from visual imitation of real-world objects toward a stronger focus on the UI’s behavior, truly interactive animations are a great way to stand out. They’re also a way to extend the magic of the original iPhone’s scrolling behavior into every aspect of the interaction. To make this work, it’s important to consider those interactions early on in the design instead of just bolting on animations late in the development process.

随着iOS7中从对真实世界对象拟物表现到如今更加关注于交互行为，真实的交互动画前方的道路变得越来越明显。这还有一个方法来扩大最初iPhone的滑动行为的魔力到交互的每个方面。为了让这些魔力实现，在设计时就要考虑这些交互是非常重要的，而不是在开发时才想到这些动画。

A special thanks goes to Loren Brichter for his advice on this article!

非常感谢[Loren Brichter](https://twitter.com/lorenb)给这篇文章的一些意见。


More articles in issue #12
