---
layout: post
comments: true
title: Run some code when one object dealloc
categories:
- iOS
tags:
- block
- iOS
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---

Motivation
------------------
I loved the way of what block gives me, especially after adopt of the BlocksKit.

Though there is one thing which bugged me, when and where should I do clean up. E.g, add an observer on one object's key with a block, when should I remove this observer. This is really a problem when I create this kind of stuff in a block or function. UILabel A is the observer, Object B is the observed object. I add the code to observe B's name, when it is changed, update A's text to reflect.

{% highlight objective-c %}
[objectB addObserverForKeyPath:@"name" identifier:@"observeNameAndUpdateLabel" options:xxxx task:^(id obj, NSDictionary *change){
//get name and set label A (should use a weak A, otherwise A will never be released)
}];
{% endhighlight %}

In some point I should call removeObservor on B to remove it, otherwise, there could be a leak.

Solution
-----------------
When: Ideally when label A's retain count reach 0.
How: Add some code when A's gone cleaned. Here is what I do:

> 1. create a class whose object able to accept one block, and execute it in its dealloc.
> 2. create a object of that class and bind a block which contains clean up code
> 3. associate the created object to A


When A get dealloc, its associated object's retain count -1, since A is the only object holds the ref, then the wrapper object gets dealloc. Then our clean up code get called. Woollaa!!
