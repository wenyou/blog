---
layout: post
title: firefox、IE的本地图片预览
description: "firefox、IE的本地图片预览"
category: javascripts
tags: [js]
---
###兼容firefox、IE的本地图片预览###

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