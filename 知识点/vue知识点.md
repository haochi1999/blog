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

# 生命周期

## 说下你对vue生命周期的理解

+ beforeCreate、created、beforeMount、mounted、beforeUpdate、updated、beforeDestroy、destroyed
+ keep-alive有自己独立的钩子函数 activated和deactivated
+ 详细回答
  + beforeCreate: data、methods、computed以及watch上的数据和方法都不能访问
  + created: 实例创建完成后发生,当前阶段以及完成了数据观测,可以做一些初始数据的获取,此阶段无法与DOM进行交互,可以通过nextTick来访问DOM
  + beforeMount: 发生在挂载前,当前阶段虚拟DOM已经创建完成,即将开始渲染,可以对数据进行更改但是不会触发updated
  + mounted: 挂载完成后发生,此阶段真实的DOM挂载完毕,数据完成双向绑定,可以访问DOM节点,使用$ref属性对DOM进行操作
  + beforeUpdate: 发生在更新之前,也就是响应式数据发生更新,虚拟DOM重新渲染之前被触发,可以在当前阶段进行更改数据,不会造成重渲染
  + updated: 发生在更新完成之后,此阶段组件DOM已完成更新,避免在此阶段修改数据,因为可能造成无限循环更新
  + beforeDestroy: 发生在实例销毁之前,在此阶段实例完全可以被使用,可以在这进行善后工作,比如清除定时器
  + destroyed: 发生在实例销毁之后,这时候只剩下DOM空壳,组件已经被拆解,数据绑定被卸除,监听被移出,子实例也统统被销毁
  + activited(keep-alive专属): 组件被激活时调用
  + deactivated(keep-alive专属): 组件被销毁时调用

## vue中组件生命周期调用顺序是什么样的?

+ 组件调用顺序都是先父后子,渲染完成的顺序是先子后父
+ 组件销毁操作时先父后子,销毁完成的顺序是先子后父

## 在什么阶段才能操作DOM?

+ mounted不会包装所有的子组件也都一起被挂载,如果希望整个视图都渲染完毕,可以在mounted颞部使用nextTick

## 你的接口请求一般放在哪个生命周期?

+ 可以在created、beforeMount、mounted 中进行调用,因为这三个钩子函数中data已经创建,可以将服务端返回的数据进行赋值
+ 推荐在created中调用异步请求:
  + 能更快获取服务端数据,减少页面加载时间
  + ssr不支持beforeMount 、mounted 钩子函数,所以放created中有助于一致性

# 路由

## vue路由hash模式和history模式实现原理是什么,区别是什么?

+ hash模式
  + "#"后面hash值的变化,不会导致浏览器向服务器发出请求,浏览器不发出请求就不会刷新页面
  + 通过监听hashchange事件可以知道hash发生了哪些变化,根据hash变化来实现页面部分的内容操作
+ history模式
  + 主要是HTML5标准发布的两个API, pushState和replaceState,这两个API可以改变url,但是不会发出请求,这样可以监听url变化来实现更新页面部分内容的操作
+ 区别
  + url展示上,hash模式有"#",history模式没有
  + 刷新页面时,hash模式可以正常加载到hash值对应的页面,而history没有处理的话会返回404,一般需要后端对所有页面都配置重定向到首页路由
  + hash兼容性更加的好,可以支持低版本浏览器和IE

## 路由懒加载是什么?如何实现?

+ 定义: 把不同路由对应的组件分割成不同的代码块,当路由被访问的时候才加载对应组件
+ 实现: 结合vue的异步组件和webpack代码分割功能
  + 1. 将异步组件定义为返回一个Promise的工厂函数(该函数返回Promise应该reslove组件本身)
  + 2. 在webpack中,我们使用动态import语法来定义代码分块点
  + const Foo = () => import('./Foo.vue')
  + 使用命名chunk,和webpack的魔法注释就可以把某个路由下的所有组件打包在同一个chunk中
  + chunkconst Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')

## vue-router导航守卫有哪些

+ 全局前置/钩子: beforeEach、beforeResolve、afterEach
+ 路由独享的守卫: beforeEnter
+ 组件内的守卫: beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave

# 进阶

## 说说vue和react的异同

+ 同
  + 使用虚拟DOM
  + 提供了响应式和组件化的视图组件
  + 将注意力集中保持在核心库,将其他功能如路由和全局状态管理交给相关的库
+ 异
  + 在react应用中,当某个组件状态发生变化时,它就会以该组件为根,重新渲染整个组件子树,在vue应用中,组件的依赖在渲染过程中自动追踪,所以系统能精确知道那个组件确实需要被重渲染
  + 在react中一切都是js,html可以用jsx表示,css也可以纳入到js中处理
  + vue提供了CLI脚手架,react提供了create-react-app

## mixin是什么?

+ mixin使我们能够为vue组件编写可插拔和可重用的功能
+ 如果你希望在两个组件之间重用一组组件选项,例如生命周期hook、方法等,则可以将其编写为mixin,并在组件中简单引用它
+ 将mixin的内容合并到组件中,如果你要在迷信中定义生命周期hook,那么它在执行时将优化于组件自己的hook

## 在vue实例中编写hook或者其他option时为什么不能使用箭头函数?

+ 箭头函数自身没有this
+ 箭头函数this关键字不会绑定到vue实例,所以强烈建议改用标准函数声明

## vue模板编译原理简单说一下

简单的说vue编译过程就是将 temlate转化为render函数的过程,会经历一下阶段(生成AST语法树/优化/codegen):

