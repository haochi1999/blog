# 基础篇

## MVVM

+ Model(数据模型)-View(UI组件)-ViewModel(view和model关联起来)
+ 数据会绑定到ViewModel层并自动将数据渲染到页面中,识图变化时会通知ViewModel层更新数据

## vue2.x双向绑定原理

+ 数据变化更新视图,视图变化更新数据,View变化更新data可以通过事件监听的方式实现,所以vue数据双向绑定的工作主要是如何根据data变化更新view
+ 深入理解:
    + 监听器 Observer: 对数据对象进行递归遍历,利用 Object.defineProperty() 对属性加上setter和getter监听数据的变化
    + 解析器 Compile: 解析vue模板指令,将模板中的变量替换成数据,然后初始化渲染页面视图,对每个指令对应的节点绑定更新函数,添加监听数据的订阅者,一旦数据有变化收到通知,调用更新函数进行数据更新
    + 订阅者 Watcher: Observer和Compile之间通信的桥梁,主要任务是订阅Observer中属性值变化的消息,当收到属性变化的消息,触发解析器对应的更新函数,每个组件实例都有相应的watcher实例对象,它会在组件渲染过程中把属性记录为依赖,之后当依赖的setter被调用,就会通知watcher重新计算,从而致使关联的组件更新(典型的观察者模式)
    + 订阅器 Dep: 订阅器采用 发布-订阅 设计模式,用来收集订阅者 Watcher,对监听器 Observer和订阅者 Watcher进行统一管理

## vue3.x响应式数据原理

+ 改用ES6中的Proxy代替Object.defineProperty
+ Proxy只代理对象的第一层,vue3是怎么处理这个问题?
  + 判断当前 Reflect.get 的返回值是否为 Object,如果是则通过 reactive方法做代理,这样实现深度观测
  + 监听数组的时候可能会触发多次get/set,如何防止多次触发?可以判断key是否为当前代理对象target自身属性,也可以判断旧值和新值是否相等,满足以上两个条件之一才有可能执行trigger

### Proxy 与 Object.defineProperty 优劣对比

+ Proxy可以直接监听对象而非属性
+ Proxy可以直接监听数组的变化
+ Proxy有多达13种拦截方法
+ Proxy返回一个新对象,我们可以直接操作新对象达到目的,而Object.defineProperty 只能遍历对象属性直接修改
+ Object.defineProperty兼容性好

# vuex

## vuex是什么?

+ vuex是专为vuejs应用开发的状态管理模式,每个vuex应用的核心是store(仓库),store基本上就是一个容器,它包含着你的应用中的大部分状态(state)
+ vuex和单纯的全局对象有什么区别?
  + vuex的状态存储是响应式的,当vue组件从store中读取状态时,如果store状态发生变化,那么相应的组件也会得到高效更新
  + 改变store中状态的唯一途径就是显式的提交(commit)mutation,这样方便我们跟踪每一个状态的变化

+ 主要包含以下几个模块:
  + State: 定义了应用状态的数据结构,设置默认的初始状态
  + Getter: 允许组件从store中获取数据
  + Mutation: 唯一更改store中状态的方法,必须是同步函数
  + Action: 用于提交mutation,而不是直接变更状态,可以包含任意异步操作
  + Module: 允许将单一的store拆分为多个store且同时保存在单一的状态树中

+ 为什么vuex的mutation中不能做异步操作?
  + vuex所有的状态更新唯一途径都是mutation,异步操作通过action来提交mutation实现,这样使我们更方便跟踪每一个状态的变化
  + 每个mutation执行完成都会对应到一个新的状态变更,这样devtools就可以打个快照存下来,然后可以实现time-travel,如果mutation支持异步操作就没办法指定状态何时更新,无法很好的进行状态跟踪,给调试带来困难

+ vuex的action有返回值吗?返回什么?
  + store.dispatch 可以处理被触发的action的处理函数返回的 Promise,并且store.dispatch仍旧返回 Promise
  + action通常是异步的,要知道action什么时候结束或者组合多个action处理更加复杂的异步流程,可以通过定义action时返回一个promise对象,就可以在派发action的时候通过处理返回的promise处理异步流程

# 常规

## computed 和 watch 的区别和运用的场景?

