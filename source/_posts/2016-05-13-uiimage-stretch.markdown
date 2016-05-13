---
layout: post
title: "UIImage stretch"
date: 2016-05-13 15:05:40 +0800
comments: true
categories: 
---

## 一张可拉伸图片的定义－译自[iOS原文](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/#Defining a Stretchable Image)
<br />
一张可拉伸的图片其实给自己定义了一个区域，在这个区域内的图片内容以一种更美观优雅的方式被重复显示。可拉伸图片通常用于视图的背景，因为这种图片可以按拉伸区域的定义被撑大或缩小，从而以一种更美观的方式来填充视图区域。

<!-- more -->

<br />
iOS提供了 `resizableImageWithCapInsets:`、`resizableImageWithCapInsets:resizingMode:` 两个方法来指定图片的拉伸区域。方法中的`insets`参数(insets = (top,left,bottom,right))把图片细分为了两个或多个区域，其中的top/left/bottom/right称为inset。给insets中的每个inset指定一个非零值，就可以把图片细分为9个区域，如下图。
<br />
> <img src="https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch1.png" width="610.5" height="208" />

<br />
从上图可以看出，每个inset的值给图片指定了一个不可以拉伸的区域。通过top inset和bottom inset在图片的上边和下边划定了两块不可拉伸的区域，通过left inset和right inset在图片的左边和右边划定了两块不可拉伸的区域。通过下图可以看出，当图片被拉伸时，这9块区域是怎么被拉伸的。显然四个角是没有被拉伸的，因为它们同时处于横向和纵向的inset区域。

<br />
> <img src="https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch2.png" width="486" height="153" />

***

## 图片保护区域

基于上面Apple对图片拉伸的描述，下面从实例例子出发来讲讲图片的保护区域

<br />
开发中常常需要对图片进行拉伸操作，如果控件的宽高比例和要显示的图片的宽高比例不同的话，图片将不会按照自身宽高比进行拉伸或者收缩，导致最终显示的图片效果变差，因此常常需要对图片进行特殊处理(按一定规则进行拉伸或者收缩)。为了使得拉伸后的图片具有美观的效果，这里要提到一个“保护”或者英文“cap”的概念。
为了保护图片在拉伸的过程中保持美观，因此我们需要知道图片要被保护的区域，即图片的上下左右各要保护多少，这样在拉伸的时候就会只拉伸非保护区域，即用非保护区域的像素来填充视图的拉伸空间。最终，水平拉伸的填充取决于左右保护区域的多少，垂直方向的填充取决于上下保护区域的多少。

### 用于描述图片保护区域的字符串格式

字符串内包含了上、左、下、右四个方向需要保护的大小的值：“topValue,leftValue,bottomValue,rightValue”，也可简写为"value"，"value" 等价于"value, value, value, value"，即上左下右四个方向需要保护的尺寸是一样的。

如：“20,40,20,40”，"40"，"40" = "40,40,40,40"

### 示例

以一个宽、高分别为"130, 107"的图片为例，设置图片保护区域为： "40, 40, 40, 40"

* 原图

> ![image_stretch3.png](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch3.png)

* 保护区域、可拉伸区域图

保护区域为：右则九宫格图四个角的区域

非保护区域为：右则九宫格图的“井”字区域

所以，最后拉伸后的效果如下图左侧模拟器里。

> ![image_stretch4.png](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch4.png)

## iOS Xcode里的image slicing功能

看完以上内容已经完全可以了解stretch的概念了，那么既然已经看到这里了，我再多聊聊`iOS Xcode里的image slicing功能`。

<br />
Xcode5开始支持Image Slicing功能，即通过可视化的方式给图片指定CapInsets。一张图片通过这个功能设定好CapInsets后，在代码里不用调用上面提到的两个方法来指定保护区域，直接加载图片就可以按设定的CapInsets来拉伸图片，对于Image Slicing功能的使用很简单就不多说了。

<br />
下面才是我想说的主要内容。仍以上面为例｀以一个宽、高分别为"130, 107"的图片为例，设置图片保护区域为： "40, 40, 40, 40"｀

<br />
Image Slicing功能默认是把可拉伸区域定义为1x1日区域(如下图红色画框部分)，运行的图片效果是左侧模拟器里上面一张黑图。左侧模拟器里下面一张黑图是通过上面提到两个SDK方法指定的保护区域，显然这两种方式运行出来的图片效果不一致。这是因为SDK方法指定的保护区域算出来的拉伸区域为50x27，即：130-2*40-50，107-2*40=27。

<br />
![image_stretch5.jpg](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch5.jpg)

<br />
想要通过Image Slicing功能划分保护区域的图片的最终显示效果与SDK划分的一致，只需要修改Image Slicing里的拉伸区域大小(如下图)，这同时也说明了在Image Slicing功能里，虽然划定了保护区域的但仍可以调整拉伸区域的大小，而SDK方式的拉伸区域的大小是计算出来的。

![image_stretch6.jpg](https://raw.githubusercontent.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/img/image_stretch6.jpg)

完结。

## 参考

* [Defining a Stretchable Image](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/)
* [WWDC 2013 Session笔记 - Xcode5和ObjC新特性](https://onevcat.com/2013/06/new-in-xcode5-and-objc/)