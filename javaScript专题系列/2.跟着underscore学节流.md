## 节流

如果你持续触发事件,每隔一段时间,只执行一次操作

使用leading代表首次是否执行

使用trailing代表结束后是否执行一次

## 时间戳实现

```javascript
// 第一版
function throttle (fn,wait) {
    var context,args;
    var previous = 0;

    return function () {
        var now = +new Date();
        context = this;
        args = arguments;
        if(now - previous > wait) {
            fn.apply(context,args);
            previous = now;
        }
    }
}
```

## 定时器实现

```javascript
// 第二版 
function throttle (fn,wait) {
    var context,args,timeout;

    return function () {
        context = this;
        args = arguments;
        if(!timeout){
            timeout = setTimeout(() => {
                timeout = null;
                fn.apply(context,args);
            }, wait);
        }
    }
}
```

## 首次执行加结束执行一次

```javascript
// 第三版
function throttle(fn, wait) {
    var context, args, timeout, result;
    var previous = 0;

    var later = function () {
        previous = +new Date();
        timeout = null;
        fn.apply(context, args);
    }

    var throttled = function () {
        var now = +new Date();
        // 下次触发剩余时间
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        // 如果没有时间剩余或者修改了系统时间
        if(remaining <= 0 || remaining > wait) {
            if(timeout){
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            fn.apply(context,args);
        }else if(!timeout) {
            timeout = setTimeout(later, remaining);
        }
    };

    return throttled;
}
```

## 优化(可控制)

```javascript
// 第四版优化 设置option参数
// leading: false 禁止第一次执行
// trailing: false 禁止停止触发

function throttle(func, wait, options) {
    var timeout, context, args, result;
    var previous = 0;
    if (!options) options = {};

    var later = function () {
        previous = options.leading === false ? 0 : new Date().getTime();
        timeout = null;
        func.apply(context, args);
        if (!timeout) context = args = null;
    };

    var throttled = function () {
        var now = new Date().getTime();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
    };

    return throttled;
}
```

## 取消

```javascript
// ...
throttled.cancel = function () {
    clearTimeout(timeout);
    previous = 0;
    timeout = null;
};
// ...
```

## 注意

leading和trailing不能同时设置false