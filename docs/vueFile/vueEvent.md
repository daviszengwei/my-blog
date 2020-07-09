---
title: 'VUE实战技巧'
date: 2020-06-28
tags:
- vue
categories: 
- vue技巧
sidebarDepth: 2
---

:::tip
两只黄鹂鸣翠柳，一堆bug上西天
:::

## hookEvent,原来可以这样监听组件生命周期
1. 内部监听生命周期函数
```javascript
export default {
  mounted() {
    this.chart = echarts.init(this.$el)
    // 请求数据，赋值数据 等等一系列操作...
    
    // 监听窗口发生变化，resize组件
    window.addEventListener('resize', this.$_handleResizeChart)
    // 通过hook监听组件销毁钩子函数，并取消监听事件
    this.$once('hook:beforeDestroy', () => {
      window.removeEventListener('resize', this.$_handleResizeChart)
    })
  },
  updated() {},
  created() {},
  methods: {
    $_handleResizeChart() {
      // this.chart.resize()
    }
  }
}
```

**在Vue组件中，可以用过<code>$on</code>, <code>$once</code>去监听所有的生命周期钩子函数，如监听组件的<code>updated</code>钩子函数可以写成 <code>this.$on('hook:updated', () => {})</code>**

2.外部监听生命周期函数  
<code>Vue</code>支持在外部监听组件的生命周期钩子函数  
```vue
<template>
  <!--通过@hook:updated监听组件的updated生命钩子函数-->
  <!--组件的所有生命周期钩子都可以通过@hook:钩子函数名 来监听触发-->
  <custom-select @hook:updated="$_handleSelectUpdated" />
</template>
<script>
import CustomSelect from '../components/custom-select'
export default {
  components: {
    CustomSelect
  },
  methods: {
    $_handleSelectUpdated() {
      console.log('custom-select组件的updated钩子函数被触发')
    }
  }
}
</script>
```