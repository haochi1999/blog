## 概念

CORS需要浏览器和服务器同时支持，浏览器中IE10以下不支持

跨域资源共享(CORS) 是一种机制，它使用额外的 HTTP 头来告诉浏览器  让运行在一个 origin (domain) 上的Web应用被准许访问来自不同源服务器上的指定的资源。

## 两种请求

### 简单请求

对于简单请求，浏览器直接发出CORS请求,就是在头部加一个Origin字段，这个字段说明本次请求来自哪个源，服务器根据这个值决定是否同意这次请求，如果不在许可范围内，响应头信息中就不会包含 Access-Control-Allow-Origin 字段

简单请求的回应

+ Access-Control-Allow-Origin：要么是请求时的Origin，要么是 ”*“
+ Access-Control-Allow-Credentials：可选，布尔值，是否允许发送Cookie
+ Access-Control-Expose-Headers：XMLHttpRequest对象getResponseHeader()方法只能拿到六个基本字段，如果想要拿到其他的字段，需要在这个字段指定
+ withCredentials属性：CORS请求默认不发送cookie和HTTP认证信息，如果要把cookie发送到服务器，一方面要服务器指定Access-Control-Allow-Credentials字段，另一方面必须在Ajax中设置withCredentials属性为true

注意：如果要发送cookie，Access-Control-Allow-Origin就不能设置为星号，必须明确指定与请求网页一致的域名，cookie依然遵循同源策略

### 非简单请求

预检请求

对服务器有特殊要求的请求，如PUT、DELETE，或者Content-Type字段类型是 application/json，非简单请求的CORS请求会在正式通信之前增加一次预检请求(OPTIONS)

+ Access-Control-Request-Method：必须，CORS请求用到哪些HTTP方法
+ Access-Control-Request-Headers：逗号分隔的字符串，指定CORS请求会额外发送的头信息字段

预检请求的回应

+ Access-Control-Allow-Origin
+ Access-Control-Allow-Methods：必须，逗号隔开的字符串，表示服务器支持的所有的跨域请求的方法
+ Access-Control-Allow-Headers：如果浏览器请求包含Access-Control-Request-Headers字段，那么该字段就是必须的，表示服务器支持的所有头信息字段，不限于浏览器在预检请求中的字段
+ Access-Control-Allow-Credentials：可选，布尔值，是否允许发送cookie
+ Access-Control-Max-Age：可选，用来表示本次预检请求的有效期，单位为秒，再次期间不用发出另一条预检请求

一旦通过了预检请求，以后浏览器每次CORS请求都和简单请求一样，会有一个Origin字段，服务器都会返回一个 Access-Control-Allow-Origin字段

### 与jsonp相比

CORS和jsonp的使用目的相同

jsonp只支持get请求，CORS支持所有类型的HTTP请求

jsonp的优势在于老浏览器
