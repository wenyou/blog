---
title: Mac安装rails失败解决办法
author: Zeeny
comments: false
date: 2014-03-23
updated: 2014-03-23
tags: [Mac,ruby]
categories: [Mac,ruby]
summary: Mac调用sudo gem install rails安装rails失败解决办法
---


# Mac调用sudo gem install rails安装rails失败解决办法

<p>环境：</p>
* mac OS X 10.9.2
* ruby 2.0.0p247
* gem 2.2.2

<p>1、错误1，第一次执行：sudo gem install rails 安装rails最新版：</p>

	Building native extensions.  This could take a while...
	/System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby 	extconf.rb
	mkmf.rb can't find header files for ruby at /System/Library/Frameworks/	Ruby.framework/Versions/2.0/usr/lib/ruby/include/ruby.h
	ERROR:  Error installing rails:
	ERROR: Failed to build gem native extension.

  Building has failed. See above output for more information on the failure.
	extconf failed, exit code 1

	Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/atomic-1.1.16 for inspection.
	Results logged to /Library/Ruby/Gems/2.0.0/extensions/universal-darwin-13/2.0.0/atomic-1.1.16/gem_make.out


<p>解决方式：安装XCode最新版,我通过App Store安装的版本是 XCode Version 5.1 (5B130a)。 或者在mac终端下输入：xcode-select --install，根据提示安装xcode工具。</p>

<p>安装后，再次继续执行：sudo gem install rails.</p>

<p>2、错误2，如下所示：</p>

	Building native extensions.  This could take a while...
	ERROR:  Error installing rails:
	ERROR: Failed to build gem native extension.

    /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/bin/ruby extconf.rb
	checking for libkern/OSAtomic.h... yes
	creating Makefile

	make "DESTDIR=" clean

	make "DESTDIR="
	compiling atomic_reference.c
	atomic_reference.c:57:59: warning: incompatible pointer types passing 'void **' to parameter of type 'volatile int64_t *' (aka 'volatile long long *') [-Wincompatible-pointer-types]
    if (OSAtomicCompareAndSwap64(expect_value, new_value, &DATA_PTR(self))) {
                                                          ^~~~~~~~~~~~~~~
	/usr/include/libkern/OSAtomic.h:507:93: note: passing argument to parameter '__theValue' here
	bool    OSAtomicCompareAndSwap64( int64_t __oldValue, int64_t __newValue, volatile int64_t *__theValue );
                                                                                            	^
	1 warning generated.
	linking shared-object atomic_reference.bundle
	clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-	command-line-argument-hard-error-in-future]
	clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
	make: *** [atomic_reference.bundle] Error 1

	make failed, exit code 2

	Gem files will remain installed in /Library/Ruby/Gems/2.0.0/gems/atomic-1.1.16 for inspection.
	Results logged to /Library/Ruby/Gems/2.0.0/extensions/universal-darwin-13/2.0.0/atomic-1.1.16/gem_make.out


<p>我的解决办法是，在mac终端通过执行：sudo ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future gem install rails 命令的方式安装，最终成功安装上rails。</p>

<p>安装后检查：rails -v  => Rails 4.0.4</p>
