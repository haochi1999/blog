# Vue.nextTick

    Vue的全局API,在下次DOM更新循环结束之后执行延迟回调

### vm.$nextTick

    它和全局方法Vue.nextTick一样,不同的是回调的额this自动绑定到调用它的实例上

### 由来

    由于Vue的数据驱动视图更新是异步的,修改数据后视图不会立即更新,而是等同一时间循环中所有数据变化完成后,再统一进行视图更新

```javascript
    // 改变数据
    vm.msg = 'hello'
    // 立即获取是获取不到'hello'
    console.log(vm.$el.textContent)
    // 使用nextTick在DOM更新后执行
    Vue.nextTick(function(){
        // 可以得到'hello'
        console.log(vm.$el.textContent)
    })
```

### 触发时机

    同一时间循环中数据变化后,DOM完成更新,立即执行nextTick(callback)中的回调

### created 与 mounted

在created和mounted阶段,如果需要操作渲染后的视图,需要使用nextTick方法

    官方文档说明:

    mounted不会保证所有的子组件也都一起被挂载,如果希望整个视图都渲染完毕,可以在mounted内部使用vm.$nextTick

### js 运行机制

    js是单线程语言

    为什么js是单线程语言?

        与他的用途有关,作为浏览器脚本语言,js主要用途是用户交互以及操作DOM,如果两个线程同时对一个DOM进行互相冲突的操作,那么浏览器无法正常解析
    
    任务分类

        同步任务

        在主线程排队执行的任务,只有前一个执行完才会执行后一个

        异步任务

        不进入主线程,进入"任务队列",只有"任务队列"通知主线程异步任务可以执行了,该任务才会进入到主线程执行

        异步任务又分为: 

        宏任务: script环境,定时器等

        微任务: Promise,process.nextTick等

        当执行栈为空时,js引擎优先执行微任务队列的任务,微任务处理完,才会处理宏任务队列的任务

        可以明确: 微任务优先级总是高于宏任务

### 任务队列

1. 所有同步任务在主线程执行,形成一个"执行栈"
2. 主线程外还存在一个"任务队列",只要异步任务有了运行结果,就在"任务队列"放置一个事件
3. 一旦"执行栈"同步任务执行完毕,就会开始读取"任务队列",看看里面有哪些事件,哪些异步任务结束等待状态,进入执行栈开始执行
4. 主线程不断重复第三步

**js运行机制: 只要主线程空了,就会去读取"任务队列",这个过程不断重复(Event Loop)**

## 分析 Vue.nextTick 源码

```javascript
// 是否使用异步中的微任务
export let isUsingMicroTask = false
// 回调函数数组
const callbacks = []
// pending状态
let pending = false

// 浅拷贝回调函数数组,清空callbacks,执行回调函数数组
function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
// 异步任务创建
let timerFunc

// 判断环境 兼容 选择异步处理方式

// 微任务
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
    // 微任务
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
  // 宏任务
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
  // 宏任务
} else {
  // Fallback to setTimeout.
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

// 上面这一大段判断,以微任务优先,因为微任务的优先级始终高于宏任务,这样使得DOM更新完,第一时间执行nickTick

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 将回调函数 push进回调函数数组
  callbacks.push(() => {
    if (cb) {
      try {
          // 将callback绑定至context上下文
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
        // 将context作为reslove
      _resolve(ctx)
    }
  })
  // 不是pending状态时 
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  // 如果没传入callback 直接返回一个promise
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```
