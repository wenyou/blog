---
layout: post
title: RubyOnRails常见错误解决办法
description: Ruby On Rails常见错误解决办法
category: ruby
tags: [rails,ruby]
---
###Ruby On Rails常见错误解决办法###
<p>1、错误1，当执行rails server时，出现如下错误：</p>

	/usr/local/lib/ruby/gems/2.0.0/gems/execjs-2.0.2/lib/execjs/runtimes.rb:51:in `autodetect': Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
	
<p>原因：没有js运行环境，解决方法：</p>

	vim Gemfile
		gem 'execjs'
		gem 'therubyracer'
	bundle install
	rails server
	
<p>打开Gemfile添加gem 'execjs'，gem 'therubyracer'，然后安装bundle install，再执行rails server，就可以了。</p>

<p>2、错误2，Mac下当执行rails server时，出现如下错误：</p>

	dyld: lazy symbol binding failed: Symbol not found: _mysql_get_client_info	Referenced from: /Users/zxsoft/.rvm/gems/ruby-2.1.1@global/extensions/x86_64-darwin-13/2.1.0/mysql2-0.3.15/mysql2/mysql2.bundle
	Expected in: flat namespace
	
	dyld: Symbol not found: _mysql_get_client_info
	Referenced from: /Users/zxsoft/.rvm/gems/ruby-2.1.1@global/extensions/x86_64-darwin-13/2.1.0/mysql2-0.3.15/mysql2/mysql2.bundle
	Expected in: flat namespace
	
	Trace/BPT trap: 5
	
<p>已经使用gem install mysql2 安装过mysql2了。</p>
