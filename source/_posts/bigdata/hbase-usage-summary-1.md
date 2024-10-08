---
title: HBase使用总结(一)
author: Zeeny
comments: false
date: 2014-12-10
updated: 2014-12-10
tags: [HBase,大数据]
categories: [大数据]
summary: HBase使用总结(一)
---


# HBase使用总结(一)

* 和Hadoop无缝集成：
* - Hadoop分析后的结果可直接写入HBase；
* - 存放在HBase的数据可直接通过Hadoop来进行分析。

## 一、管理HBase

### 1.1 查看

* hadoop fs -ls / 查看hdfs目录，会发现在根目录下多了一个hbase的目录.
* Web Console : http://172.16.37.21:60010/master-status

### 1.2 HBASE Shell

* hbase提供了一个shell的终端给用户交互
* `#$HBASE_HOME/bin/hbase shell`

* 查看Hbase运行状态：hbase(main):002:0> status


```
名称							命令表达式

创建表							create '表名称', '列族名称1','列族名称2','列族名称N'
添加记录						put '表名称', '行名称', '列名称:', '值'
查看记录						get '表名称', '行名称'
查看表中的记录总数	 count  '表名称'
删除记录						delete  '表名' ,'行名称' , '列名称'
删除一张表					先要屏蔽该表，才能对该表进行删除，第一步 disable '表名称' 第二步  drop '表名称'
查看所有记录				scan "表名称"  
查看某个表某个列中所有数据		scan "表名称" , {COLUMNS=>'列族名称:列名称'}
更新记录						就是重写一遍进行覆盖	
```

* HBASE Shell的DDL操作

```
创建表
	>create 'users','user_id','address','info'
		表users,有三个列族user_id,address,info
	
列出全部表
	>list 
	
得到表的描述
	>describe 'users'

创建表
	>create 'users_tmp','user_id','address','info'
	
删除表
	>disable 'users_tmp'
	>drop 'users_tmp'

查检表是可存在与可用
	exists 'users_tmp'
	is_enabled 'users'
	is_disabled 'users'
	
退出shell
	quit
```

* Hbase shell 的DML操作

```
添加记录
	put 'users','Zeeny','info:age','30';
	put 'users','Zeeny','info:birthday','1984-06-03';
	put 'users','Zeeny','info:company','cnzxsoft';
	put 'users','Zeeny','address:contry','china';
	put 'users','Zeeny','address:province','anhui';
	put 'users','Zeeny','address:city','heifei';
	put 'users','Xhuoban','info:birthday','1987-4-17';
	put 'users','Xhuoban','info:favorite','movie';
	put 'users','Xhuoban','info:company','cnzxsoft';
	put 'users','Xhuoban','address:contry','china';
	put 'users','Xhuoban','address:province','anhui';
	put 'users','Xhuoban','address:city','fuyang';
	put 'users','Xhuoban','address:town','zhangji'
	
获取一条记录
	1.取得一个id的所有数据
		get 'users','Zeeny'
	
	2.获取一个id，一个列族的所有数据
		get 'users','Zeeny','info'
	
	3.获取一个id，一个列族中一个列的所有数据
		get 'users','Zeeny','info:age'
	
	
更新记录
	put 'users','Zeeny','info:age','29'
	get 'users','Zeeny','info:age'
	put 'users','Zeeny','info:age','30'
	
获取单元格数据的版本数据
	get 'users', 'Zeeny',{COLUMN=>'info:age',VERSIONS=>1}
	get 'users', 'Zeeny',{COLUMN=>'info:age',VERSIONS=>2}
	get 'users', 'Zeeny',{COLUMN=>'info:age',VERSIONS=>3}
	
获取单元格数据的某个版本数据
	get 'users','Zeeny',{COLUMN=>'info:age',TIMESTAMP=>1418615226809}
	
全表扫描
	scan 'users'
	
	
删除Zeeny值的'info:age'字段
	delete 'users', 'Zeeny', 'info:age'
	get 'users', 'Zeeny'
	
删除整行
	deleteall 'users','Zeeny'
	
统计表的行数
	count 'users'
	
清空表
	truncate 'users'
```

