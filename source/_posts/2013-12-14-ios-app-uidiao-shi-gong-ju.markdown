---
layout: post
title: "iOS App UI调试工具（一）· Symbiote In Frank"
date: 2013-12-14 21:48:49 +0800
comments: true
categories: 
---
{% img http://blog.thepete.net/images/post_images/2012-06-24_A.png %}
<h3>Frank简介</h3>
{% blockquote Frank http://www.testingwithfrank.com/ Read on %}
Frank is 'Selenium for native iOS apps'. It allows you to write automated acceptance tests 
which verify the functionality of your native iOS app.

Frank是一个iOS平台的自动化测试工具。它支持编写一些自动化验收测试用例来验收iOS App的功能。
{% endblockquote %}

<h3>Symbiote简介</h3>
{% blockquote Symbiote http://blog.thepete.net/blog/2011/05/01/inspect-state-of-our-running-ios-apps/ Read on %}
Frank comes with a useful little tool called Symbiote. It’s a little web app 
which is embedded inside your native iOS application. Its purpose is to 
let you inspect the current state of your app’s UI, and to test the 
UIQuery selectors which Frank uses to help it automate your app. 
Essentially Symbiote is Firebug for your native iOS app.

Frank中包含了一个非常有用的小工具，它叫叫Symbiote。它是一个嵌入到iOS App中的一个Web应用程序。
它的功能是可以让开发者能够调试运行时的iOS App UI的当前状态(层次结构、大小、位置等信息)，
同时它也是为了测试Frank内部的一个用于自动化App的功能UIQuery选择器(UIQuery selector可以理解
为通过某些名称或ID值找到App的UI元素，和JQuery一个设计思路)。
从其功能本质上讲，Symbiote就是一个用于iOS App的FireBug。
{% endblockquote %}

<h3>Frank以及Symbiote的环境搭建</h3>
完全按照<a href="http://blog.thepete.net/blog/2012/06/24/writing-your-first-frank-test/">Writing Your First Frank Test</a>操作即可。
请注意：这个HowTo中会有Ruby环境和插件的安装，做好心理准备。
Touble shoot：nnnwjs@126.com

<h3>试用体会</h3>
<div>
    1）通过Web GUI界面动态展示程序运行时可视UI的层次结构和相关属性(宽、高等)
</div>
<div>
    2）操作App时，Web GUI界面和相关信息是同步变化的
</div>
<div>
    3）在Web GUI界面中点UI层次结构里某节点时，右侧的Web Simulator中会高亮出点击区域
</div>
<div>
    4）在Web GUI界面里可以通过选择节点或编写UIQuery selector来控制App
</div>
<div>
    5）Web GUI界面的UI/UE实再是太差劲了
</div>
<div>
    6）更多功能有待进一步试用...
</div>

<h3>参考</h3>
<ol style="margin-top:-18px; padding-left:28px;">
<li style="padding-bottom:10px;">
    <a target="_blank" href="http://www.cocoachina.com/applenews/devnews/2013/1111/7332.html">你用哪种工具进行iOS app自动化功能测试？</a><br/>
</li>
<li>
    <a target="_blank" href="http://moredip.github.io/frank_at_selenium_slides.html">Frank presentation slides</a><br/>
</li>
<li>
    <a target="_blank" href="http://www.testingwithfrank.com/">Testing With Frank(Frank网站)</a><br/>
</li>
<li style="padding-bottom:10px;">
    <a target="_blank" href="https://github.com/TestingWithFrank/Frank">Frank github</a><br/>
</li>
<li>
    <a target="_blank" href="http://blog.thepete.net/blog/2011/05/01/inspect-state-of-our-running-ios-apps/">Inspect the State of Your Running iOS App's UI With Symbiote(Symbiote介绍)</a><br/>
</li>
<li>
    <a target="_blank" href="https://github.com/TestingWithFrank/symbiote">Symbiote github(介绍、Mockup static Symbiote web app)</a><br/>
</li>
<li>
    <a target="_blank" href="http://blog.thepete.net/blog/2012/06/24/writing-your-first-frank-test/">Writing Your First Frank Test(搭建Frank和Symbiote)</a>
</li>
</ol>

<h5>iOS App UI调试工具（二）里介绍另一神器：RDP</h5>