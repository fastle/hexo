---
title: DVWA：CSRF
date: 2024-09-26 10:20:21
tags:
 - 网络攻防
 - DVWA
---
CSRF 跨站请求伪造，本质来说就是在你访问网站信息的同时，盗用你的cookie，用你的身份进行一些非法操作。

# Low
基本逻辑是修改密码的请求参数都写进了 url 中， 我们需要构造一个修改密码的 url 然后发给其他人， 服务器就会验证他们的身份并且把他们的密码改掉。

# Midium
增加了判断访问请求是否是从 dvwa 网站发起的语句。
```php
if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) {
	// Get input
} else {
    // Didn't come from a trusted source
}
```
复制 REFERER 构造数据包进行攻击即可。

# High
地址带入了 token， 这里有两种解决方法。

1. 构造一个页面让用户点击读取用户登录网站的 token 。但由于同源策略，大概率无法获得 token。
2. 如果用户网站上刚好有存储型漏洞，可以利用存储型漏洞与 csrf 漏洞配合，即通过存储型漏洞获得 token 再利用 csrf 漏洞结合拿到的 token 完成攻击

# Impossible
利用 PDO 技术防御 SQL 注入，至于防护 CSRF，则要求用户输入原始密码（简单粗暴），攻击者在不知道原始密码的情况下，无论如何都无法进行 CSRF 攻击。 
