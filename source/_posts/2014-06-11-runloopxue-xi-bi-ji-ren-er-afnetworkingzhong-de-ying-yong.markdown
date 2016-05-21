---
layout: post
title: "RunLoop学习笔记(二) AFNetworking中的应用"
date: 2014-06-11 11:16:27 +0800
comments: true
categories: [RunLoop]
---

<!--more-->

如果理解了上一篇RunLoop原理的内容，那么再来看AFNetworking的核心网络请求部分就很简单了。

对于AFNetworking来说核心的有两大部分：网络请求和缓存。

本文只讲AFNetworking的网络请求部分，以及是如何与RunLoop产生关系的。

在类AFURLConnectionOperation.m的163行左右有以下代码
	
	+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];

        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
	}
	
	+ (NSThread *)networkRequestThread {
	    static NSThread *_networkRequestThread = nil;
	    static dispatch_once_t oncePredicate;
	    dispatch_once(&oncePredicate, ^{
	        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
	        [_networkRequestThread start];
	    });
	    
	    return _networkRequestThread;
	}
	
此代码片断创建了一个名为AFNetworking单例的线程，给这个线程的RunLoop设定了一个[NSMachPort port]，让RunLoop一直监听NSMachPort实例上的消息，但是，可以看到这个port实例没有被保存，所以在整个AFNetworking源代码里是不会有线程给这个port实例发消息的，也就是变相的将此线程一直维持住，即这个线程的RunLoop是“死循环”。

我们在这个类里再搜下networkRequestThread的调用时，可以看到下面的代码：

	- (void)operationDidStart {
	    [self.lock lock];
	    if (![self isCancelled]) {
	        self.connection = [[NSURLConnection alloc] initWithRequest:self.request delegate:self startImmediately:NO];
	        
	        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
	        for (NSString *runLoopMode in self.runLoopModes) {
	            [self.connection scheduleInRunLoop:runLoop forMode:runLoopMode];
	            [self.outputStream scheduleInRunLoop:runLoop forMode:runLoopMode];
	        }
	        
	        [self.outputStream open];
	        [self.connection start];
	    }
	    [self.lock unlock];
	    
	    dispatch_async(dispatch_get_main_queue(), ^{
	        [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingOperationDidStartNotification object:self];
	    });
	}
这是发起网络请求的代码。创建了一个NSURLConnection并将它加入到刚才的单线程中执行。同时，在AFURLConnectionOperation.m文件中还可以看到其它围绕networkRequestThread这个线程展开的逻辑，比如operationDidPause、cancelConnection。所以，这个单例线程以及采用NSMachPort的方式维持RunLoop的方式才是核心。

**综上**，AFNetoworking的网络请求的核心是：**单例线程** + **NSRunLoop** + **NSMachPort** + **NSURLConnection**。

