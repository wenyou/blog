---
title: SVN使用记录
date: 2015-03-02
updated: 2015-03-02
categories: [计算机,开发工具]
tags: [SVN]
author: Zeeny
summary: SVN使用记录
comments: false
---



# SVN使用记录


### 1、SVN Skipped 'xxx' -- Node remains in conflict

```
今天svn提交发现错误

#cd /home/svn/app/

# svn up

	Updating '.':
	Skipped 'xxx' -- Node remains in conflict
	At revision 1054.
	Summary of conflicts:
	Skipped paths: 1


处理方式：

#cd /home/svn/app/
#mkdir /root/bak

#mv /home/svn/app/* /root/bak/

#svn revert .

#svn up


如果还是不可以，出绝招：

svn remove --force filename
svn resolve --accept=working  filename
svn up
```