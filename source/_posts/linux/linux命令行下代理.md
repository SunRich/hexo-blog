title: linux命令行下设置代理
date: 2017-06-16 13:39:00
tags:
  - linux
---
### yum设置
编辑/etc/yum.conf文件，按如下配置
```
proxy=http://yourproxy:8080/      #匿名代理
proxy=http://username:password@yourproxy:8080/   #需验证代理
```

### 全局代理配置
编辑/etc/profile 或~/.bash_profile ，增加如下内容：
```
http_proxy=proxy.361way.com:8080
https_proxy=proxy.361way.com:8080
ftp_proxy=proxy.361way.com:8080
export http_proxy https_proxy ftp_proxy
```
