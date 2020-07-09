---
title: vue开发的常用技巧
date: 2020-07-06
tags:
- vue
categories: 
- vue
sidebarDepth: 2
---

## require.context()
1. 场景：如页面需要导入多个组件,原始写法:
```javascript
import titleCom from '@/components/home/titleCom'
import bannerCom from '@/components/home/bannerCom'
import cellCom from '@/components/home/cellCom'
components:{titleCom,bannerCom,cellCom}
```
2. 这样就写了大量重复的代码，利用requirecontext可以写成
```javascript
const path = require('path')
const files = require.context('@/components/home', false, /\.vue$/)
const modules = {}
files.keys().forEach(key => {
  const name = path.basename(key, '.vue')
  modules[name] = files(key).default || files(key)
})
components:modules
```
这样不管页面引入多少组件，都可以使用这个方法

3. **分析require.context**  
```
require.context函数接受三个参数
1. directory {String} - 读取文件的路劲
2. useSubdirectories {Boolean} - 是否遍历文件的子目录
3. regExp {RegExp} - 匹配文件的正则
```

借用webpack官网的例子
:::tip
require.context('./test', false, /.test.js$/)
:::

值得注意的是require.context函数执行后返回的是**一个函数，并且这个函数有3个属性**
1. resolve {Function} - 接受一个参数request，request为test文件夹下面匹配文件的相对路径，返回这个匹配文件相对于整个工程的相对路径
2. keys {Function} - 返回匹配成功模块的名字组成的数组
3. id {String} - 执行环境的id，返回的是一个字符串

## 组件通讯
### props
这就是父传子的属性  
```javascript
props: {
  inpVal: {
    type: Number,   //传入值限定类型
    required: true, //是否必传
    default: 200，  //默认值，对象或数组默认值必须从一个工厂函数获取如default:()=>[]
    validator:(value) {
      // 这个值必须匹配下列字符串中的一个
      return ['success', 'warning', 'danger'].indexOf(value) !== -1
    }
  }
}
```

### $emit
子传父的方法
```vue
// 父组件
<home @title="title">
// 子组件
this.$emit('title', [{title:'这是title'}])
```

### vuex
1. vuex是一个状态管理器
2. 是一个独立的插件，适合数据共享多的项目里面，但是如果只是简单的通讯，使用起来比较麻烦
3. API
```
state: 定义存储数据的仓库，可通过this.$store.state 或 mapState 访问
getter: 获取 store 值，可认为是 store 的计算属性，可通过this.$store.getter 或
  mapGetters访问
mutation：同步改变 store 值，为什么会设计同步，因为 mutation 是直接改变 store 值，vue 对操作进行了记录，如果是异步无法追踪改变，可通过 mapMutations 调用
action：异步调用函数执行 mutation，进而改变 store 值，可通过 this.$dispatch 或 mapActions 访问
modules: 模块，如果状态过多，可以拆分成模块，最后在入口通过...解构引入
```

### listeners
2.4.0新增
1. 场景：如果父传子有很多值，那么在子组件需要定义多个解决attrs获取子传父中未在props定义的值
``` javascript
// 父组件
<home title="标题" width="80" height="80" imgUrl="imgUrl">
// 子组件
mounted() {
  console.log(this.$attrs) //{title: "这是标题", width: "80", height: "80", imgUrl: "imgUrl"}
},
```
相对应的如果子组件定义了props，打印的值就是剔除定义的属性
```javascript
props: {
  width: {
    type: String,
    default: ''
  }
},
mounted() {
  console.log(this.$attrs) //{title: "这是标题", height: "80", imgUrl: "imgUrl"}
},
```

2. 场景：子组件需要调用父组件的方法解决父组件的方法，可以通过listeners传入内部组件--在创建更高层次的组件时非常有用
```javascript
// 父组件
<home @change="change"/>

// 子组件
mounted() {
  console.log(this.$listeners) //即可拿到 change 事件
}
```
如果是孙组件要访问父组件的属性和调用方法，直接一级一级传下去就可以

3. inheritAttrs
```javascript
// 父组件
<home title="这是标题" width="80" height="80" imgUrl="imgUrl"/>

// 子组件
mounted() {
  console.log(this.$attrs) //{title: "这是标题", width: "80", height: "80", imgUrl: "imgUrl"}
},

inheritAttrs默认值为true，true的意思是将父组件中除了props外的属性添加到子组件的根节点上(说明，即使设置为true，子组件仍然可以通过$attr获取到props意外的属性)
将inheritAttrs:false后,属性就不会显示在根节点上了
```

