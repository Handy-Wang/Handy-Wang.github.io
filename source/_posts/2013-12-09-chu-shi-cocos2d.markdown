---
layout: post
title: "初识Cocos2d"
date: 2013-12-09 21:26:00 +0800
comments: true
categories: [Cocos2d]
---

<!--more-->

俗话说『山不转水转，水不转人转，人不转需求转，需求不转代码转』，好扯，总之，各种机缘巧合已经让我无法逃离地要去了解一下这个该死的让人望而生畏的Cocos家族。

记忆中从去年开始Cocos2d这个字眼时不时的出现在互联网上、同事谈论中，打那时起我也就只知道他是跟游戏开发相关，毕竟开发```搜狐新闻客户端```用不到它，而且在我这种长期做应用型开发的
苦逼攻城师的认识里开发游戏是一件既高级又复杂的事情，渐渐对它产生了距离感和畏惧心理。

往往越是担心发生的事情，其实它早已在某个阴暗的角落里萌芽、长大。终于，我要直面它了。

Cocos2d是Ricardo Quesada团队在2008年2月基于Python开发的开源2D游戏框架。

{% blockquote Cocos2D http://cocos2d.org/index.html Read on %}
Cocos2d is a framework for building 2D games, demos, 
and other graphical/interactive applications. 
It is built over pyglet. It provides some conventions 
and classes to help you structure a "scene based application"
{% endblockquote %}

同年6月该团队OC版的```Cocos2d-iPhone```也面世。好敏捷呀，NB吧？有两点我没有想到，第一）Python语言，第二）开发速度

就在大家还在感叹```Cocos2d-iPhone```多么NB多么方便时，一个中国人，请注意他叫```王哲```干了一件让所有用Cocos2D开发游戏的小伙伴们都惊呆的事儿。2010年，这哥们儿找到了兴趣、
擅长，和商业的结合点－基于Cocos2d-iPhone架构C++语言的跨平台的Cocos2d-x```http://www.cocos2d-x.org/```版问世

{% blockquote 王哲：爱偷懒+爱游戏=开源Cocos2d-x的生命基因 http://www.ituring.com.cn/article/42393 Read on %}
你是怎么开始开发Cocos2d-x的？

后来我和一个夏新同事，也就是现在带cocos2d-html5项目的林顺，一起去厦大读商学院，边读边思考管理、技术上的问题。在2010年，我们去了国内一家做操作系统的公司，我负责的那个部门是多媒体游戏。为什么去做游戏？因为整个公司里就我自己最喜欢玩游戏，而别人又不喜欢，所以我就开始入手了。当时我就想有没有一种简单的方式让别人把游戏移植过来，于是我想做一个引擎，能够方便地把iOS游戏移植到我们操作系统上面，顺便再出一个安卓版本。
{% endblockquote %}

不得不承认这哥们儿是个实干家！之后，依托于陈昊芝以及他的```CocoaChina```下的Cocos2d-x的社区和```触控科技```，Cocos2d-x得到了青春期最好的发育，到目前Cocos2d-x已广泛应用到
游戏开发中，已经到了那种如果你不用Cocos2d-x甲方就感觉乙方不专业的程度。当前，围绕Cocos2d-x的技术产品也逐渐诞生：Cocos2d-js、Cocos2d-html5、CocosStudio、Plugin-x等。

最后上一张图吧，辅助对Cocos2d家族的理解。
{% img https://raw.github.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/cocos2d-x_hierarchi.png %}

有一天，也许我也会搞搞Cocos2d-x游戏开发。。。呵呵