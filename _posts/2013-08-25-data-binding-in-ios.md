---
layout: post
title: "Multiple direction data binding in iOS - ChangeChannel"
description: "An experiment on data binding for iOS"
category: iOS
tags: [iOS,]
---
{% include JB/setup %}

Story begins
===========================

Syncing data is one of the most common senaria in iOS development. 
* Sync data from data model to view
* Update controls whose values are connected with some logic. 

KVO was the solution for one way binding, A ====> B, but not A <=====> B. In my case, even A <======> B <======> C is quite common. 

There is a feature in [*FlexibleProto*](https://itunes.apple.com/cn/app/flexibleproto-design-your/id684174184?mt=8): Move and resize the view:

> User able to change the view's frame by drag it in a natual way. 
> It is efficient but unprecise, (almost) no one can control the drag in 1pt, so there is a property panel, which user can directly modify the value. The data are synced real time between view and panel, e.g, by moving left 10pt in the view, the property panel's x will be -=10. Vise versa.

Since the change can be sourced from view and also property panel, so direct KVO can't help, it will create a KVO loop, changes on A will trigger changes on B and then A B... 

Moves on
==========================
At first, I wrap the change code in one function

{% highlight objective-c %}
// Bad one
-(void) updateFrame:(CGRect)frame{
    view.frame = frame;
    propertyPanel.value = frame;
}
{% endhighlight %}

The code is tightly coupled, in order update the frame, I have to call a function defined else where. If I want to add a third view in the play yard, I have to change this function, it will become a headache when more logic added. After playing 10 minutes PunchQuest, I know what I want: 

{% highlight c %}
// Ideal one
    Create a change bus
    attach view on its frame property
    attach propertyPanel on its value property
    view's frame change will update propertyPanel's value & vise versa
{% endhighlight %}

Getting excited
==================

Here is what I get after some work ( with some naive implementation ):

{% highlight objective-c %}
// What I get
    // create the channel with empty items and nil init value
    ChangeChannel *changeChannel = [[ChangeChannel alloc] initWithItems:@[] value:nil];

    // attach view, property channel
    [changeChannel attachChangeItem:[[ObjectChangeItem alloc] initWithObject:view keyPath:@"frame"]];
    [changeChannel attachChangeItem:[[ObjectChangeItem alloc] initWithObject:propertyPanel keyPath:@"value"]];

    // from now on, the view.frame == propertyPanel
{% endhighlight %}

Also create a block changeItem, which logs out each frame change

{% highlight objective-c %}
    [changeChannel attachChangeItem:[[BlockChangeItem alloc] initWithBlock:^(id newValue, id oldValue){
        NSLog(@"Frame changed from: %@ to %@", newValue, oldValue);
    }]];
    // From now on, every frame change in both view1, propertyPanel will be logged out
{% endhighlight %}

> Block can't fire changes but only accept it.

What if match different type of values

{% highlight objective-c %}
    ObjectChangeItem *doubleSized = [[ObjectChangeItem alloc] initWithObject:view2 keyPath:@"frame"];

    // C means Channel, O means Object, C2OBlock used to transform value from channel, O2CBlock used
    //     to transform value put into channel
    doubleSized.convertor = [[ValueConvertor alloc] initWithC2OBlock: ^(id value){
            CGRect result = [value CGRectValue];
            result.size.width *= 2;
            result.size.height *= 2;
            return [NSValue valueWithCGRect:result];
        } o2cBlock: ^(id value){
	    CGRect result = [value CGRectValue];
            result.size.width /= 2;
            result.size.height /= 2;
            return [NSValue valueWithCGRect:result];
        }];
{% endhighlight %}

> You can do whatever transformations, type convert, value convert etc.

And never ends
==================

The library is still in early stage, [join me](https://github.com/shuoli84/SLUtil) to improve it. 
* [github](https://github.com/shuoli84/SLUtil). 
* Follow me [github](https://github.com/shuoli84) [twitter](https://twitter.com/shuoli84)

{% highlight objective-c %}
    [changeChannel removeItem:view1ChangeItem]; 
    [changeChannel removeItem:propertyPanelChangeItem]; 
    [changeChannel removeItem:blockChangeItem]; 
    ......
{% endhighlight %}

