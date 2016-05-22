---
layout: post
title: "界面渲染的整体流程(续)CALayer的显示流程"
date: 2015-10-06 17:11:22 +0800
comments: true
categories: 
---

# 总结

1. 研究CALayer的display流程。

	```objectivec
	1.1 第一步：setNeedDisplay(UIView) -> setNeedDisplay(CALayer)
	1.2 第二步：View Drawing Cycle(Found dirty data) -> display(CALayer) -> displayInContext:(CALayer) 
	-> drawLayer:InContext:(CALayerDelegate<UIView>) -> drawRect:(CALayerDelegate<UIView>)
	1.3 第三步：View Drawing Cycle(Get layer model) -> Compositing Layers(OpenGLES) -> Render in device screen(GPU)
	```

2. setNeedDisplay时，是有BackingStore的参与的，通过符号断点CABackingStoreUpdate_可知。注：只创建CALayer时，需要调用setNeedDisplay才会使drawing cycle发起display流程；而创建UIView时设置frame时就会在UIView的内部调用setNeedDisplay，但是如果不把创建的view添加到父View上的话，CALayer的display方法同样也不会被调用，因为没有父View与说明不需要显示，那么肯定不会有display流程。
3. 不setNeedDisplay时，即直接修改layer的content时，无BackingStore参与，不会走display流程，conten不经过BackingStore这个缓冲区而直接被OpenGLES合成后加载到了GPU，而被显示了。相当于少了一道工序。这个流程应该与1.3一致。

#测试代码

[下载测试代码](http://)