---
layout: post
title: "UIView与CALayer中动画的创建和执行过程"
date: 2015-10-11 22:49:40 +0800
comments: true
categories: 
---

##写在最前面
本文假设您已有RunLoop和CAAnimation的相关知识，所以这里不对[RunLoop](http://blog.handy.wang/blog/2014/05/26/runloopxue-xi-bi-ji-1/)和CAAnimation的细节进行介绍。但是，这里仍然要提前提及几个知识点：CFRunLoopActivity、CAAction、CAAnimation、CATransaction

**RunLoop的6个CFRunLoopActivity**

	typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
	    kCFRunLoopEntry = (1UL << 0),//即将进入RunLoop
	    kCFRunLoopBeforeTimers = (1UL << 1),//Timer即将触发
	    kCFRunLoopBeforeSources = (1UL << 2),//Source0即将触发
	    kCFRunLoopBeforeWaiting = (1UL << 5),//RunLoop即将休眠
	    kCFRunLoopAfterWaiting = (1UL << 6),//RunLoop已被唤醒
	    kCFRunLoopExit = (1UL << 7),//RunLoop已结束
	    kCFRunLoopAllActivities = 0x0FFFFFFFU
	};
**CAAction**
CAAction即动画行为，它是一个protocol(CAAnimation实现了此protocol)，即定义一个动画要做的事情。比如，我们平时无论是否在Animaiton block里修改View的属性时，都会触发Core Animation回调CALayer，再回调UIView来获取CAAction(如：CABasicAnimation)，如果不在Animaiton block里修改View的属性时获取到的是[NSNull null]，在Animaiton block里修改View的属性时获取到的是CABasicAnimation等(后面有图标这个过程)，也可以返回自定义的动画(实现了CAAction的动画)。

**CAAnimation的继承结构**

![CAAnimation.jpg](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/uiview_calayer_animation_creation_caanimation_hierarchy.jpg)

**CATransaction**
CATransacton是一个与动画相关的概念，它负责把多个对Layer或View的可动画属性的修改集中在一起一次性提交并执行，所以Animation应该需要被包含在CATransaction中的。CATransaction分为**隐式**和**显示**。 
如下，这就是一个**显示的CATransaction**代码片断，即由开发人员来begin和commit。

	[CATransaction begin];
	[CATransaction setValue:@(NO) forKey:kCATransactionDisableActions];
	[CATransaction setValue:@(0.5) forKey:kCATransactionAnimationDuration];
	[CATransaction setValue:^() {NSLog(@"Completion...");} forKey:kCATransactionCompletionBlock];
	_animLayer.position = CGPointMake(_animLayer.position.x, _animLayer.position.y - 10);
	[CATransaction commit];
从代码片断中可见虽然只是修改了layer的position，但是最终动画的duration、动画回调都可能通过CATransaction来提供。当不是简单的修改position，而是给layer加一个CAAnimation时，最终动画的duration等参数则是以CAAnimation的内容为主。同时，CATransaction支持嵌套，以最外层commit为准。这里还需要关注一下对kCATransactionDisableActions的修改，kCATransactionDisableActions表示是否禁用CAAction的检索：若为YES则禁用，即无论在动画代码块里的UI属性修改都不会有动画效果，actionForKey:方法不会回调；若为NO则开启CAAction的检索。

