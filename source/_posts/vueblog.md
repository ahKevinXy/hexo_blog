---
title: vueblog
date: 2020-01-04 15:04:51
tags: ['vue','教程',['前端']]
categoris: 前端
---
![lBZWCQ.jpg](https://s2.ax1x.com/2020/01/05/lBZWCQ.jpg)
# VUE 基础

#### CDN

```html
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

```


#### NPM

在用Vue构建大型应用时推荐使用NPM安装.

```shell

 npm install vue

```

## 介绍



Vue 是一套用于构建用户界面的渐进式框架.Vue被设计为可以自底向上逐层应用.Vue的核心库只关注视图层.




### 起步

#### 安装

```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

```

#### 声明式渲染

vue.js的核心是一个允许采用简洁的模板语法来表明式的将数据渲染进DOM的系统:
```html
    <div id="app">
        {{message}}
    </div>
```

```js
    var app = new Vue({
        el:'#app',
        data:{
            message: 'hello vue'
        }
    })

```

#### 条件与循环

控制切换一个元素是否显示页相当简单

```html
    <div>
        <p v-if='seen' > 现在你看到</p>
    </div>

```

```js
    var app = new Vue({
        el: 'app',
        data: {
            seen : false
        }
    })

```

#### 处理用户输入

为了让用户和应用进行交互,可以v-on指令添加一个事件监听器,通过调用在Vue实例中定义的方法

```html
    <div>
        <p>{{message}}</p>
        <button v-on:onclick='reverseMessage'>翻转信息 </button>
    </div>

```


```js
    var app = new Vue({
        el:'app',
        data: {
            message:'hell vue,js'
        },
        methods:{
            reverseMessage:function(){
                this.message = this.message.split(''),reverse().join('')
            }
        }
    })

```

#### 组件化构建应用

组件系统是vue的另一个重要概率,它是一个抽象,允许使用小型,独立和通常可复用的组件构建大型应用

在Vue,一个组件本质是一个拥有预定义选项的一个Vue实例.在Vue中注册:

```js
    Vue.component('todo-item',{
        template:'<li>这是一个代办选项</li>'
    })

    var app = new Vue();
```
构建另一个组件模板

```html
 <ol>
    <to-item> </to-item>
 </ol>

```

## Vue实例

### 创建一个Vue实例

每个Vue应用都是通过用Vue 函数构建一个新的Vue实例开始

```js

    var vm = new Vue({

    })
```

当创建一个Vue实例时,可以传入一个选项对象.

一个Vue应用由一个通过new Vue 创建的跟Vue实例,以及可选的嵌套的,可复用的组件树组成


### 数据与方法

当一个Vue实例被创建时,它将data对象中的所有属性加入到Vue的响应式系统中.当这些属性的值发生改变时,视图将会产生响应,即更新为新的值


### 实例生命周期钩子

created 钩子可以用在一个实例被创建之后执行的代码


### vue 生命周期

![avatar](https://cn.vuejs.org/images/lifecycle.png)


## 模板语法

Vue.js使用基于html的模板语法,允许开发者声明式的将DOM绑定至底层Vue实例的数据
在底层实现上,Vue将模板编译成虚拟DOM渲染函数.结合响应系统,Vue能够智能的计算出最少需要重新渲染多少组件,并将DOM操作次数减到最少

### 插值

#### 文本
数据绑定最常见的形式就是使用mustache语法的文本插值
```HTML
    <span> Message: {{msg}}</span>
```

mustache标签将会被替代为对应数据对象上msg属性的值.无论何时,绑定的数据对象上msg属性发生了改变,该值处的内容都会更新

通过使用v-once指令,也能执行一次性地插值,当数据改变时,插值处的内容不会更新

```HTML
    <span v-once> 这个将不会发生改变 : {{msg}}</span>
```

#### 原始的HTML
双大括号会将数据解释为普通文本,而非HTML代码,为了输出真正的HTML,需要使用v-html指令:

```HTML
    <p> 使用 mustaches: {{rawHtml}} </p>
    <p > 使用v-html <span v-html="rawHtml" > </span></p>
```

#### 特性

Mustache语法不能作用在HTML特性上,遇到这种情况应该使用v-bind指令

```HTML
    <div v-bind:id="dynamicId"> </div>
```


## 计算属性和侦听器