启动命令:
cd /home/myNode
#生成静态页面
hexo generate / hexo g  

#清空静态文件（public目录）
hexo clean   

#部署
hexo deploy / hexo d

#生成静态页面 + 部署
hexo generate --deploy / hexo g -d

#启动hexo服务
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


让hexo进入后台运行: https://blog.csdn.net/weixin_41256398/article/details/117989926
pm2查看、重启服务和日志:https://blog.csdn.net/wzp20092009/article/details/122432243



Hexo 文档
  https://hexo.io/zh-cn/docs/
  Hexo 搭建博客笔记: https://blog.csdn.net/Leatitia/article/details/103213578


安装 Hexo
1、全局安装Hexo
  npm install -g hexo-cli
  安装后使用方法：
    hexo <command>
    eg: hexo n test   //生成文章 test.md

2、局部安装Hexo
  npm install hexo
  安装后使用方法：
    npx hexo <command>
    eg: npx hexo n test   //生成文章 test.md



Hexo写文章操作方法参考:
  如何用Hexo优雅的书写文章 https://blog.csdn.net/howareyou2104/article/details/106312703
  使用hexo新建、编辑并预览文章 https://zhuanlan.zhihu.com/p/132823826
  