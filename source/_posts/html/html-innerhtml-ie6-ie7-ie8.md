---
title: ie6、7、8下innerHTML获取html内容属性值上的双引号丢失了
author: Zeeny
comments: false
date: 2013-01-11
updated: 2013-01-11
tags: [Html,IE兼容]
categories: [HTML]
summary: ie6、7、8下innerHTML获取html内容属性值上的双引号丢失了
---


# ie6、7、8下innerHTML获取html内容属性值上的双引号丢失了

* 通过js的innerhtml来获取一个dom节点，内部的html值时，如果属性值没有空格，则ie左右版本下，都会出现标签大些，双引号丢失问题。如果属性值有空格，则标签会变成大写，引号则正常输出。

* 当属性值无空格时：
```
<div id="div2"></div>
```

	IE6 弹出:<DIV id=div2></DIV>
	IE7 弹出:<DIV id=div2></DIV>
	IE8 弹出:<DIV id=div2></DIV>

* 有空格时：
```
<div id="div2 "></div>
```

	IE6 弹出:<DIV id="div2 "></DIV>
	IE7 弹出:<DIV id="div2 "></DIV>
	IE8 弹出:<DIV id="div2 "></DIV>

* 产生这个BUG的原因是什么呢？

	When you get the innerHTML of a DOM node in IE, if there are no spaces in an attribute value, IE will remove the quotes around it.IE’s innerHTML doesn’t remove quotes from non standard attributes.

* 即使是jQuery里面，jquery团队的技术人员也没有解决这个问题。

__提供下解决方案：__

```
function ieInnerHTML(obj) {
	var zz = obj.innerHTML,z = zz.match(/<\/?\w+((\s+\w+(\s*=\s*(?:".*?"|'.*?'|[^'">\s]+))?)+\s*|\s*)\/?>/g);
	if (z){
		for (var i=0;i<z.length;i++){
			var y, zSaved = z[i];
			z[i] = z[i].replace(/(<?\w+)|(<\/?\w+)\s/,function(a){return a.toLowerCase();});
			y = z[i].match(/\=\w+[?\s+|?>]/g);
			if (y){
				for (var j=0;j<y.length;j++){
					z[i] = z[i].replace(y[j],y[j].replace(/\=(\w+)([?\s+|?>])/g,'="$1"$2'));
				}
			}
			zz = zz.replace(zSaved,z[i]);
		}
	}
	return zz;
}
```

* 在相应的地方调用函数ieInnerHTML

```
if(/*@cc_on!@*/0){
	inner_str = ieInnerHTML(obj);
}else{
	inner_str = obj.innerHTML;
}
```

* 原理：对IE版本进行对应的JS处理，而且相应的元素标签中不要出现非标准元素。