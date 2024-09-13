---
title: Firefox、IE的本地图片预览
author: Zeeny
comments: false
date: 2009-07-07
updated: 2009-07-07
tags: [Web,JS,Javascript]
categories: [Web,Javascript]
summary: firefox、IE的本地图片预览
---


# 兼容firefox、IE的本地图片预览

```
<input type=”file” name=”head_attachment” id=”head_attachment” onchange=”javascript:resizeImage(‘head_img’,this,this.value);” />

function resizeImage(imgbox,o,imgurl)  
{  
	if(document.all)  
	{  
		document.getElementById(imgbox).src = imgurl;  
		document.getElementById(imgbox).style.display = “block”;  
	}  
	else  
	{  
		document.getElementById(imgbox).src = o.files[0].getAsDataURL();  
		document.getElementById(imgbox).style.display = “block”;  
	}  
}
```