## 概念

CORS是一个W3C标准，全称跨域资源共享，他允许浏览器向跨源服务器发出异步请求。

## 两种请求

浏览器将CORS分为两类：简单请求和非简单请求。

### 简单请求

对于简单请求，浏览器直接在头信息中增加一个Origin字段来说明本次请求来自哪个源，服务器根据这个值，决定是否同意这次请求。

1. Access-Control-Allow-Origin

该字段是必须的，它的值要么是请求时Origin字段的值，要么是一个 * ，表示接受任意域名的请求。

2. Access-Control-Allow-Credentials

该字段可选，是一个布尔值，表示是否允许发送Cookie。默认情况下Cookie不包含在CORS请求中。

3. Access-Control-Expose-Headers

该字段可选，CORS请求时，XMLHttpRequest对象的 getResponseHeader()方法只能拿到6个基本字段，如果想拿到其他字段就要在里面指定。

### 非简单请求

1. 预检请求

对服务器有特殊要求的请求，比如PUT或DELETE或者 Content-Type字段类型是 application/json。

非简单的CORS请求，会在正式通信前增加一次HTTP查询请求