### children
父实例children:子实例
```javascript
//父组件
mounted(){
  console.log(this.$children) 
  //可以拿到 一级子组件的属性和方法
  //所以就可以直接改变 data,或者调用 methods 方法
}

//子组件
mounted(){
  console.log(this.$parent) //可以拿到 parent 的属性和方法
}
```
并不保证顺序，也不是响应式
只能拿到一级父组件和子组件

### $refs
```javascript
// 父组件
<home ref="home">

mounted() {
  console.log(this.$refs.home) //即可拿到子组件的实例,就可以直接操作 data 和 methods
}
```

### $root
```javascript
// 父组件
mounted(){
  console.log(this.$root) //获取根实例,最后所有组件都是挂载到根实例上
  console.log(this.$root.$children[0]) //获取根实例的一级子组件
  console.log(this.$root.$children[0].$children[0]) //获取根实例的二级子组件
}
```

### .sync
```javascript
// 父组件
<home :title.sync="title" />
//编译时会被扩展为
<home :title="title"  @update:title="val => title = val"/>

// 子组件
// 所以子组件可以通过$emit 触发 update 方法改变
mounted(){
  this.$emit("update:title", '这是新的title')
}
```

### 路由传参
1. 方案一
```javascript
// 路由定义
{
  path: '/describe/:id',
  name: 'Describe',
  component: Describe
}
// 页面传参
this.$router.push({
  path: `/describe/${id}`,
})
// 页面获取
this.$route.params.id
```

2. 方案二
```javascript
// 路由定义
{
  path: '/describe',
  name: 'Describe',
  component: Describe
}
// 页面传参
this.$router.push({
  name: 'Describe',
  params: {
    id: id
  }
})
// 页面获取
this.$route.params.id
```

3. 方案三
```javascript
// 路由定义
{
  path: '/describe',
  name: 'Describe',
  component: Describe 
}
// 页面传参
this.$router.push({
  path: '/describe',
  query: {
    id: id
  }
})
// 页面获取
this.$route.query.id
```

4. 三种方案对比  
- 方案二参数不会拼接在路由后面，页面刷新参数会丢失
- 方案一和方案三参数拼接在后面，不好看且暴露了信息

## 动态组件

场景：做一个tab切换时就会涉及到组件动态加载
```template
<component v-bind:is="currentTabComponent"></component>
```
但是这样每次组件都会重新加载，会消耗大量性能，所以<code>keep-alive</code>就起到了作用
```template
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```
这样切换效果没有动画效果，这个也不用着急，可以可用内置的<code>transition</code>
```javascript
<transition>
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
</transition>
```

## 递归组件
场景：如果开发一个tree组件，里面层级是根据后台数据决定的，这个时候就需要用到动态组件
```vue
// 递归组件：组件在它的模板内可以递归的调用自己，只要给组件设置name组件就可以了。
// 设置name在组件模板内就可以递归使用了，不过需要注意的是：
// 必须给一个条件来限制数量，否则会抛出错误：max stack size exceeded
// 组件递归用来开发一些具体有未知层级关系的独立组件。比如：
// 联级选择器和树形控件
<template>
  <div v-for="(item, index) in treeArr">
    子组件，当前层级值：{{index}} <br/>
    // 递归调用自身，后台判断是否不存在该值
    <tree :item="item.arr" v-if="item.flag"></tree>
  </div>
</template>
<script>
export default {
  // 必须定义name，组件内部才能递归调用
  name: 'tree',
  data() {
    return {}
  },
  // 接收外部传入的值
  props: {
    item: {
      type: Array
      default: ()=>[]
    }
  }
}
</script>
```
递归组件必须设置name和结束的阀值

## components和Vue.component

components: 局部注册组件
```vue
export default {
  components: {home}
}
```
Vue.component:全局注册组件
```javascript
Vue.component('home', home)
```

