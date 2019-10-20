---
title: vue_route
tags:
  - vue
originContent: ''
categories:
  - 文章
toc: false
date: 2019-09-29 23:37:49
---

# vue自动化路由

## 目的

解放双手，从此不用配置路由。当你看到项目中大批量的路由思考是拆分维护业务路由还是统一入口维护时，无需多虑，`router-auto`是你的最优选择，它帮你解决路由的维护成本，你只需要创建相应的文件夹，路由就能动态生成，路由拦截你可以在main.js中去拦截他，总之比你想象的开发还要简单。

- router-auto github地址有帮助的可以star一下
- router-auto npm地址欢迎提issue

## 实现效果

![1](https://user-gold-cdn.xitu.io/2019/9/29/16d7c6065291abeb?imageslim)


## 简要用法

具体用法请实时查看github或者npm，下面做一些简要用法介绍

## 引用
```python
const RouterAuto = require('router-auto')

module.exports = {
    entry: '...',
    output: {},
    module: {},
    plugin:[
        new RouterAuto()
    ]
}
```

## 项目结构

- package.json 等等文件与目录
- src 项目目录
  - page 页面目录
      - helloworld
        - Index.vue 页面入口
        - hello.vue 业务组件
        - router.js 额外配置
      - demo
        - test
          - Index.vue 页面入口
          - test.vue 业务组件
        - Index.vue 页面入口
      - home
        - Index.vue 页面入口




上面的目录结构生成的路由结构为
```python
import Vue from 'vue'
import Router from 'vue-router'
import helloworld from '@/page/helloworld/Index.vue'
import demo from '@/page/demo/Index.vue'
import demo_test from '@/page/demo/test/Index.vue'
import home from '@/page/home/Index.vue'
  
Vue.use(Router)
  
export default new Router({
    mode:'history',
    base:'/auto/',
    routes:[{
      path: '/helloworld/index',
      name: 'helloworld',
      component: helloworld
    },{
      path: '/demo/index',
      name: 'demo',
      component: demo
    },{
      path: '/demo/test/index',
      name: 'demo_test',
      component: demo_test
    },{
      path: '/home/index',
      name: 'home',
      component: home
    }]
})
```

> 作者：ngaiwe
链接：https://juejin.im/post/5d90798151882576e44088e8
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。