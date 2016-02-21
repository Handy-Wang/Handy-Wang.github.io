---
layout: post
title: "RunLoop学习笔记(一) 基本原理介绍"
date: 2014-05-26 17:20:41 +0800
comments: true
categories: 
---
在iOS实际开发中，大家一定遇到过以下问题：

* 在一个线程里启动一个timer，但是这个timer一次也不会被调用？
* 在一个线程里发起一个NSURLConnection网络数据请求，但是NSURLConnection的delegate没有回调？
* 在主线程环境下的一个方法体里的第一行调用performSelector:withObject:afterDelay:这种带afterDelay的方法簇时，这一行代码实际执行时机往往是在方法体执行过程的最后，为什么呢？如下：

		- (void)testPerformSelectorAfterDelay {
    		[self performSelector:@selector(printMyName) withObject:nil afterDelay:0];
    		for (int i=0; i < 10; i++) {
        		NSLog(@"index is %d", i);
    		}
		}

		- (void)printMyName {
		    NSLog(@"My name is Handy.Wang ...");
		}
		
		输出结果是：
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 0
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 1
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 2
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 3
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 4
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 5
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 6
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 7
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 8
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 9
		2016-02-21 16:43:03.736 RunLoopXX[3782:200607] My name is Handy.Wang ...

下面，我就从实际问题出发，来分享一下关于RunLoop的基本知识，同时解决上面三个问题。

<!-- more -->

##什么是RunLoop
顾名思义，RunLoop就是一个一直在运行着的(带条件的)循环。从用户在iPhone上点开一个App开始，这个App就可以接收用户的操作、可以向远程的服务器发起数据请求、在App里展示各种数据列表，所以这个App给用户的感觉就是它是一直活着的，一直在手机上运行的着，这就是RunLoop起到的作用。即，保持住App主线程，同时接收用户的事件，让用户可以一直在App里进行操作并得到在界面上的反馈。

那么没有这个RunLoop的话，可以想像得到启动一个App后，手机屏幕上闪现一下App界面后，App的生命周期就结束了。这就好比执行了一段顺序逻辑的代码，而不是执行了一段循环逻辑的代码。

所以，iOS App的主线程内部就有一个RunLoop存在，只要用户不kill App或App在后台时不被回收，那么这个RunLoop就会一直Run.

从上面我们可以看到RunLoop是与(主)线程相关的，但是线程中不是必须有RunLoop，也就是说创建并启动一个线程后，如果确实需要在线程中运行一个RunLoop来维持线程，那么就需要在线程里创建一个RunLoop.

这里有一个疑问：为什么非得要创建RunLoop来维持线程？按理说，在线程内部创建一个(带条件的)循环也可以维持住线程，线程内部会一直循环执行，但是这无数次的循环过程中很有可能没有执行任何逻辑，因为循环里期待的条件还没有发生，照这样下去会一直消耗CPU资源，这完全没有必要。所以，RunLoop解决了CPU的空转问题，即当线程内部的逻辑在没有达到某种条件而不需要执行时，这个RunLoop就会休眠；当条件满足时，RunLoop会被唤醒，线程内就执行相应的逻辑。好智能的样子。

##RunLoop细节
在Foundation Framework中有相应的NSRunLoop API来操作RunLoop，其实它是对CoreFoundation Framework中的CFRunLoop的封装。CFRunLoop的底层实现用到了machkernel、block、pthread等技术。这里核心的是machkernel，RunLoop依靠它实现了休眠和唤醒而避免了CPU的空转。

在XCode中Debug主线程时，从调用栈里可以看到类似下面的某一个函数的调用，如对\__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__的调用。

	__CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__
	__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__
	__CFRUNLOOP_IS_SERVING_THE_MAIN_DISPATCH_QUEUE__
	__CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__
	__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_CALLBACK_FUNCTION__
	__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_CALLBACK_FUNCTION__
	
从上面的函数命名上可以知道，它们是RunLoop从内部回调到外部或外部Observer的函数。其实，无论主线程或子线程的RunLoop的回调，都是通过类似以上的这些函数来完成的。讲这个点，只是为了让同学们离RunLoop的机制近一点，以便于对后面内容的理解。

####与RunLoop相关的技术
其实，与RunLoop相关的技术我们经常用到，它们是，UI层相关的NSTimer、UIEvent、Autorelease；NSObject相关的NSObject (NSDelayedPerforming)方法簇、NSObject (NSThreadPerformAdditions)方法簇；CA相关的CADisplayLink、CATransition、CAAnimation；GCD中的dispatch_get_main_queue()；NSURLConnection。

