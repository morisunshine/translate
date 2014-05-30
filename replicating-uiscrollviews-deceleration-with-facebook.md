Replicating UIScrollView’s deceleration with Facebook Pop
使用FaceceBook 的Pop框架替换UIScrollView的减速动画
====

Earlier this week Ole Begemann wrote a [really great tutorial on how UIScrollViews work](http://oleb.net/blog/2014/04/understanding-uiscrollview/), and to explain it effectively, he even created [a really simple scroll view, written from scratch.](https://github.com/ole/CustomScrollView)

这个星期的早些时候 Ole Begemann 写了一篇[UIScrollView 是如何工作的很棒的教程](http://oleb.net/blog/2014/04/understanding-uiscrollview/)，为了更有效地描述，他甚至还[自己从头开始写了一个简单的ScrollView。](https://github.com/ole/CustomScrollView)

The setup was quite simple: Use a UIPanGestureRecognizer, and change the bounds’ origin in response to translation of the pan gesture.


Extending Ole’s custom scroll view to incorporate UIScrollView’s inertial scrolling seemed like a natural extension, and with Facebook’s recent release of Pop, I thought it writing a decelerating custom scroll view would be an interesting weekend project.

In case you’re unaware, [Pop is Facebook’s recently open-sourced animation framework](http://iosdevtips.co/post/84160910513/exploring-facebook-pop) that powers the animations in Paper for iOS. The Pop API looks mostly like CoreAnimation, so it isn’t that difficult to learn.

Two of the best things about Pop:

1. It gives you spring and decay animations out of the box
2. You can animate any property on any NSObject using Pop, not just the ones that are declared animatable in UIKit
Given that Pop has built-in support for decaying animations, implementing UIScrollView’s deceleration isn’t very tough.

The main idea is to animate the bounds’ origin in a decaying fashion using the velocity off the UIPanGestureRecognizer in its ended state.

So let’s begin. Start off by forking [my fork of Ole’s CustomScrollView project,](https://github.com/rounak/CustomScrollView) jump to the handlePanGesture: method in CustomScrollView.m. (Why start with my fork and not Ole’s project? He sets the translation to zero each time the gesture fires, which in turn resets the velocity. We, however, want the velocity to be preserved.)

We want to start a decay animation only after the gesture has finished, so we’re interested in the UIGestureRecognizerStateEnded case of the switch statement.

Here’s the basic flow of the code fragment embedded below:

- We get the velocity of the gesture with the velocityInView: method.

	CGPoint velocity = [panGestureRecognizer velocityInView:self];

- We do not want any movement in x or y direction if there’s not enough horizontal or vertical content respectively. So we add checks for that.

	if (self.bounds.size.width >= self.contentSize.width) {
		    velocity.x = 0;
	}
	if (self.bounds.size.height >= self.contentSize.height) {
		    velocity.y = 0;
	}

- Also it turns out that the velocity we get from the velocityInView: method is the negative of what we actually want, so fix that.

- We want to animate the bounds (specifically the bounds’ origin) so create a new POPDecayAnimation with the kPOPViewBounds property.

	POPDecayAnimation *decayAnimation = [POPDecayAnimation animationWithPropertyNamed:kPOPViewBounds];

- Pop expects the velocity to be of the same coordinate space as the property you’re trying to animate. The value will be an NSNumber for alpha (CGFloat), CGPoint wrapped in NSValue for center, CGRect wrapped in NSValue for bounds and so on. As you might have guessed, the change in the property value during the animation is a factor of the corresponding velocity component.

- In our case, animating bounds means our velocity will be a CGRect, and its origin.x and origin.y values correspond to the change in the bounds' origin. Similarly, the size.width and size.height velocity components correspond to the change in the bounds’ size.

- We don’t want to change the size of the bounds, so lets keep that velocity component zero always. Feed in the origin.x and origin.y values of the velocity with the pan gesture’s velocity. Wrap the whole thing into an NSValue and assign it to decayAnimation.velocity

	decayAnimation.velocity = [NSValue valueWithCGRect:CGRectMake(velocity.x, velocity.y, 0, 0)];

- Finally, add the decayAnimation to the view using the pop_addAnimation: method with any arbitrary key you want.

	[self pop_addAnimation:decayAnimation forKey:@"decelerate"];

Here’s the combined code:


	//get velocity from pan gesture
	CGPoint velocity = [panGestureRecognizer velocityInView:self];
	if (self.bounds.size.width >= self.contentSize.width) {
		    //make movement zero along x if no horizontal scrolling
		    velocity.x = 0; 
	}
	if (self.bounds.size.height >= self.contentSize.height) {
		    //make movement zero along y if no vertical scrolling
		    velocity.y = 0; 
	}
	 
	//we need the negative velocity of what we get from the pan gesture, so flip the signs
	velocity.x = -velocity.x;
	velocity.y = -velocity.y;
	 
	POPDecayAnimation *decayAnimation = [POPDecayAnimation animationWithPropertyNamed:kPOPViewBounds];
	 
	//last two components zero as we do not want to change bound's size
	decayAnimation.velocity = [NSValue valueWithCGRect:CGRectMake(velocity.x, velocity.y, 0, 0)];
	 
	[self pop_addAnimation:decayAnimation forKey:@"decelerate"];


![image](http://media.tumblr.com/185a108dfa8a706c90985b18198bd39c/tumblr_inline_n4z4hwnQiv1qh9cw7.gif)

With this you should have the basic deceleration working. You’ll notice that if your velocity is high, the view will scroll beyond its contentSize, but this could easily be solved by overriding setBounds: and adding checks to see that the bounds origin do not go beyond the thresholds. I’m surprised that Pop doesn’t add these checks itself.

-----

A really interesting aspect of Pop is the ability to animate any property at all, not just ones declared as animatable by UIKit. Since we’re just animating the bounds’ origin, I thought this was a good chance to try out Pop`s custom property animation.

Pop lets you do this is by giving you constant callbacks with the progress of the animation, so that you can update your view accordingly. It takes care of the hard part of handling timing, converting the timing into the value of your property, decaying or oscillating the animation etc.

So here’s how you’ll do the same thing we did earlier, but now with your own custom property. Specifically, we’ll animate bounds.origin.x and bounds.origin.y. You’ll see that the structure of the code largely remains the same, except for the initialisation of this giant POPAnimatableProperty.

- The property name, as I understand, can be any arbitrary string. In this case, I’ve taken it to be @"com.rounak.bounds.origin" 

- There’s also an initializer block with POPMutableAnimatableProperty as the argument. (I think this initialisation pattern is called the builder pattern.)

- POPMutableAnimatableProperty has two important properties, readBlock and writeBlock. In readBlock you’ll have to feed data to Pop, and writeBlock you’ll have to retrieve that data, and update your view. The readBlock is called once whereas the writeBlock is called on each frame update of the display (via CADisplayLink).

- Pop internally converts everything into vectors, and hence it asks and gives you data in the form of a linear array of CGFloats.

- In readBlock you simply assign bounds.origin.x and bounds.origin.y to values[0] and values[1] respectively.


	prop.readBlock = ^(id obj, CGFloat values[]) {
		    values[0] = [obj bounds].origin.x;
				    values[1] = [obj bounds].origin.y;
	};

- And in writeBlock you read values[0] (bounds.origin.x) and values[1] (bounds.origin.y) and update your view’s bounds.

	prop.writeBlock = ^(id obj, const CGFloat values[]) {
	    CGRect tempBounds = [obj bounds];
	    tempBounds.origin.x = values[0];
	    tempBounds.origin.y = values[1];
	    [obj setBounds:tempBounds];
	};

- The magic happens in the writeBlock which is called each time with values that follow a decay (or an oscillating) curve.

- You’ll notice that since we’re operating in just two dimensions here, our velocity is CGPoint, and not CGRect.

Here’s the combined code:

	//get velocity from pan gesture
	CGPoint velocity = [panGestureRecognizer velocityInView:self];
	if (self.bounds.size.width >= self.contentSize.width) {
	    //make movement zero along x if no horizontal scrolling
	    velocity.x = 0;
	}
	if (self.bounds.size.height >= self.contentSize.height) {
	    //make movement zero along y if no vertical scrolling
	    velocity.y = 0;
	}
	 
	//we need the negative velocity of what we get from the pan gesture, so flip the signs
	velocity.x = -velocity.x;
	velocity.y = -velocity.y;
	 
	POPDecayAnimation *decayAnimation = [POPDecayAnimation animation];
	 
	POPAnimatableProperty *prop = [POPAnimatableProperty propertyWithName:@"com.rounak.boundsY" initializer:^(POPMutableAnimatableProperty *prop) {
	    // read value, feed data to Pop
	    prop.readBlock = ^(id obj, CGFloat values[]) {
	        values[0] = [obj bounds].origin.x;
	        values[1] = [obj bounds].origin.y;
	    };
	    // write value, get data from Pop, and apply it to the view
	    prop.writeBlock = ^(id obj, const CGFloat values[]) {
	        CGRect tempBounds = [obj bounds];
	        tempBounds.origin.x = values[0];
	        tempBounds.origin.y = values[1];
	        [obj setBounds:tempBounds];
	    };
	    // dynamics threshold
	    prop.threshold = 0.01;
	}];
	 
	decayAnimation.property = prop;
	decayAnimation.velocity = [NSValue valueWithCGPoint:velocity];
	[self pop_addAnimation:decayAnimation forKey:@"decelerate"];

The entire project with the deceleration [can be found on GitHub](https://github.com/rounak/CustomScrollView/tree/custom-scroll-with-pop). Have any feedback about this topic? I’m [@r0unak](http://twitter.com/r0unak) on Twitter. Also, try [Design Shots, a Dribbble app I helped make.](https://itunes.apple.com/us/app/design-shots/id792517951?mt=8)