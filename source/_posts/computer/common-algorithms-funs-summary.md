---
title: 常用算法与函数汇总
date: 2014-10-17
updated: 2014-10-17
categories: [计算机,算法]
tags: [算法,函数]
author: Zeeny
summary: 常用算法与函数汇总
comments: false
---


# 常用算法与函数汇总

__1. linebrk 断行 js__

```
function linebrk(s,n) {
	var ret = "";
	var i = 0;
	while(i + n < s.length) {
			ret += s.substring(i,i+n) + "\n";
			i += n;
		}
		return ret + s.substring(i,s.length);
}
```

__2. 汉诺塔问题__

* 汉诺塔问题：如果将n个盘子（由小到大）从a通过b，搬到c，搬运过程中不能出现小盘子在大盘子下面的情况。
* 分析：这个一个递归问题。只要将n-1个盘子从a通过c（没有中间点肯定不行）搬到b，再将第n个盘子从a搬到c，最后将n-1个盘子从b通过a搬到c。
* 初始状态：A上有n个盘子，B为空，C为空。
* 第一步：把A上的n-1个盘子移动到B；第二步，把A上的第n个盘子移动到C;第三步，把B上的n-1个盘子移动到C。

```
function move($a, $b, $c, $n) {
	if($n == 1) {
		echo "move disk $n from $a to $c <br />";
	} else {
		move($a, $c, $b, $n-1);
		echo "move disk $n from $a to $c <br />";
		move($b, $a, $c, $n-1);
	}
}
move('A','B','C', 4);
```

__3. 冒泡排序法 快速排序法 选择排序法 插入排序法__

* 分别用 冒泡排序法，快速排序法，选择排序法，插入排序法将下面数组中的值按照从小到的顺序进行排序。
* $arr(1, 43, 54, 62, 21, 66, 32, 78, 36, 76, 39);

```
1. 冒泡排序，就是像冒泡一样，每次从数组当中 冒一个最大的数出来。
	
	function sortPao($arr) {
		$len = count($arr);
		//该层循环控制需要冒泡的轮数
		for($i=1; $i<$len; $i++){
			//该层循环用来控制每轮 冒出一个数 需要比较的次数
			for($k=0; $k<$len-$i; $k++) {
				if($arr[$k] > $arr[$k+1]) {
					$tmp = $arr[$k+1];
					$arr[$k+1] = $arr[$k];
					$arr[$k] = $tmp;
				}
			}
		}
		return $arr;
	}
	

2. 选择排序, 每次选择一个相应的元素，然后将其放到指定的位置。

	function sortSelect($arr) {
		//实现思路 双重循环完成，外层控制轮数，当前的最小值。内层 控制的比较次数
		//$i 当前最小值的位置， 需要参与比较的元素
		for($i=0, $len=count($arr); $i<$len-1; $i++) {
			//先假设最小的值的位置
			$p = $i;
			//$j 当前都需要和哪些元素比较，$i 后边的。
			for($j=$i+1; $j<$len; $j++) {
				//$arr[$p] 是 当前已知的最小值
				if($arr[$p] > $arr[$j]){
					//比较，发现更小的,记录下最小值的位置；并且在下次比较时,应该采用已知的最小值进行比较。
					$p = $j;
				}
			}
			//已经确定了当前的最小值的位置，保存到$p中。
			//如果发现 最小值的位置与当前假设的位置$i不同，则位置互换即可:
			if($p != $i) {
				$tmp = $arr[$p];
				$arr[$p] = $arr[$i];
				$arr[$i] = $tmp;
			}
		}
		return $arr;
	}
	

3. 插入排序, 将要排序的元素插入到已经 假定排序好的数组的指定位置。

	function sortInsert($arr){
		//区分 哪部分是已经排序好的,哪部分是没有排序的
		//找到其中一个需要排序的元素, 这个元素 就是从第二个元素开始，到最后一个元素都是这个需要排序的元素。
		//利用循环就可以标志出来。
		//i循环控制 每次需要插入的元素，一旦需要插入的元素控制好了，间接已经将数组分成了2部分，下标小于当前的（左边的），是排序好的序列.
		for($i=1, $len=count($arr); $i<$len; $i++) {
			//获得当前需要比较的元素值。
			$tmp = $arr[$i];
			//内层循环控制 比较 并 插入
			for($j=$i-1; $j>=0; $j--){
				//$arr[$i];
				//需要插入的元素 $arr[$j]
				//需要比较的元素
				if($tmp < $arr[$j]) {
					//发现插入的元素要小，交换位置
					//将后边的元素与前面的元素互换
					$arr[$j+1] = $arr[$j];
					//将前面的数设置为 当前需要交换的数
					$arr[$j] = $tmp;
				} else {
					//如果碰到不需要移动的元素
					//由于是已经排序好是数组，则前面的就不需要再次比较了。
					break;
				}
			}
		}
		//将这个元素 插入到已经排序好的序列内。
		//返回
		return $arr;
	}
	
	
4. 快速排序

	function sortQuick($arr) {
		//先判断是否需要继续进行
		$len = count($arr);
		if($len <= 1) {
			return $arr;
		}
		//选择一个标尺，选择第一个元素
		$base_num = $arr[0];
		//遍历 除了标尺外的所有元素，按照大小关系放入两个数组内
		//初始化两个数组
		$left_arr = array(); //小于标尺的
		$right_arr = array(); //大于标尺的
		for($i=1; $i<$len; $i++) {
			if($base_num > $arr[$i]) {
				//放入左边数组
				$left_arr[] = $arr[$i];
			} else {
				$right_arr[] = $arr[$i];
			}
		}
		//再分别对 左边 和 右边的数组进行相同的排序处理方式
		//递归调用这个函数,并记录结果
		$left_arr = sortQuick($left_arr);
		$right_arr = sortQuick($right_arr);
		//合并左边 标尺  右边
		return array_merge($left_arr, array($base_num), $right_arr);
	}
```