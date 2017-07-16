---
title: 生成二维码
date: 2017-07-16 18:45:17
tags:
  - node.js
description: node.js生成二维码，支持带log二维码
---

## 目录

- [前言](#前言)
- [package](#package)
- [关键代码](#关键代码)
- [demo](#demo)

## package

### package.json
```json
{
  "name": "qrcode-node",
  "version": "1.0.0",
  "description": "nodejs Generate the qr code with logo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start":"node app.js"
  },
  "author": "SunRich",
  "license": "ISC",
  "dependencies": {
    "express": "^4.15.2",
    "images": "^3.0.0",
    "qr-image": "^3.2.0"
  }
}

```
### express
web框架
### images
图像处理基础库
### qr-image
生成二维码库

## 关键代码
```javascript
var express = require('express');
var qr = require('qr-image');
var images = require('images');
var app = express();

//生成二维码
var makeQr = function (req, res, logo) {
    res.header("Access-Control-Allow-Origin", "*");
    var url = req.query.url ? req.query.url : 'http://www.kaoyaya.com';
    var size = parseInt(req.query.size);
    var qrBuffer = qr.imageSync(url, {type: 'png', size: 25});
    if (logo != '') {
        qrBuffer = addLogo(qrBuffer, logo)
    }
    if (size > 0) {
        var qrImg = images(qrBuffer);
        qrBuffer = qrImg.size(size, size).encode('png');
    }
    res.type('image/png').send(qrBuffer);
}
//在二维码上添加logo
var addLogo = function (qrBuffer, logo) {
    qrImg = images(qrBuffer);
    var logo = images(logo);
    var logW = logo.width();
    var logH = logo.height();
    var qrW = qrImg.width();
    var qrH = qrImg.height();
    return qrImg.draw(logo, (qrW - logW) / 2, (qrH - logH) / 2).encode('png');
}
app.get('/', function (req, res) {
    makeQr(req, res, '')
});

app.get('/logo', function (req, res) {
    //第三个参数替换你需要的logo的路径
    makeQr(req, res, './kaoyayalogo.png')
});


app.listen(8080, function () {
    console.log('listening on port 8080!')
});

```
## demo
### 获取普通二维码
  - 请求 `get $host/?url=&size=`
  - 参数
    1. url(string):需要转换成二维码的URL地址
    2. size(int):二维码大小
  - 请求示例: `get $host/?url=http://www.kaoyaya.com&size=200`
  - 响应:  
  ![qrimge](node生成二维码/qr.png)
### 获取带logo二维码
  - 请求 `get $host/logo?url=&size=`
  - 参数
    1. url(string):需要转换成二维码的URL地址
    2. size(int):二维码大小
  - 请求示例: `get $host/logo?url=http://www.kaoyaya.com&size=200`
  - 响应:  
  ![qrimge](node生成二维码/qrlogo.png)
