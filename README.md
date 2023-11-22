# blog
我的博客，笔记
### 访问网址
* [http://stkit.cn/](http://stkit.cn)

### hexo command
```
hexo g
hexo d
````

### 配置
```
安装插件:
npm install hexo-wordcount --save
npm install hexo-generator-json-content --save
npm install hexo-generator-feed --save
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save

生成、部署
cd /home/myNode
hexo generate / hexo g
hexo deploy / hexo d
hexo generate --deploy / hexo g -d
hexo server -p 80

后台运行hexo 网站
cd /home/myNode
pm2 start run.js
pm2 stop run.js

pm2 查看当前启动的所有应用
 pm2 list
运行起来后可以使用pm2 show + 进程pid查看后台hexo进行
 pm2 show 0
pm2 查看日志 
 pm2 log
 pm2 log [project]

centos查看进程
ps -elf|grep hexo
或者：
ps aux|grep hexo
```