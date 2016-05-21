---
layout: post
title: "UITextView &amp; UITextField的九宫格对齐方式"
date: 2016-05-04 21:27:05 +0800
comments: true
categories: [UIKit]
---

<!-- more -->

#目标

iOS原生的 UITextField 和 UITextView 只支持文本内容的左、中、右对齐，但是目前我的需求是需要让UITextField 和 UITextView 支持9宫格对齐方式，并能按此对齐方式进行正常输入，如下图：

![目标](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/9point_alignment_textfield_textview1.png "目标")

#知识铺垫

iOS7之后，UILabel / UITextField / UITextView的实现采用TextKit进行了替换，所以对于这个命题我主要需要关注TextKit. 
下面以一张图来说明TextKit内部的关键类与UITextView的逻辑关系. 

![TextKit类的协作](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/9point_alignment_textfield_textview2.png "TextKit类的协作")

* 说明
  * NSTextStorage 存储着文本的样式信息且是NSAttribute的子类，可以把它近似看成NSAttributeString. NSTextStorage关联一个NSLayoutManager.
  * NSLayoutManager  负责布局动作的类，把文本按其样式信息放到NSTextContainer里进行布局排版. NSLayoutManager关联一个NSTextContainer.
  * NSTextContainer 提供给NSLayoutManager用于排版的布局空，当然这肯定是根据文本信息计算出来的. 图中的黑块部分就是最终文本被填充上去的区域.
  * UITextView 以上三步都是计算的过程，从NS打头可以看出去显示无关。UITextView则提供了显示的空间，继承自UIScrollView，它包含了一个NSTextContainer用于显示以上三步计算出来的结果. 图中的黑块部分同上，蓝色部分是UITextView与NSTextContainer之间的inset，即textContainerInset .

* 综上
  * 有了以上对TextKit与UITextView关系的初步了解后，要做到目标图中UITextView的样式，纵向上需设置textContainerInset的top和bottom，横向上需要设置UITextView的textAlignment即可。
  * UITextField要做目标图中的样式，

#源码

加微博 @我是小山我是坏人

#参考
* [Text Kit入门](http://esoftmobile.com/2013/10/17/text-kit%E5%85%A5%E9%97%A8/)
* [Text Kit进阶](http://esoftmobile.com/2013/10/17/text-kit%E8%BF%9B%E9%98%B6/)
* [第 9 章　iOS 7中文字排版和渲染引擎——Text Kit](http://www.ituring.com.cn/tupubarticle/2542)
* [Text Kit Tutorial](https://www.raywenderlich.com/50151/text-kit-tutorial)
* [Cocoa Text System everywhere…](http://orangejuiceliberationfront.com/cocoa-text-system-everywhere/)
* [How to lose margin/padding in UITextView](http://www.howwaydo.com/how-to-lose-marginpadding-in-uitextview/)
* [How to get rid of the padding / insets in an UITextView](http://www.pixeldock.com/blog/how-to-get-rid-of-the-padding-insets-in-an-uitextview/)
* [Text Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/Introduction/Introduction.html#)