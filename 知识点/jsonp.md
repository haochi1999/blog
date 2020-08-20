## 简单实现

```js
// localhost:3000
<script type="text/javascript" src="localhost:4000/test.js"></script>
// localhost:4000/test.js
alert("success");
```

script标签的src属性并不会被同源策略所约束，所以可以获取任何服务器上的脚本进行执行

## jsonp的实现模式-callback

```html
<!-- localhost:3000 -->
<script type="text/javascript">
    function callback (data) {
        alert(data.message);
    }
</script>

<script type="text/javascript" src="localhost:4000/test.js"></script>
<!-- localhost:4000/test.js -->
callback({ message: "success" });
```

创建一个回调函数，然后在远程服务器上调用这个函数，将json数据形式作为参数传递，完成回调

## 优化-动态调用

```html
 <!-- localhost:3000 -->
<script type="text/javascript">
    function callback (data) {
        alert(data.message);
    }
    // 添加创建script标签
    function addScriptTag (src) {
        const script = document.createElement('script');
        script.setAttribute('type','text/javascript');
        script.src = src;
        document.body.appendChild(script);
    }

    window.onload = function () {
        addScriptTag('localhost:4000/test.js');
    }
</script>
```

## 总结

优点：不受同源策略限制，兼容性好

缺点：

1. 只支持GET请求，不支持其他类型的请求
2. jsonp调用失败时不会返回各种HTTP状态码
3. 安全性，如果提供jsonp网站存在漏洞返回的js内容被他人控制，那么所有调用这个jsonp服务的网站都会存在漏洞，无法把危险控制在一个域名下，所以在使用jsonp时必须保证jsonp服务时安全可信的

