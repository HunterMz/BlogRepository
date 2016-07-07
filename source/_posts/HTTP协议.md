---
title: HTTP协议
tags:
  - 网络
category:
  - 计算机基础
date: 2016-07-07 14:30:32
---


## 概念
一个基于请求与相应模式的、无状态的、应用层的协议。

## URL
统一资源定位符：唯一标识并定位信息资源。格式如下：
```
http://host[":"port][abs_path]
```
* http表示要通过HTTP协议来定位网络资源
* host表示合法的Internet主机域名或者IP地址
* port指定一个端口号，为空则默认为80
* abs_path制定请求资源的URI

## 工作流程
1. 客户端与服务端建立连接
2. 客户端发送请求
3. 服务端接收到请求，返回相应的相应信息
4. 客户端接收到服务器所返回的信息，通过浏览器等显示给用户

## HTTP 请求报文
HTTP协议的请求主要组成：`请求行`、`请求头`、`请求体`(注：只有Post请求才有请求体)

### Get请求举例分析
```
请求行：
GET / HTTP/1.1 

请求头:
Host: www.baidu.com  //客户端IP和端口
Connection: keep-alive //是否长连接，keep-alive 表示进行长连接
Cache-Control: max-age=0
Accept: text/html,application/xhtml+xml,application  //客户端可以接收的数据类型
xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/38.0.2125.122 Safari/537.36
DNT: 1
Accept-Encoding: gzip,deflate,sdch  //浏览器能够进行解码的数据编码方式
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6,zh-TW;q=0.4,ja;q=0.2
```

### Post请求举例分析
```
请求行: 
POST hysj.jsp HTTP/1.1

请求头:
Host: search.cnipr.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.0; zh-CN; rv:1.9.1.13) Gecko/20100914 Firefox/3.5.13 ( .NET CLR 3.5.30729)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-cn,zh;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: GB2312,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Referer: http://search.cnipr.com/cnipr/zljs/hyjs-biaodan-y.jsp
Content-Length: 405   //请求体的长度

请求体:
username=guest&extension=&issearch=on&searchword=pd%3D%2820100901%29&presearchword=&sortfield=RELEVANCE&sRecordNumber=&searchType=0&searchFrom=0&channelid=14%2C15%2C16&searchChannel=14%2C15%2C16&strdb=14&strdb=15&strdb=16&cizi=2&sortcolumn=RELEVANCE&R1=-&txtA=&txtB=&txtC=&txtD=20100901&txtE=&txtF=&txtG=&txtH=&txtI=&txtJ=&txtK=&txtL=&txtM=&txtN=&txtP=&txtQ=&txtR=&txtSearchWord=&Submit=%BC%EC%A1%A1%CB%F7
```
## HTTP 响应报文
HTTP响应信息包含: `状态行`、`响应头`、`消息体`
1. 状态行: HTTP版本 服务器状态(比如：404找不到...) 描述信息
2. 响应头:

```
content-text：服务器发送信息的类型
date：发送时间
server：服务器类型
```
3.消息体: 服务器返回给客户端的页面内容 
        
## 常见状态码
```
200 OK      //客户端请求成功
400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
401 Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用 
403 Forbidden  //服务器收到请求，但是拒绝提供服务
404 Not Found  //请求资源不存在，eg：输入了错误的URL
500 Internal Server Error //服务器发生不可预期的错误
503 Server Unavailable  //服务器当前不能处理客户端的请求，一段时间后可能恢复正常
```



