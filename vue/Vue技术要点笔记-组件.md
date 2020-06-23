### 组件基础
- 1、data 必须是一个函数
  一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝。
  ```js
    // 定义一个名为 button-counter 的新组件
    Vue.component('button-counter', {
    data: function () {
        return {
        count: 0
        }
    },
    template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
    })
  ```

  - 2、全局注册
    
    组件都只是通过 Vue.component 全局注册。全局注册的组件可以用在其被注册之后的任何 (通过 new Vue) 新创建的 Vue 根实例，也包括其组件树中的所有子组件的模板中。
    Vue.component('my-component-name', {
        // ... options ...
    })