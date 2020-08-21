## cookie

cookie是浏览器实现的一种数据存储功能

作用

+ 会话状态管理
+ 个性化设置
+ 浏览器行为跟踪

持久cookie和会话cookie

Expires：设置Cookie过期时间

Max-Age：Cookie失效前需要经过的秒数，可以为正数负数和0，为正数时浏览器会将其持久化，即写到对应的Cookie文件，为负数时，即表示该Cookie是一个会话Cookie，当为0时会立即删除这个Cookie，如果Expires和max-Age同时存在，max-Age优先级更高

Domain：指定了Cookie可以送达的主机名，假如没有指定，那么默认就是当前文档访问的地址中的主机部分（不包含子域名），如 .taobao.com，这样无论 a.taobao.com和 b.taobao.com都可以使用Cookie，不能跨域设置Cookie，比如阿里名下的页面把Domain设置成百度是无效的

Path：Path 指定了一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部。比如设置 Path=/docs，/docs/Web/ 下的资源会带 Cookie 首部，/test 则不会携带 Cookie 首部

Secure：标记为 Secure 的 Cookie 只应通过被HTTPS协议加密过的请求发送给服务端。

HTTPOnly：防止客户端通过document.cookie等方式访问cookie，有助于避免XSS攻击

**SameSite**

可以让cookie在跨站请求时不被发送，从而阻止跨站请求伪造攻击（CSRF）

属性值

1. Strict：仅允许一方请求携带Cookie，浏览器只发送相同站请求的Cookie，即当前网页URL与请求目标URL完全一致
2. Lax：允许部分第三方请求携带Cookie
3. None：无论是否跨站都会发送Cookie

跨站与跨域的不同：跨站（有效顶级域名+二级域名）


## session

+ cookie：在客户端保存用户状态
+ session：不同的服务框架session的实现机制不一样，它可能被保存在服务端也可能保存在客户端

session实现机制

session信息存储在服务端：服务端负责生成和管理session信息，每个session都会有一个session id，server返回response给客户端时，会以cookie的形式存储session id，后续发起请求时，HTTPC报文Cookie首部都会携带session id，server根据id找到对应的session信息

session信息存储在客户端：服务端生成session信息但不存储，在response中通过Set-Cookie首部让客户端把session信息（一般通过加密）存在cookie中，后续发起请求，客户端会从Cookie首部去除session信息进行解密读取，再通过Set-Cookie写到客户端

从客户端的角度，第一种客户端承担的任务轻，毕竟只存储一个session id，第二种负责管理的信息更重一些

第二点存在的原因：Node无法提供线程服务，为了提高服务端响应速度通常开启多个node进程，进程之间共享数据是个麻烦，把session存在客户端可以很好的规避这个问题
