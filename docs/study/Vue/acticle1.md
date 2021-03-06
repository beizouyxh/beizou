# Vue 知识点

## v-show 与 v-if 有什么区别 

v-if 是真正的条件渲染，如果为假则什么也不做，直到为真才开始渲染。适合不需要频繁切换条件的场景。

v-show：不管条件是什么都会被渲染，简单的基于css的“display"属性进行切换。适用于需要非常频繁的切换条件的场景。

## vue 的单向数据流

**父组件的prop 的更新会使得子组件所有的prop得到更新，但是反过来不行。**

这样可以防止子组件随意修改父组件的状态，只能通过$emit派发一个自定义事件，父组件接收后由父组件修改。

## vue生命周期的理解

**vue的生命周期指一个vue实例从创建、初始化、编译模板、挂载dom->渲染、更新->渲染、卸载等一系列过程**
1. beforeCreate中拿不到任何数据，它在实例初始化之后，数据观测和事件配置之前被调用。
2. created中可以拿到data中的数据，但是dom还没有挂载。会判断有无el,如果没有el则停止后面的模板挂载。ajax请求和页面初始化。
3. beforeMount和created拿到的数据相同，在挂载之前被调用，相关的render函数首次被调用
4. mounted被创建dom已经更新。常用于获取VNode信息和操作，ajax请求
5. beforeUpdate和updataed使用比较少，一般用计算属性和watch代替
6. destoryed实例销毁后调用，调用后所有事件的监听器被移除，子实例也会被销毁


### vue父组件和子组件生命周期钩子函数执行顺序

- 加载渲染过程

​        父 beforeCreate -> 父 created -> 父 beforeMount -> 子 beforeCreate -> 子       created -> 子 beforeMount -> 子 mounted -> 父 mounted

- 子组件更新过程

  父 beforeUpdate -> 子 beforeUpdate -> 子 updated -> 父 updated

- 父组件更新过程

  父 beforeUpdate -> 父 updated

- 销毁过程

  父 beforeDestroy -> 子 beforeDestroy -> 子 destroyed -> 父 destroyed

### 在哪个生命周期内调用异步请求

created周期

能更快获取到服务器数据，减少页面加载的时间。

SSR不支持beforeMount、mounted钩子函数

### 在什么阶段操作DOM

mounted钩子函数

## 组件中data 为什么是函数？

因为组件是用来复用的，如果组件中的data是一个对象，子组件中的data属性会相互影响。如果是函数，组件实例中的data属性不会相互影响。

new vue的实例不会被复用

## v-model原理

v-model指令在表单input,textarea,select等元素上创建双向绑定

1.在表单元素上做了事件监听

2.如果值发生了变化时，在事件回调函数把最新的值设置到vue实例上

3.因为vue的实例已经实现了数据的响应化，他的响应化的set函数会触发，通知界面中所有模型的依赖的更新

## vue组件通信方式

1.props / $emit   父子组件通信

2.EventBus($emit /  $on)   父子、兄弟、隔代通信

3.provide / inject 隔代组件通信

4.vuex     父子、兄弟、隔代

## vuex

vuex是vue的状态管理系统，核心是store仓库，包含着大部分状态state

好处：

1.集中管理共享数据，易于开发和后期维护
2.实现组件之间的通信，提高开发效率
3.响应式，能够实时保持数据和页面的同步
特点;

​          1.vuex响应式的，组件从store读取状态时，如果状态发生变化，相应组件状态也会更新

          2. 改变store中的转态唯一途径是显示提交mutation

模块：

- state: 提供唯一的公共数据源，所有共享的数据都要统一放到Store 的state中进行存储 ,使用：在要使用的组件中this.$store.state.属性；或在computed中...mapState([属性])
- Getter: 允许组件从store中获取数据(只起到包装作用，不会修改)，this.$store.getters.名称和...mapGetters辅助函数仅仅是将store中的getter映射到局部计算属性
- mutation: 修改store中的状态(不允许组件中修改状态)，this.$store.commit('mutation中的方法')触发和...mapMutations，同步函数,不能执行异步函数;
- action: 用于提交mutation ，this.$store.dispatch和mapActions不变更状态，可以包含任何异步操作
- module:允许将单一的store拆分为多个store,且同时保存单一的状态树中。

## Vue SSR 

SSR就是vue在客户端将标签渲染成整个html片段的过程在服务端完成，服务端形成的html片段再返回个客户端的过程叫服务端渲染。

优点：

