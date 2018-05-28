---
title: 上传绕过
date: 2017-5-27 8:50:30
tags: Web安全
toc: true
---

## 常见绕过姿势
* 客户端绕过(burp 改包)
* Boundary边界(不一致）

* 服务端
	* 文件类型MIME(检测Content-Type)
	* 文件后缀
	* 文件内容
	* 服务端目录路径(path相关的内容)

* 配合文件包含
* 服务器解析漏洞
* CMS，编辑器漏洞
* 操作系统文件名规则
* WAF

## 服务端绕过
### MIME
```
Conent-Type: image/gif (原为Content-Type: text/plain)
```

[mime对照表](https://github.com/tom0li/tom0li.github.io/blob/master/mime.txt)
### 文件后缀
常见黑名单

	asp|asa|cer|cdx|aspx|ashx|ascx|asax
	php|php2|php3|php4|php5|asis|htaccess
	htm|html|shtml|pwml|phtml|phtm|js|jsp
	vbs|asis|sh|reg|cgi|exe|dll|com|bat|pl|cfc|cfm|ini

* 文件后缀大小写
* 后缀pphphp 过滤前一个php后剩下第一个字符p与最后hp组合
* 后缀加空格
* 名单绕过 例如：
	* asa和cer aspx
	* exe和exee
	* jsp jspx jspf
	* php php4

### 文件绕过
在木马内容基础上再加了一些文件信息，有点像下面的结构  
GIF89a<?php phpinfo(); ?>  

图片马

## 解析漏洞
### IIS5.x-6.x
#### 目录解析 
当目录名包含 .asp .asa .cer 时 该目录下文件会asp解析
#### 文件解析
例如 www.xxxx.com/sw.asp;.jpg 按asp解析，不解析;后内容

即文件名有.asp;  .asa; 等 优先asp解析 

### apache

Apache是从后向前解析，如果后面不可识别再向前继续

比如 te.php.sfd.rar ".rar"和".sfd"两种后缀不识别

www.sss.com/esf.php.php123 按php解析

配置问题

（1）如果在 Apache 的 conf 里有这样一行配置 AddHandler php5-script .php 这时只要文件名里包含.php 即使文件名是 test2.php.jpg 也会以 php 来执行。
（2）如果在 Apache 的 conf 里有这样一行配置 AddType application/x-httpd-php .jpg 即使扩展名是 jpg，一样能以 php 方式执行。

[利用最新Apache解析漏洞（CVE-2017-15715）绕过上传黑名单](https://www.leavesongs.com/PENETRATION/apache-cve-2017-15715-vulnerability.html)

### nginx
	Nginx 0.5.*
	Nginx 0.6.*
	Nginx 0.7 <= 0.7.65
	Nginx 0.8 <= 0.8.37
	以上 Nginx 容器的版本下，上传一个在 waf 白名单之内扩展名的文件 shell.jpg，然后以
	shell.jpg.php 进行请求。
	Nginx 0.8.41 – 1.5.6：
	以上 Nginx 容器的版本下，上传一个在 waf 白名单之内扩展名的文件 shell.jpg，然后以
	shell.jpg%20.php 进行请求。


### php CGI 解析漏洞
	IIS 7.0/7.5
	Nginx < 0.8.3

以上的容器版本中默认 php 配置文件 cgi.fix_pathinfo=1 时  
上传一个存在于白名单的扩展名文件 shell.jpg，在请求时以 shell.jpg/shell.php 请求，会将 shell.jpg 以 php 来解析

### 其他
#### 0x00截断
	text.php(0x00).jpg
	text.php%00.jpg
	text.asp(0x00).jpg
#### .htaccess
通过一个.htaccess 文件调用php的解析器解析一个文件名中有"ht"的任意文件，以php解析  
建一个.htaccess文件 内容  
```
<FilesMatch "ht">      
SetHandler application/x-httpd-php </FilesMatch>
```
## 操作系统文件名规则
上传不符合win文件规则的文件名  
												
	test.asp.			生成test.asp		  内容 
	test.asp(空格）			test.asp		
	test.php:1.jpg			test.php
	test.php::$DATA			test.php		<?php phpinfo();?>
	test.php::$DATA.jpg		0.jpg			<?php phpinfo();?>
	test.php::$DATA......	test.php		<?php phpinfo();?>
会去掉

linux下后缀名大小写  
在linux下，如果上传php不被解析，可以试试上传pHp后缀的文件名。

## WAF

删除Conten-Type  
删除Content-Disposion字段的空格  
修改Content-Dispositon字段值大小写

[上传绕过WAF](http://docs.ioin.in/writeup/www.am0s.com/_jchw_376_html/index.html)
[文件解析漏洞汇总](https://zhuanlan.zhihu.com/p/25149704)

#### 参考
		
		http://thief.one/2016/09/22/%E4%B8%8A%E4%BC%A0%E6%9C%A8%E9%A9%AC%E5%A7%BF%E5%8A%BF%E6%B1%87%E6%80%BB-%E6%AC%A2%E8%BF%8E%E8%A1%A5%E5%85%85/
		https://xianzhi.aliyun.com/forum/read/458.html