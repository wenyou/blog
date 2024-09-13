---
title: url重写一例
author: Zeeny
comments: false
date: 2009-06-21
updated: 2009-06-21
tags: [Web,Apache]
categories: [Web]
summary: Apache url重写一例
---


# url重写一例

```
<Directory "D:/ApacheGroup/www">
  RewriteEngine on
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteRule ^([a-zA-Z]+)/index.html$  index.php?controller=$1 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+).html$  index.php?controller=$1&action=$2 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+)/index.html$  index.php?controller=$1&action=$2&id=1 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+)/([0-9]\*).html$  index.php?controller=$1&action=$2&id=$3 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+)/([0-9]\*)/index.html$  index.php?controller=$1&action=$2&cid=$3&pid=1 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+)/([0-9]\*)/([0-9]\*).html$   index.php?controller=$1&action=$2&cid=$3&pid=$4 [L]
  RewriteRule ^([a-zA-Z]+)/([a-zA-Z]+)/([0-9]\*)/([0-9]\*)/([0-9]\*).html$   index.php?controller=$1&action=$2&cid=$3&mid=$4&pid=$5 [L]
</Directory>
```