####RunLoop的构成

* **RunLoop与线程是一一对应的且可以嵌套 <br/>**
首先，RunLoop与线程是一一对应的且可以嵌套，即在RunLoop里可以再创建RunLoop。 虽然RunLoop与线程一一对应，但是线程里的RunLoop不是必须存在的，开发过程中需要RunLoop时，则需要手动创建和运行RunLoop(尤其是在子线程中, Main RunLoop除外)，下面我举个例子：

		调用[NSTimer scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:]带有schedule的方法簇来启动Timer.
	
		此方法会创建Timer并把Timer放到当前线程的RunLoop中，然后RunLoop会在Timer设定的时间点回调到Timer设定的selector或Invocation。但是，在主线程和子线程中调用此方法的效果是有差异的，即在主线程中调用scheduledTimer方法时timer可以在设定的时间点触发，但是在子线程里则不能触发。这是因为子线程中没有创建RunLoop且也没有启动RunLoop，而主线程中的RunLoop默认是创建好的且一直运行着。所以，子线程中需要像下面这样调用。
	
    	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        	[NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(doTimer) userInfo:nil repeats:NO];
        	[[NSRunLoop currentRunLoop] run];
    	});
   
		那为什么下面这样调用同样不会触发Timer呢？
    	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        	[[NSRunLoop currentRunLoop] run];
        	[NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(doTimer) userInfo:nil repeats:NO];
    	});
    
    	我的分析是：scheduledTimerWithTimeInterval内部在向RunLoop中传递Timer时是调用与线程实例相关的单例方法[NSRunLoop currentRunLoop]来获取当RunLoop的，即RunLoop实例不存在就创建一个与当前线程相关的RunLoop并把Timer传递到RunLoop中，存在则直接传Timer到RunLoop中即可。而在RunLoop开始运行后再向其传递Timer时，由于dispatch_async代码块里的两行代码是顺序执行，RunLoop里的Timer一直Repeat没有运行完所以RunLoop这个死循环也没有结束就无法执行到“[NSTimer scheduledTimerWithTimeInterval:...”，自然也就没有时机把Timer加到当前RunLoop，所以更不会触发Timer了。

* **RunLoop的RunLoopModes** <br/>
其次，RunLoop有不同的Mode(NSRunLoop/CFRunLoopMode)。
	* 一个RunLoop可以运行不同的RunLoopMode下，但是同一时间段内只能运行中一个RunLoopMode下。
	* 如果要切换RunLoopMode，需要先停止RunLoop，修改RunLoopMode，再重启新RunLoop。<br/><br/>
	
	举例：App的Main RunLoop，在App启动的时运行在NSInitializationRunLoopMode下，App启动后运行在NSDefaultRunLoopMode下，有用户操作时App运行在NSTrackingRunLoopMode下。
	
	常用的RunLoopMode: 
	* **NSDefaultRunLoopMode**: RunLoop的默认Mode，空闲状态。
	* **UITrackingRunLoopMode**: 有滑动等其它需要追踪的事件发生时，RunLoop处于此Mode。
	* **NSInitializationRunLoopMode**: 这是一个Private RunLoopMode，App启动时处于此Mode，启动完成进入App主界面后App处于NSDefaultRunLoopMode。
	* **NSRunLoopCommonModes**: 此Mode默认包含了NSDefaultRunLoopMode和UITrackingRunLoopMode。所以，当RunLoop运行在NSDefaultRunLoopMode或UITrackingRunLoopMode时，监听NSRunLoopCommonModes的Observer都被回调。另外，还可以向NSRunLoopCommonModes里添加其它自定义的Mode.

* **RunLoop的InputSource、Timer、Observer** <br/>到这里，我们知道RunLoop不只是为了维持线程的持续运转，更重要的是为了在线程里面接收RunLoop外部的一些逻辑请求，我把这些描述逻辑请求的数据结构(InputSource、Timer)统称为RunLoop的数据源，RunLoop根据数据源里设定的条件处理完数据源后会回调到数据源的Observer.<br /><br />
那么结合RunLoop的不同模式以及App运行时有很多RunLoop，所以在RunLoop内部肯定是有一个RunLoopMode与InputSource/Timer的一对多的关系，即每个RunLoopMode都对应了多个不同的数据源(InputSource/Timer)。也就是说，一个RunLoop运行在某个RunLoopMode时，只会触发此模式下的InputSource、Timer集合，进而再回调到对应的Observer。至于RunLoopMode里的InputSource、Timer集合元素的执行先后顺序取决于InputSource、Timer的自身描述，比如InputSource的UIEvent触发时机不一样、Timer的时间间隔不一样。
	* **InputSource(CFRunLoopSource)**<br/>
	InputSource就是RunLoop数据源的一种，它是一个抽象概念，它有具体的实现，如后面会提到的UIEvent、CFSocket、CFMachPort、CFMessagePort。在RunLoop里InputSource分为：source0、source1、自定义source
		* **source0**: 处理App内部事件，App负责管理自己的事件触发，如：UIEvent(Touch事件等，GS发起到RunLoop运行再到事件回调到UI)、CFSocket
		* **source1**: 由RunLoop和内核管理，由Mach port驱动，如CFMachPort、CFMessagePort。特别要注意一下Mach port，它是一个轻量级的进程间通讯的方式，可以理解为它是一个通讯通道，假如同时有几个进程都挂在这个通道上，那么某些进程向这个通道发送消息后，别一些挂在这个通道上的进程就可以收到相应的消息。这个Port的概念非常重要，因为它是让RunLoop休眠和被唤醒的关键，它是RunLoop与系统内核进行消息通讯的窗口。
		* **自定义source**: 自定义的source也必须是source0和source1中的一种，然后可以被添加到RunLoop里运行。
		
		在CFRunLoopSource的源码里，source0、source1是以一个union的数据结构存在的
		
			union {
				CFRunLoopSourceContext version0;
				CFRunLoopSourceContext version1;
			} _context;
			
			typedef stuct {
				......
			} CFRunLoopSourceContext
		
	* **Timer(NSTimer/CFRunLoopTimer)**<br/>涉及到Timer的技术点有:
	  * **NSTimer**: +(NSTimer *)timerWithTimeInterval:...、+(NSTimer *)scheduledTimerWithTimeInterval:...，前一个方法创建的timer需要手动的添加到RunLoop里、后一个方法创建的timer会被自动的添加到当前RunLoop里，无论这两种中的哪种情况，都要确保timer被添加到的RunLoop是处于Running的。
	  * **带afterDelay的方法簇**: -performSelector:withObject:afterDelay:，此方法簇的内部创建了一个隐式的timer并添加到当前RunLoop中，但是也要注意上一条同样的问题，一定要确保当前RunLoop处于Running状态。比如说下面的代码里的doPerformSelectAfterDelay方法是不会被调用到的，因为第一个的RunLoop没有运行，第二个的RunLoop已经运行完了。
	  
			dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
				[self performSelector:@selector(doPerformSelectAfterDelay) withObject:nil afterDelay:0];
			});

			dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
				[[NSRunLoop currentRunLoop] run];
				[self performSelector:@selector(doPerformSelectAfterDelay) withObject:nil afterDelay:0];
			});	
			
		所以，正确的写法是如下：
		
			dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
				[self performSelector:@selector(doPerformSelectAfterDelay) withObject:nil afterDelay:0];
				[[NSRunLoop currentRunLoop] run];
			});	
	  
	  * **CADisplayLink**: +(CADisplayLink *)displayLinkWithTarget:selector:、- (void)addToRunLoop:forMode:。CADisplayLink完全是一个Timer，它的刷新频率与UI主线程的刷新频率是完全一致的。CADisplayLink默认每秒运行60次， 如果将它的frameInterval属性设置为2，意味CADisplayLink每隔一帧运行一次，使得回调频率由每秒60次降为30次。

	* **Observer(CFRunLoopObserver)**<br/>
	与Observer非常相关的就是CFRunLoopActivity，即从侧面反映出Observer回调时RunLoop当前的状态。具体每个状态的意义见我在每个状态后面加的注释。
	
		/* Run Loop Observer Activities */
		typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
		    kCFRunLoopEntry = (1UL << 0), //InputSource/Timer已经加入到RunLoop了
		    kCFRunLoopBeforeTimers = (1UL << 1), //Timer即将要被执行了
		    kCFRunLoopBeforeSources = (1UL << 2), //InputSource即将要被执行了
		    kCFRunLoopBeforeWaiting = (1UL << 5), //RunLoop即将休眠了
		    kCFRunLoopAfterWaiting = (1UL << 6), //RunLoop即将被唤醒
		    kCFRunLoopExit = (1UL << 7), //RunLoop停止运转了
		    kCFRunLoopAllActivities = 0x0FFFFFFFU
		};
		
	所以，我们可以用Observer监听RunLoop状态的变化，事实上Framework里很多机制都是由RunLoopObserver触发的。如CAAnimation，RunLoop应该是在收集完每一轮要做的事情后，在RunLoop一轮的后期(kCFRunLoopBeforeWaiting, kCFRunLoopAfterWaiting)来执行动画。
			
