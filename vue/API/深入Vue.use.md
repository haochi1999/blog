# Vue.use 用法

### vue 提供了 Vue.use 的全局 api 来注册插件

## 用法

### Vue.use(plugin)

+ 参数如果是一个对象,必须提供install方法
+ 参数如果是一个函数,自身会被当做install方法,调用时会将vue作为参数传入
+ Vue.use(plugin)调用后,install方法会默认接受第一个参数为vue

### 这个方法需要在 new Vue( ) 之前调用

### Vue.use会阻止多次注册相同的插件,调用多次也只会注册一次

# Vue.use 源码分析

```javascript
export function initUse (Vue: GlobalAPI) {
  // plugin参数只接受函数或者对象
  Vue.use = function (plugin: Function | Object) {
      // 保存注册组件的数组,不存在就创建
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    // 判断是否存在,不允许重复注册
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters (额外参数)
    // 将传入的参数转化为数组
    const args = toArray(arguments, 1)
    // 将vue对象拼接到数组的头部,作为第一参数
    args.unshift(this)
    // 如果是对象,判断对象是否提供了install方法
    if (typeof plugin.install === 'function') {
        // 调用install方法,并且将参数数组传入,改变this指针为该组件
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
        // 如果传入的是一个方法,则直接执行
        // 当call或者apply第一个参数为null || undefined,this指向window || global
      plugin.apply(null, args)
    }
    // 将注册过的plugin存进数组
    installedPlugins.push(plugin)
    return this
  }
}
```

```typescript
// 将类数组对象转为真数组
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}
```

### 总结:

```txt
1. 检查插件是否安装,安装了就不再安装
2. 如果没安装,就调用插件的install方法,并且传入vue实例
```

## ElementUI的install

```javascript
const install = function(Vue, opts = {}) {
  locale.use(opts.locale);
  locale.i18n(opts.i18n);
  // components包含了ElementUI所有的组件
  // 将所有组件注册为全局组件
  components.forEach(component => {
    Vue.component(component.name, component);
  });

  Vue.use(InfiniteScroll);
  Vue.use(Loading.directive);

  // 自定义一些参数
  Vue.prototype.$ELEMENT = {
    size: opts.size || '',
    zIndex: opts.zIndex || 2000
  };

  // 在vue原型添加一些方法,这样使得我们通过this.$xxx直接调用
  Vue.prototype.$loading = Loading.service;
  Vue.prototype.$msgbox = MessageBox;
  Vue.prototype.$alert = MessageBox.alert;
  Vue.prototype.$confirm = MessageBox.confirm;
  Vue.prototype.$prompt = MessageBox.prompt;
  Vue.prototype.$notify = Notification;
  Vue.prototype.$message = Message;

};

/* istanbul ignore if */
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue);
}
```

## 自己写个axios插件

```javascript
import Vue from 'vue';
import axios from 'axios';

const myAxios = {}

myAxios.install = function(Vue){
  // 做一些axios配置
  ....

  // vue原型添加方法
  Vue.prototype.$http = axios
}

export default myAxios
```