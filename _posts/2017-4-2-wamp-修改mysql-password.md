---
title: wampserver-mysql-修改密码问题
date: 2017-4-2 8:50:30
tags: issue
toc: true
---

### 命令行操作
```mysql
use mysql;
update user set authentication_string=PASSWORD('hooray') where user='root';
flush privileges;
quit;
```

#### 修改登陆phpmyadmin
* 进入wamp安装目录/apps/phpmyadmin 打开config.inc.php
```php
$cfg['Servers'][$i]['password'] = 'newpassword';
$cfg['Servers'][$i]['AllowNoPassword'] = false;					
```

* 进入目录wamp/apps/phpmyadmin/libraries/ 打开config.default.php 文件
```php
$cfg['Servers'][$i]['password'] = 'admin';
```