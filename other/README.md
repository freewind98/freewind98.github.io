# Typecho换Gravatar国内源

## 说明

Gravatar是Globally Recognized Avatar的缩写，意为“全球通用头像”，如果在Gravatar的服务器上放置了你自己的头像，只要提供你与这个头像关联的Email地址，就能够显示出你的Gravatar头像来

Gravatar的概念首先是在国外的独立WordPress博客中兴起的，当你在网站留言时，网站都会根据你所提供的Email地址为你显示出匹配的头像

当然Typecho也是默认有支持Gravatar头像的功能

Typecho被Gravatar头像拖慢了访问速度，是因为Typecho使用的Gravatar的镜像默认是国外镜像，国内访问极其不稳定

## 目前可用Gravatar国内镜像

- [https://dn-qiniu-avatar.qbox.me/avatar/](https://dn-qiniu-avatar.qbox.me/avatar/)
- [https://cravatar.cn/avatar/](https://cravatar.cn/avatar/)
- [https://cdn.sep.cc/avatar/](https://cdn.sep.cc/avatar/)

## 调用
分析Typecho输出Gravatar代码，如下

可以看到如果常量`__TYPECHO_GRAVATAR_PREFIX__`没有定义，则Gravatar使用[https://secure.gravatar.com](https://secure.gravatar.com)或[http://www.gravatar.com](http://www.gravatar.com)，如果定义了就使用`__TYPECHO_GRAVATAR_PREFIX__`常量 的值，我们只需要定义一下`__TYPECHO_GRAVATAR_PREFIX__`常量就可以啦

```php
/**
 * 获取gravatar头像地址 
 * 
 * @param string $mail 
 * @param int $size 
 * @param string $rating 
 * @param string $default 
 * @param bool $isSecure 
 * @return string
 */
public static function gravatarUrl($mail, $size, $rating, $default, $isSecure = false)
{
    if (defined('__TYPECHO_GRAVATAR_PREFIX__')) {
        $url = __TYPECHO_GRAVATAR_PREFIX__;
    } else {
        $url = $isSecure ? 'https://secure.gravatar.com' : 'http://www.gravatar.com';
        $url .= '/avatar/';
    }

    if (!empty($mail)) {
        $url .= md5(strtolower(trim($mail)));
    }

    $url .= '?s=' . $size;
    $url .= '&amp;r=' . $rating;
    $url .= '&amp;d=' . $default;

    return $url;
}
```



在站点根目录下的`config.inc.php`文件加入如下代码

```php
/** 更换头像源 */
define('__TYPECHO_GRAVATAR_PREFIX__', 头像源); 
```

例如

```php
/** 更换头像源 */
define('__TYPECHO_GRAVATAR_PREFIX__', 'https://cdn.v2ex.com/gravatar/'); 
```


# Typecho设置多域名

## 前言

typecho后台只能设置一个域名，但我们通常一个站点会有两域名(一个带www,一个不带的)，比如我在后台设置了个 `https://kevinlu98.cn`，但此时我用`https://www.kevinlu98.cn`带www的域名访问我的博客会出现一些js调用请求的接口为`https://kevinlu98.cn/xxx`，就理所应当的出现了跨域问题

如何解决呢？

## 解决

先去域名提供商添加解析

进行站点根目录找到`var/Widget/Options.php`

全文搜索`__TYPECHO_SITE_URL__`可以定位到如下代码

```PHP
if (defined('__TYPECHO_SITE_URL__')) {
	$this->siteUrl = __TYPECHO_SITE_URL__;
} else if (defined('__TYPECHO_DYNAMIC_SITE_URL__') && __TYPECHO_DYNAMIC_SITE_URL__) {
	$this->siteUrl = $this->rootUrl;
}
```

在代码下面加上

```php
// 多个, 自已设置了几个解析就添加几个
if($_SERVER['HTTP_HOST']=='你的其它域名'){
	$this->siteUrl = 'https://你的其它域名';  
}
```

完整例子

```php
/** 初始化站点信息 */
if (defined('__TYPECHO_SITE_URL__')) {
	$this->siteUrl = __TYPECHO_SITE_URL__;
} else if (defined('__TYPECHO_DYNAMIC_SITE_URL__') && __TYPECHO_DYNAMIC_SITE_URL__) {
	$this->siteUrl = $this->rootUrl;
}
if($_SERVER['HTTP_HOST']=='www.kevinlu98.cn'){
	$this->siteUrl = 'https://www.kevinlu98.cn';  
}
```

## 进后台看看

后台这里也会自动识别了

![](https://imagebed-1252410096.cos.ap-nanjing.myqcloud.com/20220103/17f2bd5cf3e442198f86dcdbd39f2c0d.png) 

![](https://imagebed-1252410096.cos.ap-nanjing.myqcloud.com/20220103/a2187f815341486f96faec3a2306b0ca.png) 

## 测试访问
访问一下看看是否正常

不带www: [点击访问](https://kevinlu98.cn/)

带www: [点击访问](https://www.kevinlu98.cn/)
