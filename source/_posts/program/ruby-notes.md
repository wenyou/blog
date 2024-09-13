---
title: Ruby笔记
author: Zeeny
comments: false
date: 2014-03-25
updated: 2014-03-25
tags: [ruby]
categories: [ruby]
summary: Ruby笔记
---


# ruby 笔记

## 安装 mysql：

* wget http://www.cmake.org/files/v2.8/cmake-2.8.4.tar.gz
* tar zxvf cmake-2.8.4.tar.gz
* cd cmake-2.8.4
* ./configure
* make && make install
* mkdir /usr/local/mysql  => 创建mysql安装目录
* /usr/sbin/groupadd mysql
* /usr/sbin/useradd -g mysql mysql
* mkdir /var/mysql  => 创建数据目录
* chown mysql:mysql -R /var/mysql
* wget http://cdn.mysql.com/Downloads/MySQL-5.5/mysql-5.5.37.tar.gz
* tar zxvf mysql-5.5.37.tar.gz
* cd mysql-5.5.37
* cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql/ -DMYSQL_DATADIR=/var/mysql -DMYSQL_UNIX_ADDR=/var/mysql/mysqld.sock -DWITH_INNOBASE_STORAGE_ENGINE=1 -DENABLED_LOCAL_INFILE=1 -DMYSQL_TCP_PORT=3306 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DMYSQL_UNIX_ADDR=/var/mysql/mysql.sock -DMYSQL_USER=mysql -DWITH_DEBUG=0
* ps:(Myslq 5.5.9以上版本编译出现错误汇总：CMake Warning: The variable, 'MYSQL_USER', specified manually, was not used during the generation. 需要把预编译里面的MYSQL_USER去掉，即可预编译成功！)
* make && make install
* cp support-files/my-medium.cnf /etc/my.cnf =>复制配置文件
* cp support-files/mysql.server /etc/init.d/mysqld  =>复制启动脚本
* chmod 755 /etc/init.d/mysqld
* cd /usr/local/mysql/  进到安装目录
* ./scripts/mysql_install_db --user=mysql --ldata=/var/mysql =>初始化数据库
* /etc/init.d/mysqld start => 启动数据库服务器


## 安装rvm 安装ruby
* curl -sSL https://get.rvm.io | bash -s stable =>安装rvm
* 安装完成rvm后，依次执行：source ~/.bashrc ，source ~/.bash_profile ，或重新开启终端，来使rvm环境生效。
* 修改 RVM 的 Ruby 安装源到国内的淘宝镜像服务器，这样能提高安装速度：sudo sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db
* rvm list known => 列出已知的ruby版本,列表里面的都可以拿来安装。* 
* rvm install ruby-2.1.1 =>安装ruby 2.1.1
* rvm default 2.1.1 (rvm 2.1.1 --default) =>设置默认使用ruby 2.1.1
* rvm use 2.0.0 =>设置使用ruby 2.0.0
* rvm list => 查询已经安装的ruby
* rvm remove 2.0.0 => 卸载一个已安装版本
* rvm remove ruby-1.8.7 --docs --gems  => 删除某个版本的ruby，并且把文档和gems都删除。


## 安装gem
* wget http://production.cf.rubygems.org/rubygems/rubygems-2.2.2.tgz
* tar zxvf rubygems-2.2.2.tgz
* cd rubygems-2.2.2
* ruby setup.rb
* 
* gem sources --remove https://rubygems.org/    => 删除rubygems.org
* gem sources -a http://ruby.taobao.org   =>使用ruby.taoboa.org
* gem sources -l  =>查看


## 安装rails
* 安装对应版本的rails，我这里是ruby2.0，我就安装rails4.0作为ruby2.0的rails版本：rvm 2.0.0 exec gem install rails -v4.0 或者 sudo rvm 2.1.1 exec gem install rails

* 多个版本切换使用问题，比如我现在安装了ruby1.9.3，对应的rails是3.2.12，同时也安装了ruby2.0.0，对应的rails版本是4.0，我现在想使用rails3来创建一个rails项目，该怎么办？使用 rvm 1.9.3 exec rails new project 就可以创建一个使用rails3.2.12版本的rails项目了。