## Vue.extend
场景：vue组件中有些需要将一些元素挂载到元素上，这个时候extend就起到作用了
是构造一个组件的语法器
写法：
```vue
// 创建构造器
var Profile = Vue.extend({
  templage:'<p>{{extendData}}</br>实例传入的数据为:{{propsExtend}}</p>',//template对应的标签最外层必须只有一个标签
  data: function() {
    return {
      extendData: '这是extend扩展的数据',
    }
  },
  props: ['propsExtend']
})

// 创建的构造器可以挂载在元素上，也可以通过components或Vue.component()注册使用
// 挂载到一个元素上，可以通过propsData传参
new Profile({propsData:{propsExtend:'我是实例传入的数据'}}).$mount('#app-extend')

// 通过 components 或 Vue.component() 注册
Vue.component('Profile', Profile)
```

## mixins

场景：有些组件有些重复的js逻辑，如校验手机验证码，解析时间等，mixins就可以实现这种混入  
**混入（mixins）**:是一种分发Vue组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。  
mixins值是一个**数组**

```javascript
// mixin.js
// 定义一个混入对象
export const myMixin = {
  data() {
    return {
      num:1,
    }
  },
  created() {
    this.hello();
  },
  methods: {
    hello() {
      console.log('hello from mixin')
    }
  }
}
```

```vue
// 把混入对象混入到当前的组件中
<template>
  <div class="template1">
    组件一
  </div>
</template>

<script>
import {myMixin} from '@/assets/mixin.js'

export default {
  mixins: [myMixin]
}
</script>
```
**mixins的特点**  
:::tip
1. 方法和参数在各个组件中不共享（混入对象的值在组件一和组件二等等互相独立，不受影响）
2. 值为对象的选项，如methods, components等，选项会被合并，键冲突的组件会覆盖混入对象的（比如，混入对象和组件中都有一个func方法，那结果就是组件的这个方法会覆盖混入对象的这个方法）
3. 值为函数的选项，如created, mounted等，就会被合并调用，混入对象里的钩子函数在组件里的钩子函数之前调用
:::

**与vuex的区别**
```
- vuex: 用来做状态管理的，里面定义的变量在每个组件中均可以使用和修改，在任一组件中修改此变量的值之后，其他组件中此变量的值也会随之修改。
- Mixins: 可以定义公用的变量，在每个组件中使用，引入组件中之后，各个变量是相互独立的，值的修改再组件中不会相互影响。
```

**与公共组件的区别**
```
- 组件：在父组件中引入组件，相当于在父组件中给出一片独立的空间供子组件使用，然后根据props来传值，但本质上两者是相对独立的。
- Mixins：则是在引入组件之后与组件中的对象和方法进行合并，相当于扩展了父组件的对象与方法，可以理解为形成了一个新的组件。
```

## Vue.use()

场景：我们使用element时会先import，再Vue.use()一下，实际上就是注册组件，触发install方法；
这个在组件调用会经常使用到；
会自动组织多次注册相同的插件。

## Vue.nextTick

场景：页面加载时需要让文本框获取焦点
用法：在下次DOM更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的DOM

```vue
mounted() { //因为 mounted 阶段 DOM 并未渲染完毕，所以需要$nextTick
  this.$nextTick(() => {
    this.$refs.input.focus() //通过 $refs 获取 DOM 并绑定 focus 方法
  })
}
```

## v-cloak

场景：在网速慢的情况下，在使用vue绑定数据的时候，渲染页面时会出现**变量闪烁**
用法：这个指令保持在元素上直到关联实例结束编译。和css规则如[v-cloak]{display:none}一起使用时，这个指令可以隐藏未编译的Mustache标签直到实例准备完毕

```javascript
// template 中
<div class="app" v-cloak>
  <p>{{value.name}}<p>
</div>

// css中
[v-cloak] {
  display: none;
}
```
**这样就可以解决闪烁，但是会出现白屏，这样可以结合骨架屏使用**

## v-once

场景：有些template中的静态DOM没有改变，这时就只需要渲染一次，可以降低性能开销
```html
<span v-once>这是只需要加载一次的标签</span>
```
<code>v-once</code>和<code>v-pre</code>的区别: 
<code>v-once</code>只渲染一次；<code>v-pre</code>不编译，原样输出

## 事件修饰符

```
.stop:阻止冒泡
.prevent:阻止默认行为
.self:仅绑定元素自身触发
.once: 2.1.4 新增,只触发一次
.passive: 2.3.0 新增,滚动事件的默认行为 (即滚动行为) 将会立即触发,不能和.prevent 一起使用
```