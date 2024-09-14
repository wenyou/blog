---
title: Web网页打印多一张空白页问题
author: Zeeny
comments: false
date: 2024-09-04
updated: 2024-09-04
tags: [Web,HTML]
categories: [Web,HTML]
summary: Web网页打印多一张空白页问题
---

# Web网页打印多一张空白页问题

```
<script type="application/javascript">
    window.print();
</script>
```

* 打印网页出错，多出一张空白页，解决方法。
<p>打印区域添加边框</p>
<p>解决办法是在：打印区域添加一个边框把整个打印内容包裹起来；或者直在body标签属性里直接设置：border: 1px solid transparent; </p>

```
<body data-type="orderprofile" id="print" style="border:1px solid transparent;">
  打印内容......
</body>

或者：
<div style="border:1px solid transparent;">
  打印内容......
</div>
```
