#### 创建vue应用 -> vite+vue3
参考：https://vuejs.org/guide/quick-start.html#creating-a-vue-application
```
npm init vue@latest 

运行：npm install && npm run dev
打包：npm run build
```

Note: 
- JSX: javascript拓展语法 
- Pinia：同vuex
- Vitest: 测试框架，见：https://juejin.cn/post/7078906878779981832

#### 步骤分析
##### index.html -> 引入main文件和定义id为app的div
```
<body>
  <div id="app"><div>
  <script type="module" src="/src/main.ts">
<body>
```
Note:
- type="module"标注意味着可以在script标签内使用import

##### main.ts 创建vue实例并挂载
```
import {createApp} from 'vue'
import App from './App.vue'

// 参数App.vue作为根组件传入，并实例化vue
const app = createApp(App)

// 挂载vue实例才能实现页面渲染，参数是一个DOM元素。
app.mount('#app')
```
#### 定义一个子组件定义模板HelloWorld.vue，放于/src/components下
<template>
  <div>
    {{ greeting }}
  </div>
<template>

<script>
  export default {
    data() {
      return {
        greeting: 'Hello World!'
      }
    }
  }
</script>

#### App.vue -> 根组件调用子组件

<script>
// 引用子组件
import HelloWorld from './component/HelloWorld.vue'
</script>

<template>
  <!-- 调用子组件-->
  <HelloWorld/>
</template>