* gem install rails =>  安装rails

* rvm 2.1.1 exec rails new ./zxpm --database=mysql 或 rails new ./zxpm --database=mysql  => 生成应用程序并指定使用mysql数据库。

* rails new demo --skip-bundle --skip-test-unit -d mysql  => 生成一个rails项目，后面的参数表示不安装bundle。例：rvm 2.1.1 exec rails new ./zxpm --skip-bundle --skip-test-unit --database=mysql 。

* 修改gem源地址以提高bundel install速度，cd ./zxpm，vi Gemfile，source 'http://ruby.taobao.org/'，bundle install --local --without production。

* Bundle (Rails 项目)时，vi Gemfile，source 'http://ruby.taobao.org/'。

* rails server [-p 3001] => 加上-p是指定使用的端口号


## 安装 passenger  nginx
* rvm 2.1.1 exec gem install passenger
* /usr/local/rvm/gems/ruby-2.1.1/gems/passenger-4.0.40/bin/passenger-install-nginx-module
* /opt/nginx/sbin/nginx 启动nginx
* ./nginx -s stop 关闭
* ./nginx 启动
* vi /opt/nginx/conf/nginx.conf

		location / {
            root   /var/www/zxpm/public;
            index  index.html index.htm;
            passenger_enabled on;
            autoindex  on;
            rails_env development;
        }


## 其他
* rvm info => 查询当前版本。
* gem list => 可以用gem list来查看自己安装了那些gem包。
* gem install --no-rdoc --no-ri mysql2 => 带上 --no-rdoc --no-ri 不安装文档。


## 技巧和出错解决
<p>ps:1，在使用rvm安装ruby的时候由于国内被墙，直接这样安装不成功，解决方法是：进入rvm目录下的 archives目录，cd ~/.rvm/archives，手动下载对应的ruby版本，下载格式为tar.bz2格式的，一般使用curl在ruby.taobao.org下载。eg:curl http://ruby.taobao.org/mirrors/ruby/2.0/ruby-2.0.0-p247.tar.bz2 > ruby-2.0.0-p247.tar.bz2，下载完之后继续使用 rvm install ruby-2.0.0命令就可以安装ruby2.0了。</p>

<p>ps:2，使用rvm use 2.1.1 或 rvm 2.1.1 --default 出现如下错误：</p>

	Gemset '' does not exist, 'rvm ruby-2.1.1 do rvm gemset create ' first, or append '--create'.
	
<p>解决方法：rvm use 2.1.1@newgemset --create --default</p>

<p>ps:3，使用gem install passenger 或 rvm 2.1.1 exec gem install passenger 出现如下错误：</p>

	ERROR:  While executing gem ... (Gem::UnsatisfiableDependencyError)
    Unable to resolve dependency: 'passenger (= 4.0.40)' requires 'daemon_controller (>= 1.2.0)'

<p>解决方法：[rvm 2.1.1 exec ]gem install daemon_controller</p>

<p>ps:4，使用passenger-install-nginx-module，出现如下错误：</p>

	Error: Cannot find the `curl-config` command.
	
<p>解决方法：yum install curl curl-devel</p>


### gemset的使用
* [http://ruby-china.org/wiki/rvm-guide](http://ruby-china.org/wiki/rvm-guide)


### tips

	sudo env ARCHFLAGS="-arch x86_64" gem install --no-rdoc --no-ri mysql2 -- --with-mysql-config=/usr/local/mysql/bin/mysql_config

	sudo gem install --no-rdoc --no-ri mysql2 -- --with-mysql-dir=/usr/local/mysql-5.5.31-osx10.6-x86 --with-mysql-include=/usr/local/mysql-5.5.31-osx10.6-x86/include --with-mysql-lib=/usr/local/mysql-5.5.31-osx10.6-x86/lib --with-mysql-config=/usr/local/mysql-5.5.31-osx10.6-x86/bin/mysql_config