+ computed: 计算属性,依赖其它属性值,并且值有缓存,只有它依赖的属性值变化,下一次获取的值才会重新计算
+ watch: 没有缓存性,更多的是观察的作用,每当监听的数据变化都会执行回调进行后续操作,深度监听 deep: true
+ 运用场景:
  + 当我们需要进行数值计算并且依赖于其他数据时,应该使用computed,因为可以利用它的缓存性,避免每次获取值都重新计算
  + 当我们需要在数据变化时执行异步或者开销大的操作时,应该使用watch,使用watch允许我们执行异步操作,限制我们执行该操作的频率,并在得到最终结果前设置中间状态,这些都是计算属性无法做到的

## vue2.x组件通信方式

+ 父子组件通信
  + 事件机制( 父->子 props, 子->父 $on、$emit )
  + 获取父子组件实例 $parent、$children
  + Ref获取实例的方式调用组件的属性或方法
  + Provide、inject(不推荐使用,组件库很常用)
+ 兄弟组件通信
  + eventBus: 通过一个空的vue实例作为事件中心,用它来触发事件和监听事件,从而实现任何组件间通信 Vue.prototype.$bus = new Vue
  + vuex
+ 跨级组件通信
  + vuex
  + $attrs、$listeners
  + Provide、inject

## v-if和v-show区别

+ 条件不成立时v-if不会渲染dom元素,v-show操作的是样式(display)切换当前dom显示和隐藏
+ v-if适应于在运行时很少改变条件,不需要频繁切换条件的场景
+ v-show适用于非常频繁切换条件的场景

## 为什么v-for和v-if不建议用在一起

+ 当v-for和v-if处于同一节点时,v-for优先级更高,这意味着v-if将分别重复运行于每个v-for循环中,如果要遍历的数组很大,真正展示数据很少时,将造成很大的性能浪费
+ 这种场景建议使用computed,先对数据进行过滤

## 组件中data为什么是一个函数?

+ 一个组件被复用多次的话,也就创建多个实例,本质上这些实例都是用一个构造函数
+ 如果data是对象,对象属于引用类型,会影响所有的实例,为了保证每个实例间data不冲突,data必须是一个函数(闭包)

## 子组件为什么不能修改父组件传递的prop?怎么理解vue的单向数据流?

+ vue提倡单向数据流,父级props的更新会流向子组件,但是反过来不行
+ 这是为了防止意外的改变父组件的状态,使得应用数据流变得难以理解
+ 如果破坏了单向数据流,当应用复杂时,维护成本非常高

## v-model如何实现双向绑定的?

+ v-model用来在表单控件或组件上创建双向绑定的
+ 本质是 v-bind和v-on 的语法糖
+ 在一个组件上使用v-model,默认为组件绑定名为value的prop和名为input的事件

## nextTick的实现原理是什么?

+ 在下次DOM更新循环结束之后延迟回调,在修改数据之后立即使用 nextTick来获取更新后的DOM
+ nextTick主要使用了宏任务和微任务(微任务优先级高)
+ 根据执行环境分别尝试采用不同的异步方法,如果都不兼容,最后将会采用setTimeout定义一个异步方法,多次调用 nextTick会将方法存入队列中,通过这个异步方法清空当前队列

## vue不能检测数组的哪些变动?vue怎么用vm.$set()解决对象新增属性不响应的问题?

+ vue不能检测以下数组的变化:
  + 当利用索引直接设置一个数组项时
  + 当修改数组的length时
+ vm.$set()实现原理:
  + 如果目标是数组,直接使用数组的splice方法触发
  + 如果目标是对象,先判断属性是否存在,对象是否是响应式,最终如果要对属性进行响应式处理,则通过调用defineReactive 方法添加getter和setter

## vue的事件绑定原理?

+ 原生事件绑定通过addEventListener绑定给真实元素,组件事件绑定通过vue自定义的 $on实现的

## 说下虚拟DOM以及key属性的作用

+ 由于在浏览器中频繁的操作DOM会产生一定的性能问题,这是虚拟DOM产生的原因
+ 虚拟DOM本质是用一个原生js对象去描述一个DOM节点,是对真实DOM的一种抽象
+ 虚拟DOM的实现原理包括三部分:
  + js对象模拟真实DOM树,对真实DOM进行抽象
  + diff算法: 比较两棵DOM树的差异
  + pach算法: 将两个虚拟DOM对象的差异应用到真正的DOM数
+ key是vue在vnode唯一标记,通过key我们diff操作更精准、快速
  + 更精准: 带key避免了就地复用的情况
  + 更快速: 利用key唯一性生成map对象来获取对应节点,比遍历方式更快

## 为什么不建议用index作为key?

+ 因为和没写基本上没区别,不管数组顺序怎么颠倒,index都是0,1,2这样排列,导致vue会复用错误的子节点,做很多额外的工作