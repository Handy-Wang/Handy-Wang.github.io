---
layout: post
title: "Runtime（一）之 Message 浅析"
date: 2014-03-10 15:52:57 +0800
comments: true
categories: [NSObject, Message, Runtime]
---

<h3>Objective-C对象模型</h3>
本文假定您对Objective-C对象模型已了解
{% img https://raw.github.com/Handy-Wang/Handy-Wang.github.io/source/source/_posts/class-diagram.jpg %}


<h3>主题</h3>
<h6>一）方法的动态决议</h6>
<h6>二）消息转发</h6>
<!--more-->
	
<h3>一）方法的动态决议</h3>

即程序运行时动态提供方法的实现，编译时是没有方法的实现的。这一机制与消息机制不冲突，即发送消息后，如果运行时发现接收者没有消息的实现，
则会触发方法<br/>
+(BOOL)resolveInstanceMethod:(SEL)sel; 和 +(BOOL)resolveClassMethod:(SEL)sel;<br/>
以拦截需要"动态决议"的方法，显而易见的是这两个方法分别对应实例方法和类方法的调用拦截。

针对“动态决议“的“编译时是没有方法的实现”这一特性，我总结了两类动态决议的实际场景：<br/>
1）@dynamic<br/>
与@synthesis一样用于修饰属性，但@synthesis用于生成确实名称的getter、setter方法（如果@property中没有特殊指定getter、setter方法名）；@dynamic则是告诉编译器此属性的getter、setter方法的实现在运行时动态指定。<br/>
2）调用没有实现的实例方法或静态方法<br/>
因为编译时消息接收者是没有方法实现的，所以在运行时上面提到的两个方法一定能被触发。

重点总结：<br/>
1）一定是被调用方法（静态方法和实例方法）的实现在编译时不存在，则“方法的动态决议机制”才有效，否则无效<br />
2）属性方法的动态决议可通过@dynamic修饰属性以及配合方法 +(BOOL)resolveInstanceMethod:(SEL)sel; 来指定方法的实现<br />
3）通过runtime相关函数提供方法的动态实现时，如果方法的实现有返回值，那么返回值必须是对象类型不能为基本数据类型(通过代码实验证明)<br />

知识扩展<br />
使用runtime相关函数在运行时添加静态和实例方法实现时，发现一些小知识点。<br />
1）对象实例、类实例、实例对象、类对象
	对象实例 通过类(或自己定义的类)实例化后得到的对象，如:AboutMethods *am = [[[AboutMethods alloc] init]]; 
		am就是对象实例；类的实现是通过objc_object结构体来实现的，objc_object中命名为object,一方面说明类本身是对象
		另一方面说明对象实例是objc_object对象实例化的结果
	类实例 类本身就是对象(objc_object)，它的内部isa指向了描述它的数据结构objc_class 称之为metaclass，
		也就是说class类(objc_object)是metaclass类(objc_class)实例化的结果。
		从objc_class名称上看出它才是真正的class，
		所以我通常认识的类实际是objc_class类的实例，所以叫类实例
	实例对象 实例对象即实例的对象也就是类实例 class
	类对象 类对象即类的对象也就是metaclass
2）self、[self class]、[[self class] class]、[类名 class]、[[类名 class] class]、objc_getMetaClass['类名']、self->isa、[类名 class]->isa、self->isa->isa的区别：
	要区分它们之前，首先要考虑调用它们的上下文：实例方法里和静态方法里， 以下分析假设有一个类 名为AboutMethods
		如果是静态方法里调用：
			1.self 指实例对象AboutMethods class
			2.[self class] 当前在静态方法内，所以[self class]访问的也是静态方法 +class 
					相当于 [AboutMethods class], 也就是说[self class] 等价 [AboutMethods class]
					即还是得到实例对象class；而不等价于self->isa 取类对象metaclass。
					所以，形如[xx class]或者嵌套方式如[[xx class] class]的对class静态方法的调用
					都是返回实例对象class，即对象实例的isa，而不是获取类对象metaclass。
			3.[[self class] class]、[类名 class]、[[类名 class] class] 如2.分析，都是获取对象实例class
			4.objc_getMetaClass["类名"] 从这个runtime函数名上即可知道这是获取类的metaclass，即类对象
					如：objc_getMetaClass("AboutMethods") 返回的是AboutMethods类的类对象metaclass，
					即类实例class的isa
			5. self->isa、[类名 class]->isa 如前面所述，self、[xxx class]都是返回实例对象，
					那么实例对象的isa就是类对象metaclass
			6. self->isa->isa 因为self->isa是metaclass，所以self->isa->isa是root metaclass
					即NSOjbect的metaclass
		如果是实例方法里调用：
			1.self 指对象实例
			2.[self class] 指类实例即实例对象class
			3.[[self class] class]、[类名 class]、[[类名 class] class] 同静态方法里，都是获取对象实例class
			4.objc_getMetaClass["类名"] 同静态方法里，即获取类对象metaclass
			5. self->isa 由于self是对象实例，所以self->isa获取的是类实例即实例对象class
			6. [类名 class]->isa 由于[类名 class]获取的是类实例即实例对象，
					所以[类名 class]->isa获取到的是类对象metaclass
			7. self->isa->isa 由于self是对象实例，所以self->isa表示获取类实例即实例对象，
					那么self->isa->isa表示是获取类对象即metaclass
			8. 注：在对象实例上直接调用isa已经被抛弃了，建议用object_getClass()函数

<h3>二）消息转发</h3>

待续。。。