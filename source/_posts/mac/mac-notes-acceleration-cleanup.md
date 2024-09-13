---
title: Mac笔记之加速和清理
author: Zeeny
comments: false
date: 2014-08-29
updated: 2014-08-29
tags: [Mac]
categories: [Mac]
summary: Mac笔记之加速和清理
---


# Mac笔记(三)之加速和清理

__1. 禁用Dashboard__

```
如果你经常使用的话，dashboard固然好。然而，你也要清楚，每个widgets 和 web clips都非常占内存和资源。怎样禁止？ 方法也很简单。 

打开Terminal 敲入以下命令: 
defaults write com.apple.dashboard mcx-disabled -boolean YES 

然后你可以重启macbook或者敲入以下命令： 
killall Dock 

当你又需要dashboard的时候，以下命令可以逆转： 
defaults write com.apple.dashboard mcx-disabled -boolean NO 

再重启或敲入 : 
killall Dock 
```