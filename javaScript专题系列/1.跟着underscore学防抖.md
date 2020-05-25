# 防抖

频繁的事件触发:

1. window的 resize、scroll
2. mousedown、mousemove
3. keyup、keydown
4. ...

你尽管触发事件,但是我一定在事件触发n秒后才执行,如果在n秒内重复触发,那就会以新的事件的时间为准,n秒后才执行

## 第一版

```javascript
function debounce (func,wait) {
    var timeout;
    return function () {
        clearTimeout(timeout);
        timeout = setTimeout(func,wait);
    }
}
```

## this & event对象

修复两个小问题:

1. this指向
2. event对象

```javascript
 function debounce (fn,wait) {
    let timer;
    return function () {
        let context = this;
        let args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function(){
            fn.apply(context,args);
        },wait);
    }
}
```

## 立即执行

```javascript
function debounce(fn, wait, immediate) {
    let timer;
    return function () {
        let context = this;
        let args = arguments;
        if (timer) clearTimeout(timer);
        if (immediate) {
            var callNow = !timer;
            timer = setTimeout(() => {
                timer = null;
            }, wait);
            if (callNow) fn.apply(context, args);
        } else {
            timer = setTimeout(function () {
                fn.apply(context, args);
            }, wait);
        }
    }
}
```

## 返回值 & 取消防抖

```javascript
function debounce(fn, wait, immediate) {
    let timer, result;
    let debounced = function () {
        let context = this;
        let args = arguments;
        if (timer) clearTimeout(timer);
        if (immediate) {
            var callNow = !timer;
            timer = setTimeout(() => {
                timer = null;
            }, wait);
            if (callNow) result = fn.apply(context, args);
        } else {
            timer = setTimeout(function () {
                fn.apply(context, args);
            }, wait);
        }
        return result;
    }

    debounced.cancel = function () {
        clearTimeout(timer);
        timer = null;
    }

    return debounced;
}
```

## 动画防止一帧时间渲染多次

```javascript
function debounce(func) {
    var t;
    return function () {
        cancelAnimationFrame(t)
        t = requestAnimationFrame(func);
    }
}
```
