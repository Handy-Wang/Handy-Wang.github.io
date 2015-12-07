---
layout: post
title: "2015年我都干嘛了"
date: 2015-12-06 11:33:18 +0800
comments: true
categories: 
---
2015年即将结束，那这一年我到底都干嘛了？

	从去年(2014)的12月份就开始暗无天日的加班忙碌，直到今天下半年才好了很多，眼看这就要要2016了，
	也该回顾一下2015年的工作了。

####AutoLayout
2015年12月，加入了一个新团队－嗨购，仍然是负责iOS项目开发，人员上加上我两人，版本处于0.0.1版本。

当时接收项目后，发现这个项目的界面采用storyboard+autolayout开发，另外那个哥们去AutoLayout的使用也处于学习阶段，所以遇到的主要问题就是修改界面的Bug。由于我之前没有使用过storyboard和autolayout，再加上UI设计师对界面的细节控制要求很高，所以有些应付不了了。在另一个哥们儿的辅助下，将就着开发了0.1.0版本(仍采用storyboard+autolayout)，其实这个过程非常苦恼，“是好好学习一下AutoLayout，然后在项目里继续使用AutoLayout呢？” 还是 “采用自己熟悉的纯代码方式来编写？”
<!--more-->
最后，综合考虑项目的进度压力、质量的可控性、学习的时间成本后，决定还是暂时放弃Storyboard+AutoLayout，转而使用保守的纯代码方式开发。

在0.1.0版本后，恰逢元旦，另外那个哥们儿休假半个月左右，回家结婚去了。我就逮着这个时机，在项目0.2.0时把项目界面实现方式、网络层、数据库等进行了大重构。

现在相对不忙碌了，计划补补 Todo: [AutoLayout](https://developer.apple.com/library/mac/documentation/UserExperience/Conceptual/AutolayoutPG/index.html)以及 Todo: [Masonry](https://github.com/SnapKit/Masonry).

####CocoaPods
其实，上面提到AutoLayout被我弃用后，就感觉比较反人类了，然后我仍然还要反人类一次。

嗨购项目在0.2.0及之前，对第三方库的依赖方式是通过CocoaPods来管理的，但是为什么这之后我把CocoaPods去掉了呢？这就不得不提到Jenkins(我会在后面聊聊Jenkins)和CocoaPods的代码管理。

本来采用Jenkins对iOS项目打包发布是一件很NB的事情，但是对通过CocoaPods管理第三库的iOS项目打包是一件非常痛苦的事情。说起痛苦，主要是集中在Jenkins上的配置以及项目本身的一些配置，如：Jenkins里build目录的指定、项目中schema的share等问题。这一系列问题我都有记录，这里就不细说了。虽然痛若，但是经过两三天的努力，可以通过Jenkins打包CocoaPods管理的项目。

感觉这一点还好，但是，没有想到最后让我放弃CocoaPods的原因尽是它的优点：无感知管理项目的第三方库依赖。
因为，

1. 第三方库难免有Bug，所以我也经常修改这些源码里Bug，修改地方多了后也就忘了修改过哪些地方了。然后有一天我pod update了一下，如果本地的源码修改没有了，我得从项目的SVN上对比着再被回来，当时非常恼怒。。。
2. 由于天国的各种墙，安装cocoapods时我修改了gem源为https://rubygems.org/ . 但是，没有想到在某几天pod update和pod install根本用不了，导致严重影响了我预期的开发进度。

所以，我又选择了保守的方案，删除cocoapods，转而采用源代码依赖的方式，这样一来Jenkins打包也简化了。

	现在回想，关于那段配置Jenkins打包CocoaPods项目的经历非常可贵，那是对意志的磨砺。
	
* **相关知识**
* [Jenkins打包CocoaPods项目](https://www.evernote.com/l/AFkw93ypG4JB9Ka-cwCPgU-xtTWZx9RTEa4)
* [更换CocoaPods的镜像](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)

####Jenkins
刚到嗨购的时候，发布安装包的方式是archive ipa文件并上传到fir.im进行发布，以供其它人员进行安装试用或测试。随着项目的推进，期望把这种Inhouse Release的方式自动化，所以引入之前在搜狐时就一直采用的Jenkins进行自动打包发布。

关于Jenkins的安装我简要提一下，主要还是回顾一下使用过程中遇到的问题

**Jenkins环境**

* 下载Jenkins.war，然后启动java -jar jenkins.war
* Jenkins的相关目录
	* /Applications/Jenkins - which is having the jenkins.war alone.
	* /Users/Shared/Jenkins - with all the plugins, war, jobs, usercontent, etc.
	* /Users/Shared/Jenkins/Home - JENKINS_HOME
	* /Library/LaunchDaemons/org.jenkins-ci.plist - Jenkins自启动文件
	* /Library/Application Support/Jenkins/jenkins-runner.sh - 自启动shell脚本


**Jenkins+CocoaPods项目**

* 参见[我的Evenote Sharing](https://www.evernote.com/l/AFkw93ypG4JB9Ka-cwCPgU-xtTWZx9RTEa4)

**七牛空间**

* 参见[我的Evenote Sharing](https://www.evernote.com/l/AFkw93ypG4JB9Ka-cwCPgU-xtTWZx9RTEa4)

**打包状态邮件通知**

* 在Jenkins里安装Mail Notification Plugin
* 以126邮件为例，配置SMPT服务器及端口
* 注意：Jenkins管理员邮箱要与发送邮件的账号一致，不然会发送邮件失败。
* 在具体项目中，可以配置相应的收件人、trigger等。

####Redmine

在2008年刚参加工作的时候，老东家就有搭建Redmine用于项目管理、issue跟踪、Wiki书写。在搜狐的时候，是由服务器同学搭建的trac来写wiki。由于刚到嗨购团队的时候，接口文件都是服务器同学通过word文档来记录并分发给相关开发人员。我觉得这样非常的不方便，不但不方便更改，也不便于分发。所以，我偷摸地在公司内网搭建了Redmine，给相关技术人员分发了账号和权限，从而以后**“妈妈再也不用担心我们的接口文档了”**。

	Todo: 也有朋友在git上写项目的wiki，改天一定要试下
	
到目前为止，我采用过三种搭建Redmine的方法：

1. 纯手工24K金方式打造Apache+MySQL+ROR
	* 完全手工地安装Apache、MySQL、RubyOnRails环境
	* 此法太虐心，会遇到各种东西没有安装，非常煎熬
2. 采用BitNami一键式安装Redmine
	* 采用BitNami安装的Redmine，在一个单独的目录下安装了Apache、MySQL等环境。
	* BitNami安装Redmine后，操作系统里安装的Apache、MySQL的环境变量被修改了
	* 为了安装一个Redmine，把操作系统里本身配置的一些环境变量给搞乱，不好不好。
3. 通过Redmine项目源码布署
	* Redmine本身是一个ROR项目，再加上09年左右的时候做过一个ROR项目，所以此法可试。
	* 下载Redmine源码: http://www.redmine.org/projects/redmine/wiki/Download
	* 解压Redmine到某个目录
	* 配置数据库连接，并创建或导入Redmine数据库表
	* 在Redmine项目运行sudo ruby bin/rails server webrick -e production来启动项目，从而抛弃Apache
	* **此法最好，对于我来说最简单直接，也符合程序思维(建立数据库、配置数据连接、部署项目源码并运行)**

####Push Notification Simulator

Todo:

####Crashlytics

Todo:

####嗨购项目架构演进概要

Todo: