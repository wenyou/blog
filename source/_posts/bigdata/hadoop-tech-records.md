---
title: Hadoop相关技术点滴记录
author: Zeeny
comments: false
date: 2015-02-25
updated: 2015-02-25
tags: [Hadoop,大数据]
categories: [大数据]
summary: Hadoop相关技术点滴记录
---


# Hadoop相关技术点滴记录

## 一、点滴技巧

* 一条记录内容的字段信息之间可以用换行符'\007'(不可显示的ASCII码)分隔；记录之间用'\r\n'(换行符)分隔,如：RUL'\007'HOT'\007'AUTHOR'\007'TITLE'\007'CONTENT'\r\n'


* hadoop入门

```
一、概论
	作为hadoop程序员，他要做的事情就是：
	1、定义Mapper，处理输入的Key-Value对，输出中间结果。
	2、定义Reducer，可选，对中间结果进行规约，输出最终结果。
	3、定义InputFormat和OutputFormat，可选，InputFormat将每行输入文件的内容转换为Java类供Mapper函数使用，不定义时默认为String。
	4、定义main函数，在里面定义一个Job并运行它。
	
	然后的事情就交给系统了。
	
	
	1.基本概念：
		Hadoop的HDFS实现了google的GFS文件系统，NameNode作为文件系统的负责调度运行在 master，DataNode运行在每个机器上。同时Hadoop实现了Google的MapReduce，JobTracker作为 MapReduce的总调度运行在master，TaskTracker则运行在每个机器上执行Task。
	
	2.main()函数
		main()函数，创建JobConf，定义Mapper，Reducer，Input/OutputFormat 和输入输出文件目录，最后把Job提交給JobTracker，等待Job结束。
	
	3.JobTracker
		JobTracker，创建一个InputFormat的实例，调用它的getSplits()方法，把输入目录的文件拆分成FileSplist作为Mapper task 的输入，生成Mapper task加入Queue。
	
	4.TaskTracker 向 JobTracker索求下一个Map/Reduce。

		Mapper Task先从InputFormat创建RecordReader，循环读入FileSplits的内容生成Key与Value，传给Mapper函数，处理完后中间结果写成SequenceFile.

		Reducer Task 从运行Mapper的TaskTracker的Jetty上使用http协议获取所需的中间内容（33%），Sort/Merge后（66%），执行Reducer函数，最后按照OutputFormat写入结果目录。

		TaskTracker 每10秒向JobTracker报告一次运行情况，每完成一个Task10秒后，就会向JobTracker索求下一个Task。

		Nutch项目的全部数据处理都构建在Hadoop之上，详见Scalable Computing with Hadoop 。
	
	
二、程序员编写的代码
	我们做一个简单的分布式的Grep，简单对输入文件进行逐行的正则匹配，如果符合就将该行打印到输出文件。因为是简单的全部输出，所以我们只要写Mapper函数，不用写Reducer函数，也不用定义Input/Output Format。
	
	package demo.hadoop
	public class HadoopGrep {
		public static class RegMapper extends MapReduceBase implements Mapper {
			private Pattern pattern;
			public void configure(JobConf job) {
				pattern = Pattern.compile(job.get( ” mapred.mapper.regex ” ));
			}
		
			public void map(WritableComparable key, Writable value, OutputCollector output, Reporter reporter) throws IOException {
				String text = ((Text) value).toString();
				Matcher matcher = pattern.matcher(text);
				if (matcher.find()) {
					output.collect(key, value);
				}
			}
		}
	
		private HadoopGrep () {
		} // singleton
	
		public static void main(String[] args) throws Exception {
			JobConf grepJob = new JobConf(HadoopGrep. class );
			grepJob.setJobName( ” grep-search ” );
			grepJob.set( ” mapred.mapper.regex ” , args[ 2 ]);
			grepJob.setInputPath( new Path(args[ 0 ]));
			grepJob.setOutputPath( new Path(args[ 1 ]));
			grepJob.setMapperClass(RegMapper. class );
			grepJob.setReducerClass(IdentityReducer. class );

			JobClient.runJob(grepJob);
		}
	}
	
	RegMapper类的configure()函数接受由main函数传入的查找字符串，map() 函数进行正则匹配，key是行数，value是文件行的内容，符合的文件行放入中间结果。

	main()函数定义由命令行参数传入的输入输出目录和匹配字符串，Mapper函数为RegMapper类，Reduce函数是什么都不做，直接把中间结果输出到最终结果的的IdentityReducer类，运行Job。

	整个代码非常简单，丝毫没有分布式编程的任何细节。
	
	
三.运行Hadoop程序
	Hadoop这方面的文档写得不全面，综合参考GettingStartedWithHadoop 与Nutch Hadoop Tutorial 两篇后，再碰了很多钉子才终于完整的跑起来了，记录如下：
	
	3.1 local运行模式
		完全不进行任何分布式计算，不动用任何namenode,datanode的做法，适合一开始做调试代码。

		解压hadoop，其中conf目录是配置目录，hadoop的配置文件在hadoop-default.xml，如果要修改配置，不是直接修改该文件，而是修改hadoop-site.xml，将该属性在hadoop-site.xml里重新赋值。

		hadoop-default.xml的默认配置已经是local运行，不用任何修改，配置目录里唯一必须修改的是hadoop-env.sh 里JAVA_HOME 的位置。

		将编译好的HadoopGrep与RegMapper.class 放入hadoop/build/classes/demo/hadoop/目录 找一个比较大的log文件放入一个目录，然后运行hadoop / bin / hadoop demo.hadoop.HadoopGrep log文件所在目录 任意的输出目录 grep的字符串
		
		查看输出目录的结果，查看hadoop/logs/里的运行日志。
		
		在重新运行前，先删掉输出目录。

	3.2 单机集群运行模式
		现在来搞一下只有单机的集群.假设以完成3.1中的设置，本机名为hadoopserver
		第1步. 然后修改hadoop-site.xml ，加入如下内容：
			< property >
			< name > fs.default.name
			< value > hadoopserver:9000

			< property >
			< name > mapred.job.tracker
			< value > hadoopserver:9001

			< property >
			< name > dfs.replication
			< value > 1

		从此就将运行从local文件系统转向了hadoop的hdfs系统，mapreduce的jobtracker也从local的进程内操作变成了分布式的任务系统，9000，9001两个端口号是随便选择的两个空余端口号。

		另外，如果你的/tmp目录不够大，可能还要修改hadoop.tmp.dir属性。

		第2步. 增加ssh不输入密码即可登陆。
			因为Hadoop需要不用输入密码的ssh来进行调度，在不su的状态下，在自己的home目录运行ssh-keygen -t rsa ,然后一路回车生成密钥，再进入.ssh目录,cp id_rsa.pub authorized_keys

			详细可以man 一下ssh, 此时执行ssh hadoopserver，不需要输入任何密码就能进入了。


		3.格式化namenode，执行
			bin/hadoop namenode -format

		4.启动Hadoop
			执行hadoop/bin/start-all.sh, 在本机启动namenode,datanode,jobtracker,tasktracker

		5.现在将待查找的log文件放入hdfs,。
			执行hadoop/bin/hadoop dfs 可以看到它所支持的文件操作指令。
			执行hadoop/bin/hadoop dfs put log文件所在目录 in ，则log文件目录已放入hdfs的/user/user-name/in 目录中

		6.现在来执行Grep操作
			hadoop/bin/hadoop demo.hadoop.HadoopGrep in out
			查看hadoop/logs/里的运行日志，重新执行前。运行hadoop/bin/hadoop dfs rmr out 删除out目录。

		7.运行hadoop/bin/stop-all.sh 结束


	3.3 集群运行模式
		假设已执行完3.2的配置，假设第2台机器名是hadoopserver2

		1.创建与hadoopserver同样的执行用户，将hadoop解压到相同的目录。

		2.同样的修改haoop-env.sh中的JAVA_HOME 及修改与3.2同样的hadoop-site.xml

		3. 将hadoopserver中的/home/username/.ssh/authorized_keys 复制到hadoopserver2,保证hadoopserver可以无需密码登陆hadoopserver2
			scp /home/username/.ssh/authorized_keys username@hadoopserver2:/home/username/.ssh/authorized_keys

		4.修改hadoop-server的hadoop/conf/slaves文件, 增加集群的节点，将localhost改为
				hadoop-server
				hadoop-server2
	
		5.在hadoop-server执行hadoop/bin/start-all.sh
			将会在hadoop-server启动namenode,datanode,jobtracker,tasktracker
			在hadoop-server2启动datanode 和tasktracker

		6.现在来执行Grep操作
			hadoop/bin/hadoop demo.hadoop.HadoopGrep in out
			重新执行前,运行hadoop/bin/hadoop dfs rmr out 删除out目录
	
		7.运行hadoop/bin/stop-all.sh 结束
```