关于**隐式的CATransaction**，有[资料](http://blog.csdn.net/yaozhuoyu/article/details/9533909)说是

    在大多数情况下，我们并不需要去创建自己的transaction。
    当我们给layer添加一个显式或者隐式的动画的时候，core animation会自动的为我们创建一个隐式的
    transaction。

也有[资料](http://geeklu.com/2012/09/animation-in-ios/)说是

    CATransaction也分两类,显式的和隐式的,当在某次RunLoop中设置一个animatable属性的时候,
    如果发现当前没有事务,则会自动创建一个CA事务

但是，无论通过修改Layer的position属性来达到隐式动画效果时，还是通过修改View的center属性达到显式动画效果时断点+[CATransaction begin]、CA::Transaction::create()方法都没有被调用，说明没有新创建CATransaction.

    隐式动画
    _animLayer.position = CGPointMake(_animLayer.position.x, _animLayer.position.y - 10);

    显式动画
    [UIView animateWithDuration:0.5 animations:^{
        _animView.center = CGPointMake(_animView.center.x, _animView.center.y - 10);
    }];
但是前面我们提到Animation是要依附于CATransaction的commit才得以执行的，而且我们在实验隐式动画和显式动画时，当CAAction被查找(后面会图示)完成后CA::Transaction:commit会被调用到，所以RunLoop回调进行创建动画的过程时应该事先已创建了一个根Transaction，再次通过断点CA::Transaction::create方法也证明了Main RunLoop通过回调**__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__**只调用了一次CA::Transaction::create方法来创建了一个根Transaction，所以这就能解释能为什么隐式动画和显式动画时Core Animation内部没有新创建CATransaction而只是在动画CAAction返回后调用CA::Transaction:commit就可以执行动画了。所以，我理解的隐式CATransaction是指前面提到的根Transaction。

##正文
在了解了以上知识后，我才会告诉你，本文主要是介绍我们在UIView或CALayer中通过动画块或不通过动画块来修改其属性时的动画创建和执行过程。先看一段代码，如下：

    代码一：隐式动画
    _animLayer = [IDCAAnimationTestLayer new];
    _animLayer.position = CGPointMake(_animLayer.position.x, _animLayer.position.y - 10);

    代码二：无动画效果
    _animView.layer.position = CGPointMake(_animView.layer.position.x, _animView.layer.position.y - 10);
    
    代码三：无动画效果
    [CATransaction begin];
    [CATransaction setValue:@(NO) forKey:kCATransactionDisableActions];
    [CATransaction setValue:@(0.5) forKey:kCATransactionAnimationDuration];
    _animView.layer.position = CGPointMake(_animView.layer.position.x, _animView.layer.position.y - 10);
    [CATransaction commit];

    代码四：显式动画
    [UIView animateWithDuration:0.5 animations:^{
        _animView.center = CGPointMake(_animView.center.x, _animView.center.y - 10);
    }];

    代码五：无动画效果
    _animView.center = CGPointMake(_animView.center.x, _animView.center.y - 10);

从以上代码中可以看到：为什么同样是对CALayer或UIView的相同属性作修改，而有些有动画而有些没有动画呢？带着这个疑问，我们接下来看下面这张图。

![E_UIView:CALayer的动画创建及执行过程.jpg](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/uiview_calayer_animation_creation_callprocess.jpg)
如上图，我来分别梳理一个修改CALayer和UIView属性时的动画创建过程(没错，其实不在动画块里修改属性也有动画创建过程存在的)。

#####CALayer的动画创建和执行过程(如图中黑实线箭头及序号)
1. RunLoop触发两类回调事件：BeforeWaiting、点击事件、ExitRunLoop
2. RunLoop回调事件回调到开发人员写的修改CALayer属性的代码
3. 当修改CALayer的animatable属性时会触发CALayer自己的actionForKey:方法来查找相应的CAAction，CALayer的actionForKey方法的默认实现会返回一个实现了CAAction protocol的CAAnimation，如CABasicAnimation
4. 开发人员的修改CALayer属性的代码执行完毕后，在RunLoop的回调函数的后续逻辑会调用CA::Transaction:commit()来提交之前修改CALayer属性而生成的CAAction.
5. 最后相应的动画效果显示到屏幕上(TODO:这里的细节还有待调研)

#####UIView的动画创建和执行过程(如图中虚线箭头及序号)
1. RunLoop触发两类回调事件：BeforeWaiting、点击事件、ExitRunLoop
2. RunLoop回调事件回调到开发人员写的修改UIView几何或透明度等UI属性的代码，这里修改有两种情况如下：
2.1.1 不在动画代码块里修改UIView属性：修改UIView的几何、透明度等UI属性实际上最终是通过CALayer的对应属性修改来体现的，如修改UIView的center实际上最终是通过CALayer的position来体现的。
2.1.2 在动画代码块里修改UIView属性
2.2 如2.1.1所说，无论是否在动画代码块里修改UIView的几何、透明度等UI属性，实际上是通过CALayer修改对应属性来完成的，这是因为UIView与CALayer的关系是平行的，且UIView是CALayer的Delegate。
3. 与**CALayer的动画创建和执行过程**的流程一致。但是需要注意的是：如果是2.1.1触发这一步，那么返回的是[NSNull null]，即没有动画(表示停止对CAAction的检索)；如果是2.1.2触发这一步，那么返回的结果与**CALayer的动画创建和执行过程**第3步一样，即实现了CAAction protocol的CAAnimation子类。从这两种情况我们就不难明白，为什么不在动画代码块里修改UIView的UI属性没有动画，反之有动画，这都是由actionForKey:是否有确切的CAAction值决定的。顺带讲一下，actionForKey:有三种返回值情况：id<CAAction>、nil，nil表示没有没有任何动画行为。
4. 同**CALayer的动画创建和执行过程**
5. 同**CALayer的动画创建和执行过程**

讲到这里，我再来回顾前面的问题**“为什么同样是对CALayer或UIView的相同属性作修改，而有些有动画而有些没有动画呢？”**，再来看一下代码并作相应解释：


    代码一：隐式动画
    _animLayer = [IDCAAnimationTestLayer new];
    _animLayer.position = CGPointMake(_animLayer.position.x, _animLayer.position.y - 10);
    解释：这里首先解释一个名词叫Root Layer和非Root Layer。很简单，Root Layer就是指有对应UIView的Layer，
    非Root Layer就是指没有对应UIView的Layer，在CALayer里只有对非Root Layer的UI属性修改才会有隐式动画效果。
    其实，不难理解，因为Root Layer有UIView，且修改UI属性的代码没有写到动画代码块里，
    所以CAAction的返回被UIView的actionForLayer:forKey:方法返回为[Null null]，进一步回调到CALayer里actionForKey:的返回值为nil。
    但是，如果在动画代码块里修改Root Layer的UI属性是会有动画效果的。
    显然，_animLayer是一个非Root Layer，修改position属性后会按照上面的“CALayer的动画创建和执行过程”来执行，所以有动画效果。

    代码二：无动画效果
    _animView.layer.position = CGPointMake(_animView.layer.position.x, _animView.layer.position.y - 10);
    解释：显然_animView.layer是一个Root Layer，而且没有写在动画代码块里，所以不会有隐式动画。
    
    代码三：无动画效果
    [CATransaction begin];
    [CATransaction setValue:@(NO) forKey:kCATransactionDisableActions];
    [CATransaction setValue:@(0.5) forKey:kCATransactionAnimationDuration];
    _animView.layer.position = CGPointMake(_animView.layer.position.x, _animView.layer.position.y - 10);
    [CATransaction commit];
    解释：同**代码二**

    代码四：显式动画
    [UIView animateWithDuration:0.5 animations:^{
        _animView.center = CGPointMake(_animView.center.x, _animView.center.y - 10);
    }];
    解释：参见**UIView的动画创建和执行过程**及其中的2.1.2点，所以有动画效果。

    代码五：无动画效果
    _animView.center = CGPointMake(_animView.center.x, _animView.center.y - 10);
    解释：参见**UIView的动画创建和执行过程**及其中的2.1.1点，所以没有动画效果。

以上内容就是本章的全部内容，欢迎邮件到nnnwjs@126.com讨论。

## 参考
* [RunLoop学习笔记(一) 基本原理介绍](http://blog.handy.wang/blog/2014/05/26/runloopxue-xi-bi-ji-1/)
* [Core Animation 高级动画技巧](http://blog.csdn.net/yaozhuoyu/article/details/9533909)
* [谈谈iOS Animation](http://geeklu.com/2012/09/animation-in-ios/)
* [iOS开发基础知识：Core Animation(核心动画)](http://www.jianshu.com/p/8c1c1697c0ce)
* [iOS 事件处理机制与图像渲染过程](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=400417748&idx=1&sn=0c5f6747dd192c5a0eea32bb4650c160&scene=4#wechat_redirect)
* 参考以上资料的过程一定要做实验
