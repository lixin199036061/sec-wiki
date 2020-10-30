# （CVE-2018-16356）PbootCMS sql注入漏洞

> 原文：[https://www.zhihuifly.com/t/topic/3067](https://www.zhihuifly.com/t/topic/3067)

# （CVE-2018-16356）PbootCMS sql注入漏洞

## 一、漏洞简介

PbootCMS是一款使用PHP语言开发的开源企业建站内容管理系统（CMS）。 PbootCMS中存在SQL注入漏洞。该漏洞源于基于数据库的应用缺少对外部输入SQL语句的验证。攻击者可利用该漏洞执行非法SQL命令。

## 二、漏洞影响

## 三、复现过程

```
http://www.0-sec.org/api.php/List/index?order=123 
```

$order参数是我们可以控制的

![image](img/4ee7555daa329451a80809ce182dd4fc.png)

转到函数getList，该参数在函数顺序中使用

![image](img/17fa9fc1b5f7f20231b601eb57b5de99.png)

再去看function order函数

![image](img/856acafa45ee01a647d9e94b1bdcb80d.png)

![image](img/44a75dec4919bd19b7c810b8bd9fc97c.png)