####RunLoop与AutoRelease
我们知道AutoRelease对象是被AutoReleasePool管理的，那么AutoRelease对象在什么时候被回收呢？可能有人会说在AutoReleasePool回收的时候。这样说没错，但是对于Application主线程的AutoReleasePool里的AutoRelease对象来说呢，因为Main AutoReleasePool是不会被回收的。带着这个问题，我们来分析一下：

第一种情况：在我们自己写的for循环或线程体里，我们都习惯用AutoReleasePool来管理一些临时变量的autorelease，使得在for循环或线程结束后回收AutoReleasePool的时候来回收AutoRelease临时变量。所以，刚才的回答没错，但是比较片面。

另一种情况：我们在主线程里创建了一些AutoRelease对象，这些对象可不能指望在回收Main AutoReleasePool时才被回收，因为App一直运行的过程中Main AutoReleasePool是不会被回收的。其实，这种AutoRelease对象的回收机制就依赖Main RunLoop的状态，Main RunLoop的Observer会在Main RunLoop结束休眠被唤醒后且即将开始下一轮时(kCFRunLoopAfterWaiting状态时回调)通知UIKit，UIKit收到这一通知后就会调用_CFAutorleasePoolPop方法来回收主线程中的所有AutoRelease对象。

####UITrackingRunLoopMode与Timer
在主线程中Schedule一个Timer并正常运行，然后滑动动界面上的TableView或UIScrollView时，之前的Timer不触发了。为什么呢？

	[NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(doTimer1) userInfo:nil repeats:YES];

