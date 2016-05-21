---
layout: post
title: "iOS App UI调试工具（二）· Realtime Debug Portal"
date: 2013-12-17 05:30:58 +0800
comments: true
categories: [调试]
---

<!--more-->

{% img https://github-camo.global.ssl.fastly.net/03548ab74978beee15b711ab26b5ed1bd71714f2/687474703a2f2f7777772e76696e716f6e2e636f6d2f636f6465626c6f672f66636b656469746f722f75706c6f61642f696d6167652f323031332d30362f325f322e706e67 %}
<h3>Realtime Debug Portal简介</h3>
{% blockquote Realtime Debug Portal https://github.com/vinqon/Realtime-Debug-Portal Read on %}
RDP是一个类似Web Inspector的工具，把这个工具引入我们的项目工程，并做一些简单的配置，然后启动应用，
在浏览器输入手机的IP地址，就可以看到UIView的树状结构和Log信息，还可以在浏览器中对View进行移动，
隐藏，选中高亮等操作。
{% endblockquote %}

<h3>配置步骤</h3>
```
把库文件、头文件以及资源文件(bundle)引入项目即可，有两点需要注意一下：

把工程中的Build Settings中的Other Linker Flags设置为-ObjC;
使用iOS5或以上SDK;
然后在合适位置调用以下代码：
#import "libRDP.h"
[RDP startServer];
启动应用之后，状态栏会显示出你需要访问的地址，模拟器一般会显示http://127.0.0.1:8080 ，
请使用Chrome或者Safari打开。

当选中某个view之后，RDP会在这个view上面盖一层蓝色透明遮罩以便开发者区别。
用户可以通过按下方向键来移动view，每次会移动1个逻辑像素；按住shift加方向键可以移动10个逻辑像素；
按住w字母键，加方向键可以调整大小；
点击h可以切换hidden状态；
```
也可参考github上<a target="_blank" href="https://github.com/vinqon/Realtime-Debug-Portal">这里</a>

<h3>试用体会</h3>
<div>
	1）目前，支持视图的Inspect、position和size调整、隐藏或显示
</div>
<div>
    2）方便集成到项目，UE简单明了很容易上手，基本能满足界面布局调试需求
</div>

<h3>参考</h3>
<ol style="margin-top:-18px; padding-left:28px;">
	<li style="padding-bottom:10px;">
	    <a target="_blank" href="http://www.vinqon.com/codeblog/?detail/11109">Redesign Your App for iOS 7 之 页面布局</a><br/>
	</li>
	<li>
	    <a target="_blank" href="https://github.com/vinqon/Realtime-Debug-Portal">Realtime Debug Portal的github</a><br/>
	</li>
</ol>

