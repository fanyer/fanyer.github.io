---
layout: post
title:  "移动端适配"
date:   2015-08-12 22:15:52 +0800
categories: jekyll update
---
目前为止出现的一些关于移动端适配的技术方案：

- 通过媒体查询的方式即CSS3的`meida queries`
- 以天猫首页为代表的 `flex` 弹性布局
- 以淘宝首页为代表的 `rem` + `viewport缩放`
- rem 方式

### 1. Meida Queries

meida queries 的方式可以说是我早期采用的布局方式，它主要是通过查询设备的宽度来执行不同的 css 代码，最终达到界面的配置。核心语法是：

```
@media screen and (max-width: 600px) { /*当屏幕尺寸小于600px时，应用下面的CSS样式*/
  /*你的css代码*/
}
```
#### 优点

- `media query`可以做到设备像素比的判断，方法简单，成本低，特别是对移动和PC维护同一套代码的时候。目前像Bootstrap等框架使用这种方式布局
- 图片便于修改，只需修改css文件
- 调整屏幕宽度的时候不用刷新页面即可响应式展示

#### 缺点

- 代码量比较大，维护不方便
- 为了兼顾大屏幕或高清设备，会造成其他设备资源浪费，特别是加载图片资源
- 为了兼顾移动端和PC端各自响应式的展示效果，难免会损失各自特有的交互方式



### 2. Flex 弹性布局

以天猫的实现方式进行说明：

它的`viewport`是固定的：
```
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
```

高度定死，宽度自适应，元素都采用px做单位。

随着屏幕宽度变化，页面也会跟着变化，效果就和PC页面的流体布局差不多，在哪个宽度需要调整的时候使用响应式布局调调就行（比如网易新闻），这样就实现了『适配』

### 3. `rem` + `viewport缩放`

这也是淘宝使用的方案，根据屏幕宽度设定 `rem` 值，需要适配的元素都使用 `rem` 为单位，不需要适配的元素还是使用 `px` 为单位。

#### 实现原理

根据 `rem` 将页面放大 `dpr` 倍, 然后 `viewport` 设置为` 1/dpr`.

如iphone6 plus的 `dpr` 为3, 则页面整体放大3倍, 1px(css单位)在plus下默认为3px(物理像素)
然后 `viewport` 设置为1/3, 这样页面整体缩回原始大小. 从而实现高清。