+ 首先解析模板,生成AST语法树(一种用js对象的形式来描述整个模板),使用大量正则表达式来对模板进行解析,遇到标签、文本的时候都会执行对应的钩子进行相应处理
+ vue数据是响应式的,但是模板并不是所有数据都是响应式的,有一些数据首次渲染后就不会再变化,对应的DOM也不会变化,优化过程就是深度遍历AST树,按条件对树节点进行标记,被标记的就可以跳过对它们的比对,对模板起到很大的优化作用
+ 编译最后一步将优化后的AST树转化为可执行的代码

## diff算法说一下

+ 同级比较,再比较子节点
+ 先判断一方有子节点一方没有子节点的情况,如果新的children没有子节点,将旧的子节点移除
+ 比较都有子节点的情况(核心diff)
+ 递归比较子节点

## 说说你对keep-alive组件的了解

+ keep-alive是vue内置的一个组件,可以使被包含的组件保留状态,避免重新渲染,具有以下特性:
  + 一般结合路由和动态组件一起使用,用于缓存组件
  + 提供 include和exclude属性,两者都支持字符串或正则表达式,include表示只有名称匹配的组件会被缓存,exclude表示任何名称匹配的组件都不会被缓存,其中exclude优先级最高
  + 对应两个钩子函数 activated 和 deactivated ,当组件被激活时,触发钩子函数 activated,当组件被移除时,触发钩子函数 deactivated

## 说说你对SSR的了解

+ SSR也就是服务端渲染,将vue在客户端把标签渲染成html的工作放在服务端完成,然后把html返回给客户端
+ SSR优势
  + 更好的SEO
  + 首屏加载速度更快
+ SSR缺点
  + 开发条件受限制,服务器端渲染只支持beforeCreate和created两个钩子
  + 当我们需要一些外部扩展库时需要特殊处理,服务端渲染应用程序也需要处于Nodejs的运行环境
  + 更多的服务端负载

## 你都做过哪些vue的性能优化?

+ 编译阶段
  + 尽量减少data中的数据,data中的数据会增加getter和setter,会收集对应的watcher
  + v-if和v-for不能连用
  + v-for给每项元素绑定事件时使用事件代理
  + SPA 页面采用keep-alive缓存组件
  + 在更多的情况下,使用v-if替代v-show区别
  + key值保证唯一(不使用索引)
  + 使用路由懒加载、异步组件
  + 防抖节流
  + 第三方模块按需引入
  + 图片懒加载
+ SEO优化
  + 预渲染
  + SSR
+ 打包优化
  + 压缩代码
  + tree shaking
  + cdn加载第三方模块
  + 多线程打包happypack
  + sourceMap优化
  + splitChunks抽离公共文件
+ 用户体验
  + 骨架屏
  + PWA
  + 客户端缓存、服务端缓存、服务器开启gzip压缩

## vue2.x如何监测数组的变化

+ 使用了函数劫持,重写了数组的方法,vue将data中的数组进行了原型链重写,指向了自定义的数组原型方法,当调用数组api的时候可以通知依赖更新
+ 如果数组包含着引用类型,会对数组中的引用类型再次递归遍历进行监控,这样实现了监测数组变化

## 说说你对SPA单页面的理解,它的优缺点是什么?

+ 仅在web页面初始化时加载对应的html、js、css,一旦页面加载完成,SPA不会因为用户操作而进行页面的重新加载和跳转,取而代之的是利用路由机制实现html内容的变换,UI与用户交互避免页面的重新加载
+ 优点
  + 用户体验好、快,内容的改变不需要重新加载整个页面,避免了不必要的跳转和重复渲染
  + SPA相对对服务器压力小
  + 前后端职责分离,架构清晰,前端进行交互逻辑,后端负责数据处理
+ 缺点
  + 初次加载耗时多: 为实现单页面web应用,需要在加载页面时将js、css统一加载,部分页面按需加载
  + 前进后退路由管理: 由于单页应用一个页面显示所有内容,所以不能使用浏览器前进后退功能,所有页面的切换需要自己建立堆栈管理
  + SEO难度较大: 由于所有内容都在一个页面动态替换显示,所以在SEO上有天然的劣势

## 即将到来的vue3.0特性你有了解吗?

+ 监听机制的改变
  + 3.0将带来机遇代理proxy的observer实现,提供全语言覆盖的反应性跟踪
  + 消除了vue2基于 Object.defineProperty 的实现所存在的很多限制:
+ 只能检测属性,不能检测对象
    + 检测属性的添加和删除
    + 检测数组的索引和长度的变更
    + 支持Map、Set、WeakMap、WeakSet
+ 模板
  + 模板方面没有太大的变更,只是改了作用域插槽, 2.x的机制导致作用域变了,父组件会从新渲染,而3.0把作用域插槽改成了函数方式,这样只会影响子组件的重新渲染,提升了渲染的性能
  + render函数方面,vue3.0会进行一系列更改来方便习惯直接使用api来生成vdom
+ 对象式的组件声明方式
  + vue2.x中的组件通过声明的方式传入一系列option,和typescript的结合需要通过一些修饰器的方式来做,虽然能实现功能但是比较麻烦
  + 3.0修改了组件的声明方式,改成了类式的写法,这样使得和typescript的结合变得很容易
+ 其他方面的更改
  + 支持自定义渲染器,从而使得weex可以通过自定义渲染器的方式来扩展,而不时直接fork源码来改
  + 支持 Fragment（多个根节点）和 Protal（在 dom 其他部分渲染组建内容）组件,针对一些特殊的场景做了处理
  + 基于 tree shaking优化,提供了更多的内置功能