- ##### 更好的SEO,因为直接从服务端返回已经渲染好的页面

- 首屏加载时间更快：无需等待下载js文件后再去渲染

缺点：

- 更多的开发条件限制：因为服务器端渲染只支持beforeCreate 和 created两个钩子函数，会导致一些外部扩展库需要特殊处理才能在服务端运行，需要处于node.js server中运行
- 更多的服务器负载

### vue-router 路由模式

hash 、 history  、 abstract



## MVVM

Model --View --ViewModel模式，促进了前后端的分离，提高了开发效率。

核心是ViewModel 向上与视图层进行双向绑定，向下与model层通过接口请求进行数据交互，起到承上启下作用

View：视图层，也就是用户界面。由html和css来构建

```js
<div id="app">
    <p>{{message}}</p>
    <button v-on:click="showMessage()">Click me</button>
</div>
```



Model：数据模型层，后端进行各种业务逻辑处理和数据操控，为前端提供api接口

```js
{
    "url": "/your/server/data/api",
    "res": {
        "success": true,
        "name": "IoveC",
        "domain": "www.cnblogs.com"
    }
}

```



ViewModel:视图数据层

```js
var app = new Vue({
    el: '#app',
    data: {  // 用于描述视图状态   
        message: 'Hello Vue!', 
    },
    methods: {  // 用于描述视图行为  
        showMessage(){
            let vm = this;
            alert(vm.message);
        }
    },
    created(){
        let vm = this;
        // Ajax 获取 Model 层的数据
        ajax({
            url: '/your/server/data/api',
            success(res){
                vm.message = res;
            }
        });
    }
})

```

## Vue实现数据双向绑定

4个步骤

1. 实现一个监听器Observer:对数据对象进行遍历，利用Object.definePrototype()数据劫持，利用getter和setter对数据进行获取和赋值，实现数据的监听
2. 实现一个解析器Compile:扫描解析每个节点的相关指令，根据初始化模板数据以及初始化相应的订阅器。
3. 实现一个订阅者Watcher:Watcher订阅者是Observer 和 Compie之间的通信桥梁，根据订阅Observer属性值变化的消息，触发解析器
4. 实现一个订阅器Dep:订阅器采用发布-订阅模式，收集订阅者watcher，对监听器和订阅者进行统一管理

## Proxy 与 Object.defineProperty 优劣对比

**Proxy 的优势如下:**

- Proxy 可以直接监听对象而非属性；
- Proxy 可以直接监听数组的变化；
- Proxy 有多达 13 种拦截方法,不限于 apply、ownKeys、deleteProperty、has 等等是 Object.defineProperty 不具备的；
- Proxy 返回的是一个新对象,我们可以只操作新的对象达到目的,而 Object.defineProperty 只能遍历对象属性直接修改；
- Proxy 作为新标准将受到浏览器厂商重点持续的性能优化，也就是传说中的新标准的性能红利；

**Object.defineProperty 的优势如下:**

- 兼容性好，支持 IE9，而 Proxy 的存在浏览器兼容性问题,而且无法用 polyfill 磨平

## 虚拟DOM的实现原理

- 用JavaScript对象模拟真实DOM树，对真实DOM进行抽象
- diff算法 --比较两颗虚拟DOM树的差异
- pach算法--将两个虚拟DOM对象的差异应用到真正的DOM树。

优点

- **保证性能下限：**
- **不需要手动操作DOM：**
- **跨平台**：虚拟DOM本质是javascript,   而DOM与平台强相关

缺点：

**无法进行极致优化**

## computed和watch的区别和使用场景

### 区别

**computed计算属性**：依赖其他属性值，并且computed的值有**缓存**，只有它依赖的属性值发生改变，下一次获取computed的值才会重新计算computed的值。

**watch侦听器**：起到【观察】的作用，当**监听的数据变化时都会执行回调**进行后续操作，**无缓存性**。



### 场景

当我们需要进行**数据计算**，并且依赖于其他数据时，使用computed，**有缓存，避免每次获取值时都要重新计算。**



当我们需要在**数据变化时执行异步或开销较大的操作**时，使用watch,允许我们执行异步操作，限制我们执行该操作的频率。



## Vue中的key有什么用？

key是给每一V-node的唯一id,依靠key可是更准确、更快速的进行diff操作

diff算法过程中，对新旧节点进行对比，当无法匹配时用新节点的key与旧节点进行对比，从而找到相应的旧节点。避免了重复渲染