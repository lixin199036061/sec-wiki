# （CVE-2007-1036）JBoss JMX Console HtmlAdaptor Getshell

> 原文：[https://www.zhihuifly.com/t/topic/2980](https://www.zhihuifly.com/t/topic/2980)

# （CVE-2007-1036）JBoss JMX Console HtmlAdaptor Getshell

## 一、漏洞简介

此漏洞主要是由于JBoss中/jmx-console/HtmlAdaptor路径对外开放，并且没有任何身份验证机制，导致攻击者可以进入到jmx控制台，并在其中执行任何功能。该漏洞利用的是后台中jboss.admin -> DeploymentFileRepository -> store()方法，通过向四个参数传入信息，达到上传shell的目的，其中arg0传入的是部署的war包名字，arg1传入的是上传的文件的文件名，arg2传入的是上传文件的文件格式，arg3传入的是上传文件中的内容。通过控制这四个参数即可上传shell，控制整台服务器。但是通过实验发现，arg1和arg2可以进行文件的拼接，例如arg1=she，arg2=ll.jsp。这个时候服务器还是会进行拼接，将shell.jsp传入到指定路径下

## 二、漏洞影响

全版本

## 三、复现过程

输入url:[http://172.26.1.169:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin%3Aservice%3DDeploymentFileRepository](http://172.26.1.169:8080/jmx-console/HtmlAdaptor?action=inspectMBean&name=jboss.admin%3Aservice%3DDeploymentFileRepository),定位到store方法

![image](img/854046eb9315a57dd6ffda591058646b.png)

传入相应的值，即可getshell

![image](img/27828d0e3be5776d42605631a4e061e9.png)