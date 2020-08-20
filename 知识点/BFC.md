## 什么是BFC

BFC全称 块级格式化上下文 

W3C CSS2.1规范中的一个概念

它决定了元素如何对其内容进行定位以及其他元素的关系和相互作用,可以说BFC是一个作用范围

## 触发BFC条件

+ 根元素或其它包含它的元素
+ 浮动元素(元素的 float 不是 none)
+ 绝对定位元素(position 为 absolute 或 fixed)
+ 内联块(diaplay: inline-block)
+ 表格单元格(diaplay: table-cell)
+ 表格标题(display: table-caption)
+ 具有overflow且值不是visible的块元素
+ 弹性盒(flex 或 inline-flex)
+ display: flow-root
+ column-span: all

## BFC的约束规则

+ 内部的盒会在垂直方向一个接一个排列(BFC中常规流)
+ 处于同一个BFC中的元素相互影响,可能会发生外边距重叠
+ 每个元素的margin box的左边,与容器块border box的左边相接触,即使存在浮动也是如此
+ BFC就是页面上的一个隔离的独立容器,容器里面的子元素不会影响外面的元素,反之亦然
+ 计算BFC的高度时,考虑BFC所包含的所有元素,连浮动元素也参与计算
+ 浮动盒区域不叠加在BFC上

## BFC可以解决的问题

+ 垂直外边距重叠问题
+ 去除浮动
+ 自使用两列布局(float + overflow)