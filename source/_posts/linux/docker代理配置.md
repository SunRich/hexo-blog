title: docker代理配置
date: 2017-06-16 13:39:00
tags:
  - linux
---
## docker代理配置
### 修改Docker的systemd文件，添加http代理配置。
1. 修改/lib/systemd/system/docker.service
```
 EnvironmentFile=-/etc/sysconfig/docker
```
2. 创建或修改/etc/sysconfig/docker
```
HTTP_PROXY=http://proxy.example.com:80/
 HTTPS_PROXY=http://proxy.example.com:80/
 NO_PROXY=localhost,127.0.0.1,internal-docker-registry.somecorporation.com
 export HTTP_PROXY HTTPS_PROXY NO_PROXY
```
3. 刷新配置,使代理生效
```
systemctl daemon-reload
systemctl restart docker
```

```flow
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end

st->op->cond
cond(yes)->e
cond(no)->op
```