这个问题就与RunLoop的RunLoopMode有关。因为schedule一个Timer时，Timer是被加到了RunLoop的NSDefaultRunLoopMode下，而在滑动TableView或UIScrollView时，RunLoop的Mode被切换到了UITrackingRunLoopMode下，RunLoop就只会去跟踪UI上的滑动事件了而Timer被暂停不会被触发了。

	通过对CFRunLoopWakeUp的符号断点得知，UIApplication是通过
	-[UIApplication pushRunLoopMode:requester:] ()
	和
	-[UIApplication popRunLoopMode:requester:] ()
	两个方法来切换RunLoop模式的。


所以，只需要把Timer加到RunLoop的NSRunLoopCommonModes下就可以解决此问题了，因为NSRunLoopCommonModes包含了UITrackingRunLoopMode。

	NSTimer *timer1 = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(doTimer1) userInfo:nil repeats:YES];
	
 	[runloop addTimer:timer1 forMode:NSRunLoopCommonModes];

####RunLoop与GCD
RunLoop与GCD并没有直接关系，当且仅当GCD使用到main_queue时才有关系，如下：

    //实验GCD Timer 与 Runloop的关系，只有当dispatch_get_main_queue时才与RunLoop有关系
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        NSLog(@"GCD Timer...");
    });
    
如果debug代码NSLog(@"GCD Timer...");这一行的话，可以在调用栈里看到Main RunLoop是通过__CFRUNLOOP_IS_SERVING_THE_MAIN_DISPATCH_QUEUE__来回调的，而且把dispatch_get_main_queue换成dispatch_get_global_queue，那么在调用栈里将不会看到RunLoop相关的调用，而是和pthread相关。所以，GCD的dispatch_after timer只有在主线程中时才与RunLoop相关，即仅当GCD使用到main_queue时才有关系。

####RunLoop的挂起和唤醒
* **RunLoop的挂起**<br />
	RunLoop的挂起是通过_CFRunLoopServiceMachPort -> mach_msg -> mach_msg_trap这个调用顺序来告诉内核监听哪个MachPort(前面提到的通道)，然后等待消息的发生(等待与InputSource、Timer描述内容相关的消息)，内核就把RunLoop挂起了，即RunLoop休眠了。
* **RunLoop的唤醒**<br />
	当RunLoop被挂起后，如果之前监听的事情发生了，那么就需要被另一个进程或线程来唤醒主动监听的RunLoop，那么这个进程或线程就需要赂MachPort发一个消息即可。