* HBase的Java_API (一)

```
//hbase操作必备，设master-NameNode为hbase master(HMaster)
private static Configuration getConfiguration() {
	Configuration conf = HBaseCinfiguration.create();
	conf.set("hbase.rootdir", "hdfs://master-NameNode:9000/hbase");
	//使用eclipse时必须添加这个，否则无法定位
	conf.set("hbase.zookeeper.quorum", "master-NameNode");
	
	return conf;
}
```

* hbase表重命名

```
方法一：hbase shell
	
	hbase shell> disable 'tableName'
	hbase shell> snapshot 'tableName', 'tableSnapshot'
	hbase shell> clone_snapshot 'tableSnapshot', 'newTableName'
	hbase shell> delete_snapshot 'tableSnapshot'
	hbase shell> drop 'tableName'
	
	
方法二：java api
	
	void rename(HBaseAdmin admin, String oldTableName, String newTableName) { 
		String snapshotName = randomName(); 
		admin.disableTable(oldTableName); 
		admin.snapshot(snapshotName, oldTableName); 
		admin.cloneSnapshot(snapshotName, newTableName); 
		admin.deleteSnapshot(snapshotName); 
		admin.deleteTable(oldTableName);
	}
```

* HBase的Java_API (二)

```
//创建一张表
public static void create(String tableName, String columnFamily) throws IOException {
	HBaseAdmin admin = new HBaseAdmin(getConfiguration());
	if(admin.tableExists(tableName)) {
		System.out.println("table exists!");
	} else {
		HTableDescriptor tableDesc = new HTableDescriptor(tableName);
		tableDesc.addFamily(new HColumnDescriptor(columnFamily));
		admin.createTable(tableDesc);
		System.out.println("create table success!");
	}
}
```

* HBase的Java_API (三)

```
//添加一条记录
public static void put(String tableName, String row, String columnFamily, String column, String data) throws IOException {
		HTable table = new HTable(getConfiguration(), tableName);
		Put p1 = new Put(Bytes.toBytes(row));
		p1.add(Bytes.toBytes(columnFamily), Bytes.toBytes(column), 		Bytes.toBytes(data));
		table.put(p1);
		System.out.println("put'"+row+"',"+columnFamily+":"+column+"','"+data+"'");
}
```

* HBase的Java_API (四)

```
//读取一条记录
public static void get(String tableName, String row) throws IOException {
	HTable table = new HTable(getConfiguration(), tableName);
	Get get = new Get(Bytes.toBytes(row));
	Result result = table.get(get);
	System.out.println("Get: "+result);
}
```

* HBase的Java_API (五)

```
//显示所有数据
public static void scan(String tableName) throws IOException {
	HTable table = new HTable(getConfiguration(), tableName);
	Scan scan = new Scan();
	ResultScanner scanner = table.getScanner(scan);
	for (Result result : scanner) {
		System.out.println("Scan: "+result);
	}
}
```

* HBase的Java_API (六)

```
//删除表
public static void delete(String tableName) throws IOException {
	HBaseAdmin admin = new HBaseAdmin(getConfiguration());
	if(admin.tableExists(tableName)){
		try {
				admin.disableTable(tableName);
				admin.deleteTable(tableName);
		} catch (IOException e) {
				e.printStackTrace();
				System.out.println("Delete "+tableName+" 失败");
		}
	}
	System.out.println("Delete "+tableName+" 成功");
}
```

* HBase的Java_API (七)

```
public static void main(String[] args) throws IOException {
	String tableName="hbase_tb";
	String columnFamily="cf";

	HBaseTestCase.create(tableName, columnFamily);
	HBaseTestCase.put(tableName, "row1", columnFamily, "cl1", "data");
	HBaseTestCase.get(tableName, "row1");
	HBaseTestCase.scan(tableName);
	HBaseTestCase.delete(tableName);
}
```

### 1.3 HBASE结合MapReduce批量导入

