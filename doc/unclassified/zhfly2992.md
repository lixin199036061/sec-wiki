# （CVE-2018-1000129）Jolokia 反射型xss

> 原文：[https://www.zhihuifly.com/t/topic/2992](https://www.zhihuifly.com/t/topic/2992)

# （CVE-2018-1000129）Jolokia 反射型xss

## 一、漏洞简介

## 二、漏洞影响

Jolokia < 1.50

## 三、复现过程

Jolokia Web应用程序容易受到经典的“ **反射跨站点脚本（XSS）”**攻击。默认情况下，Jolokia返回带有*application / json*内容类型的响应，因此在大多数情况下，将用户提供的输入插入响应中并不是什么大问题。但是通过阅读源代码发现，仅通过向`mimeType`请求添加GET参数就可以修改响应的Content-Type ：

```
http://www.0-sec.org:8161/api/jolokia/read?mimeType=text/html 
```

之后，相对容易地找到至少一个在响应中按原样插入URL参数的情况：

```
http://www.0-sec.org:8161/api/jolokia/read<svg%20onload=alert(docu
ment.cookie)>?mimeType=text/html 
```

使用`text/html`Content Type，可以实现经典的反射XSS攻击。利用此问题，攻击者可以在应用程序输入参数内提供任意客户端JavaScript代码，这些代码最终将在最终用户的Web浏览器中呈现和执行。可以利用它来窃取易受攻击的域中的cookie，并有可能获得对用户身份验证会话的未授权访问，更改易受攻击的网页的内容或损害用户的Web浏览器。

![image](img/e9e749349328a9aef2f52115af1f2022.png)