* **RunLoop内部运行逻辑伪代码** －根据RunLoop源代码整理的伪代码

		SetupThisRunLoopRunTimeoutTimer(); //By GCD timer.
		do {
			__CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
			__CFRunLoopDoObservers(kCFRunLoopBeforeSources);
			
			__CFRunLoopDoBlocks(); // 执行6个__CFRUNLOOP_IS_*中的某些函数
			__CFRunLoopDoSource0(); // 执行加入source0里的逻辑
			
			CheckIfExistMessagesInMainDispatchQueue(); //问GCD有没有需要要主线程中执行的事情
			
			__CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);
			var wakeUpPort = SleepToWaitForWakingupPorts(); //mach_msg_trap
			//mach_msg_trap
			//Zzz......
			//Received mach_msg, wake up
			__CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
			//Handle msgs
			if (wakeUpPort == timerPort) { //Timer的port消息
				__CFRunLoopDoTimers();
			} else if (wakeUpPort == mainDispatchQueuePort) { //GCD的port消息
				//GCD
				__CFRunLoop_IS_SERVING_THE_MAIN_DISPATCH_QUEUE__();
			} else {
				__CFRunLoopDoSource1(); //基于port的消息通读，如：某个端口来网络数据了等情况
			}
			
			__CFRunLoopDoBlocks(); // 执行6个__CFRUNLOOP_IS_*中的某些函数
			
		} while (!stop && !timeout)
		
####总结
在本文开头时，我提了三个问题，结合上面讲到的内容，分别作答如下：

* 在一个线程里启动一个timer，但是这个timer一次也不会被调用？
	
		答：线程里并没有创建RunLoop，所以要想让线程里的Timer正常运行，那么需要创建RunLoop、把Timer放到RunLoop里、再运行RunLoop.
		
		NSTimer *timer1 = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(doTimer1) userInfo:nil repeats:YES];
		[runloop addTimer:timer1 forMode:NSRunLoopCommonModes];
		[runloop run];
		
* 在一个线程里发起一个NSURLConnection网络数据请求，但是NSURLConnection的delegate没有回调？

		答：需要在保证NSURLConnection线程alive的前提下，将NSURLConnection以NSRunLoopCommonModes加入到RunLoop中运行。为什么是以NSRunLoopCommonModes模式呢，结合本文讲解可知，为了在RunLoop处于TrackingMode时也能处理接收到数据回调的CALLBACK。如，在滑动数据列表时也能加载并显示出网络图片。
		
* 在主线程环境下的一个方法体里的第一行调用performSelector:withObject:afterDelay:这种带afterDelay的方法簇时，这一行代码实际执行时机往往是在方法体执行过程的最后，为什么呢？如下：

		- (void)testPerformSelectorAfterDelay {
    		[self performSelector:@selector(printMyName) withObject:nil afterDelay:0];
    		for (int i=0; i < 10; i++) {
        		NSLog(@"index is %d", i);
    		}
		}

		- (void)printMyName {
		    NSLog(@"My name is Handy.Wang ...");
		}
		
		输出结果是：
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 0
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 1
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 2
		2016-02-21 16:40:25.880 RunLoopXX[3782:200607] index is 3
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 4
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 5
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 6
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 7
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 8
		2016-02-21 16:40:25.881 RunLoopXX[3782:200607] index is 9
		2016-02-21 16:43:03.736 RunLoopXX[3782:200607] My name is Handy.Wang ...

		答：在testPerformSelectorAfterDelay方法体内的代码肯定是按顺序执行，先执行performSelector:withObject:afterDelay方法，再执行for循环。由于performSelector只是向RunLoop注册Timer Source并不是执行，尽管delay是0，所以注册完后就执行完了，接着就执行for循环，然后testPerformSelectorAfterDelay方法运行完了，整个App就没有什么可做的了进入了sleep。我们结合本文讲的知识和上面提到的伪代码可知，Timer真正地执行时机是取决于是在RunLoop否收收到了timer machport msg，由于RunLoop已进入了sleep，而且delay是0，所以瞬间之后RunLoop收到了timer mach_msg从而把RunLoop唤醒然后回调Timer要做的事情，再然后RunLoop接着进入sleep.

到此，RunLoop的相关知识就介绍完了。下篇我将结合开源项目AFNetworking来讲讲RunLoop的实际应用。
<br /><br /><br />

##参考资料
* [Threading Programming Guide - iOS Developer Library](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW1)
* [视频: iOS线下分享《RunLoop》by 孙源@sunnyxx](http://v.youku.com/v_show/id_XODgxODkzODI0.html)
* [深入理解RunLoop](http://www.cocoachina.com/ios/20150601/11970.html)