```
static class BatchImportMapper extends Mapper<LongWritable, Text, LongWritable, Text> {
	SimpleDateFormat dateformat1 = new SimpleDateFormat("yyyyMMddHHmmss");
	Text v2 = new Text();

	protected void map(LongWritable key, Text value, Context context) throws java.io.IOException ,InterruptedException{
		final String[] splited = value.toString().split("\t");
		try {
			final Date date = new Date(Long.parseLong(splited[0].trim()));
			final String dateFormat = dateformat1.format(date);
			String rowKey = splited[1]+":"+dateFormat;
			v2.set(rowKey+"\t"+value.toString());
			context.write(key, v2);
		} catch (NumberFormatException e) {
			final Counter counter = context.getCounter("BatchImport", "ErrorFormat");
			counter.increment(1L);
			System.out.println("出错了"+splited[0]+" "+e.getMessage());
		}
	};
}
	
	
static class BatchImportReducer extends TableReducer<LongWritable, Text, NullWritable>{
	protected void reduce(LongWritable key, java.lang.Iterable<Text> values, Context context) throws java.io.IOException ,InterruptedException {
		for (Text text : values) {
			final String[] splited = text.toString().split("\t");
		
			final Put put = new Put(Bytes.toBytes(splited[0]));
			put.add(Bytes.toBytes("cf"), Bytes.toBytes("date"), Bytes.toBytes(splited[1]));
			//省略其他字段，调用put.add(....)即可
			context.write(NullWritable.get(), put);
		}
	};
}
	
public static void main(String[] args) throws Exception {
	final Configuration configuration = new Configuration();
	//设置zookeeper
	configuration.set("hbase.zookeeper.quorum", "master-NameNode");
	//设置hbase表名称
	configuration.set(TableOutputFormat.OUTPUT_TABLE, "wlan_log");
	//将该值改大，防止hbase超时退出
	configuration.set("dfs.socket.timeout", "180000");
	
	final Job job = new Job(configuration, "HBaseBatchImport");
	
	job.setMapperClass(BatchImportMapper.class);
	job.setReducerClass(BatchImportReducer.class);
	//设置map的输出，不设置reduce的输出类型
	job.setMapOutputKeyClass(LongWritable.class);
	job.setMapOutputValueClass(Text.class);
	
	job.setInputFormatClass(TextInputFormat.class);
	//不再设置输出路径，而是设置输出格式类型
	job.setOutputFormatClass(TableOutputFormat.class);
	
	FileInputFormat.setInputPaths(job, "hdfs://master-NameNode:9000/input");
	
	job.waitForCompletion(true);
}
```

### 1.4 HBASE的Java_API 练习

```
查询手机13450456688的所有上网记录
public static void scan(String tableName) throws IOException {
	HTable table = new HTable(getConfiguration(), tableName);
	Scan scan = new Scan();
	scan.setStartRow(Bytes.toBytes("13450456688:/"));
	scan.setStopRow(Bytes.toBytes("13450456688::"));
	ResultScanner scanner = table.getScanner(scan);
	int i=0;
	for (Result result : scanner) {
		System.out.println("Scan: "+i+++" "+result);
	}
}


查询134号段的所有上网记录
public static void scanPeriod(String tableName) throws IOException {
	HTable table = new HTable(getConfiguration(), tableName);
	Scan scan = new Scan();
	scan.setStartRow(Bytes.toBytes("134/"));
	scan.setStopRow( Bytes.toBytes("134:"));
	scan.setMaxVersions(1);
	ResultScanner scanner = table.getScanner(scan);
	int i=0;
	for (Result result : scanner) {
		System.out.println("Scan: "+i+++" "+result);
	}
}
```

## 2. 出错处理

* 2.1 The type org.apache.hadoop.hbase.protobuf.generated.HBaseProtos$SnapshotDescription cannot be resolved. It is indirectly referenced from required .class files

```
解决：在项目中添加hbase-protocol-0.98.8-hadoop1.jar 和 protobuf-java-2.5.0.jar两个依赖包。
```
