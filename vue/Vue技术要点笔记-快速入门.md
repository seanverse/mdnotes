## Vue 技术要点笔记

### 基础入门
- 1、**声明式渲染**-插值 

```html
<div id="app">
  {{ message }}
</div>
```

```js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

- 2、**声明式渲染**-指令。v-bind attribute 被称为指令。指令带有前缀 v-,以表示Vue提供的特殊Attribute。

```html
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>
```

- 3、**条件与循环** v-if, v-for等
 
 ```html
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
 ``` 

- 4、**处理用户事件**-v-on 指令添加一个事件监听器
  
```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转消息</button>
</div>
```

```js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```

- 5、v-model 指令，它能轻松实现表单输入和应用状态之间的双向绑定。
  
```html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```

```js
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```

### 组件化

- 1、定义组件：在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例。
  
```js

// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

//使用组件到模板中
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供 todo 对象todo 对象是变量，即其内容可以是动态的。
      我们也需要为每个组件提供一个“key”，稍后再作详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})

```

### Vue实例的生命周期图示和生命周期钩子

 [生命周期图示](https://cn.vuejs.org/images/lifecycle.png)

### 模板语法

+ 1、插值
   + 文本  `<span>Message: {{ msg }}</span>`    
    通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。
    <span v-once>这个将不会改变: {{ msg }}</span>
   + 原始Html     
    双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：   
    
    ```html   
      <p>Using mustaches: {{ rawHtml }}</p>
      <p>Using v-html directive: <span v-html="rawHtml"></span></p>
    ```

   + Attribute v-bind指令
 + 
    ```html
      <div v-bind:id="dynamicId"></div>
      <!--布尔 attribut 语法特殊 -->
      <button v-bind:disabled="isButtonDisabled">Button</button>
    ```

    + 使用 JavaScript 表达式。
      对所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。有个限制就是，每个绑定都只能包含单个表达式。
    
    ```html
   
      {{ number + 1 }}
      {{ ok ? 'YES' : 'NO' }}
      {{ message.split('').reverse().join('') }}
      <div v-bind:id="'list-' + id"></div>
    
      <!--以下是错误的用法示例-->
      <!-- 这是语句，不是表达式 -->
      {{ var a = 1 }}
      <!-- 流控制也不会生效，请使用三元表达式 -->
      {{ if (ok) { return message } }}   

    ```

+ 2、指令
  
> 指令 (Directives) 是带有 v- 前缀的特殊 attribute。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。
  
   + 参数。一些指令能够接收一个“参数”，在指令名称之后以冒号表示。

   ```html
    <a v-bind:href="url">...</a>
    <!--在这里 href 是参数，告知 v-bind 指令将该元素的 href attribute 与表达式 url 的值绑定。-->
   ```

   + 动态参数
    
   从 2.6.0 开始,可以用方括号括起来的 JavaScript 表达式作为一个指令的参数： 
   
   ```html
    <a v-bind:[attributeName]="url"> ... </a>
    <!--这里的 attributeName 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。
    例如，如果你的 Vue 实例有一个 data property attributeName，其值为 "href"，那么这个绑定将等价于 v-bind:href。 
    -->    
    <!-- 你可以使用动态参数为一个动态的事件名绑定处理函数： -->
    <a v-on:[eventName]="doSomething"> ... </a>
    <!-- 在这个示例中，当 eventName 的值为 "focus" 时，v-on:[eventName] 将等价于 v-on:focus。-->   
   ```

   **动态参数的值要注意大小写问题，要求是小写或者存一个大写的property**

   + 修饰符 (modifier)
 
    是以半角句号 . 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如，.prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()：
    
    ```html
       <form v-on:submit.prevent="onSubmit">...</form>
    ```

   + v-指令缩写 v-bind : 和v-on的缩写 @
   
    ```html
      <!-- 完整语法 -->
      <a v-bind:href="url">...</a>
      <!-- 缩写 -->
      <a :href="url">...</a>
      <!-- 动态参数的缩写 (2.6.0+) -->
      <a :[key]="url"> ... </a>
      <!-- 完整语法 -->
      <a v-on:click="doSomething">...</a>
      <!-- 缩写 -->
      <a @click="doSomething">...</a>
      <!-- 动态参数的缩写 (2.6.0+) -->
      <a @[event]="doSomething"> ... </a>
    ```

### 计算属性和侦听器

- 1、计算属性
  - 模板内表达式是很便利，但是放入太多逻辑会让模板过重难以难护。因为，复杂逻辑应当使用**计算属性**。
  
    ```js

      var vm = new Vue({
      el: '#example',
      data: {
          message: 'Hello'
      },
      computed: {
          // 计算属性的 getter
          reversedMessage: function () {
          // `this` 指向 vm 实例
          return this.message.split('').reverse().join('')
          }
      }
      })
    ```

  - 计算属性缓存 vs 方法
  
    方法可以可以模拟出计算属性效果：
    
    ```js
    <p>Reversed message: "{{ reversedMessage() }}"</p>
    
    // 在组件中
    methods: {
    reversedMessage: function () {
        return this.message.split('').reverse().join('')
    } 

    ```

    计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。

  - 计算属性 vs **侦听属性**
  
    ```js

    <div id="demo">{{ fullName }}</div>
    var vm = new Vue({
    el: '#demo',
    data: {
        firstName: 'Foo',
        lastName: 'Bar',
        fullName: 'Foo Bar'
    },
    watch: {
        firstName: function (val) {
        this.fullName = val + ' ' + this.lastName
        },
        lastName: function (val) {
        this.fullName = this.firstName + ' ' + val
        }
    }
    })

    ```
    但是watch属性并进行更新，相比computed的代码更多更杂。

    - 计算属性的setter
    
    ```js

    computed: {
    fullName: {
        // getter
        get: function () {
        return this.firstName + ' ' + this.lastName
        },
        // setter
        set: function (newValue) {
        var names = newValue.split(' ')
        this.firstName = names[0]
        this.lastName = names[names.length - 1]
        }
      }
    }

    ```

    现在再运行 vm.fullName = 'John Doe' 时，setter 会被调用，vm.firstName 和 vm.lastName 也会相应地被更新。

- 2、侦听器
  
  Vue 通过 watch 选项提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

### Class 与 Style 绑定

  将 v-bind 用于 class 和 style 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。

- 1、绑定HTML Class

  ```html
    <div class="static"  v-bind:class="{ active: isActive, 'text-danger': hasError }" />
  ```
  上面的语法表示 active 这个 class 存在与否将取决于数据 property isActive 的 truthiness。‘text-danger’同理。

  - 绑定的数据对象不必内联定义在模板里，而是绑定到一个object data：

  ```html
    <div v-bind:class="classObject"></div>
  ```

  ```js
  data: {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
  ```
  - 我们也可以在这里绑定一个返回对象的计算属性(computed)。这是一个常用且强大的模式：

  ```html
  <div v-bind:class="classObject"></div>
  ```

  ```js
  data: {
    isActive: true,
    error: null
  },
  computed: {
    classObject: function () {
      return {
        active: this.isActive && !this.error,
        'text-danger': this.error && this.error.type === 'fatal'
      }
    }
  }
  ```

  - 我们可以把一个数组传给 v-bind:class，以应用一个 class 列表：

  ```html
  <!--bind到一个data object-->
  <div v-bind:class="[activeClass, errorClass]"></div>
  <!--三元表达式的方式使用 -->
  <div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
  <!--还可以在数组语法中使用对象语法-->
  <div v-bind:class="[{ active: isActive }, errorClass]"></div>
  ```

  ```js
  data: {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
  ```

  - 用在组件上

  当在一个自定义组件上使用 class property 时，这些 class 将被添加到该组件的根元素上面。这个元素上已经存在的 class 不会被覆盖。

  ```js
    Vue.component('my-component', {
      template: '<p class="foo bar">Hi</p>'
    })
  ```

  ```html
    //使用时
    <my-component class="baz boo"></my-component>
    //渲染的结果
    <p class="foo bar baz boo">Hi</p>
  ```

  对于带数据绑定 class 也同样适用：

  ```html
    <my-component v-bind:class="{ active: isActive }"></my-component>
    <!-- 当 isActive 为 truthy 时，HTML 将被渲染成为：-->
    <p class="foo bar active">Hi</p>
  ```

- 2、绑定内联样式
  
  - v-bind:style的语法可以bind到一个js对象：
  
    ```html
    <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
    ```
    ```js
    data: {
      activeColor: 'red',
      fontSize: 30
    }
    ```

  - 直接绑定到一个样式对象通常更好，这会让模板更清晰：
    ```html
      <div v-bind:style="styleObject"></div>
    ```
    ```js
      data: {
        styleObject: {
          color: 'red',
          fontSize: '13px'
        }
      }
    ```

  - 对象语法常常结合返回对象的计算属性使用。
  
  - 数组语法
  
    ```html
      <div v-bind:style="[baseStyles, overridingStyles]"></div>
    ```
  - 自动添加前缀 

    当 v-bind:style 使用需要添加浏览器引擎前缀的 CSS property 时，如 transform，Vue.js 会自动侦测并添加相应的前缀。
  - 指定到多重值
  
    ```html
      <div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
    ```
  这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 display: flex。

### 条件渲染 

- 1、v-if

  ```js
    div v-if="type === 'A'">
      A
    </div>
    <div v-else-if="type === 'B'">
      B
    </div>
    <div v-else-if="type === 'C'">
      C
    </div>
    <div v-else>
      Not A/B/C
    </div>

    //多个元素时
    <template v-if="ok">
      <h1>Title</h1>
      <p>Paragraph 1</p>
      <p>Paragraph 2</p>
    </template>

  ```

  **用Key区分可复用的元素**
    
  ```html
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input placeholder="Enter your username">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Enter your email address">
    </template>
  ```

  上面的代码中切换 loginType 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，<input> 不会被替换掉——仅仅是替换了它的 placeholder。但是有时需要表达这两个元素是完全独立的：

  ```html
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input placeholder="Enter your username" key="username-input">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Enter your email address" key="email-input">
    </template>
  ```

- 2、v-show
    
```html
  <h1 v-show="ok">Hello!</h1>
```

与v-if不同的是带有 v-show 的元素始终会被渲染并保留在 DOM 中。v-show 只是简单地切换元素的 CSS property display。 v-show 不支持 <template> 元素，也不支持 v-else。


### 列表渲染v-for
- 1、常用用法

```html
  <ul id="example-1">
    <li v-for="item in items" :key="item.message">
      {{ item.message }}
    </li>
  </ul>
```

```js
  var example1 = new Vue({
    el: '#example-1',
    data: {
      items: [
        { message: 'Foo' },
        { message: 'Bar' }
      ]
    }
  })
```

```html
  //你也可以用 v-for 来遍历一个对象的 property。
  <div v-for="(value, name) in object">
    {{ name }}: {{ value }}
  </div>
  //还可以用第三个参数作为索引：
  <div v-for="(value, name, index) in object">
    {{ index }}. {{ name }}: {{ value }}
  </div>
```

- 2、数组更新检测
  - Vue 将被侦听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。
    
    这些被包裹过的方法包括：
    - push()
    - pop()
    - shift()
    - unshift()
    - splice()
    - sort()
    - reverse()
 
  - 替换数组
    
    变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 filter()、concat() 和 slice()。它们不会变更原始数组，而总是返回一个新数组。当使用非变更方法时，可以用新数组替换旧数组：
    
    ```js
      example1.items = example1.items.filter(function (item) { return item.message.match(/Foo/) })
    ```
    Vue进行优化而不是重新渲染整个列表，尽量重用一些相同元素的DOM。

  - 显示过滤/排序后的结果
  
    我们想要显示一个数组经过过滤或排序后的版本，而不实际变更或重置原始数据。在这种情况下，可以创建一个计算属性，来返回过滤或排序后的数组。
    
    ```html
        <li v-for="n in evenNumbers">{{ n }}</li>
        <ul v-for="set in sets">
            <li v-for="n in even(set)">{{ n }}</li>
        </ul>
    ```
    ```js
        data: {
        numbers: [ 1, 2, 3, 4, 5 ]
        },
        computed: {
        evenNumbers: function () {
            return this.numbers.filter(function (number) {
            return number % 2 === 0
            })
            }
        }
        methods: {
             even: function (numbers) {
             return numbers.filter(function (number) {
                return number % 2 === 0
                })
            }
        }
    ```

  - 在v-for里使用值范围
  
    ```html
      <div>
          <span v-for="n in 10">{{ n }} </span>
      </div>
    ```


### 事件处理

  当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。

- 1、监听事件

  ```html
  <!--1、直接内嵌js代码的方式-->
  <button v-on:click="counter += 1">Add 1</button>
   <!--2、`greet` 是在下面定义的方法名 -->
  <button v-on:click="greet">Greet</button>
  ```

- 2、向方法传递原始的Dom事件
  
  可以用特殊变量 $event 把它传入方法：

  ```html
    <button v-on:click="warn('Form cannot be submitted yet.', $event)">
    Submit
    </button>
  ```
  ```js
    methods: {
    warn: function (message, event) {
        // 现在我们可以访问原生事件对象
        if (event) {
        event.preventDefault()
        }
        alert(message)
    }
    }
  ```

- 3、事件修饰符

  在事件处理程序中调用 event.preventDefault() 或 event.stopPropagation() 是非常常见的需求。尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。Vue.js 为 v-on 提供了事件修饰符。之前提过，修饰符是由点开头的指令后缀来表示的。

  ```html
    <!-- 阻止单击事件继续传播 -->
    <a v-on:click.stop="doThis"></a>

    <!-- 提交事件不再重载页面 -->
    <form v-on:submit.prevent="onSubmit"></form>
    <!-- 修饰符可以串联 -->
    <a v-on:click.stop.prevent="doThat"></a>

    <!-- 只有修饰符 -->
    <form v-on:submit.prevent></form>

    <!-- 添加事件监听器时使用事件捕获模式 -->
    <!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
    <div v-on:click.capture="doThis">...</div>

    <!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
    <!-- 即事件不是从内部元素触发的 -->
    <div v-on:click.self="doThat">...</div>

    <!-- 点击事件将只会触发一次 -->
    <a v-on:click.once="doThis"></a>

    <!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
    <!-- 而不会等待 `onScroll` 完成  -->
    <!-- 这其中包含 `event.preventDefault()` 的情况 -->
    <div v-on:scroll.passive="onScroll">...</div>
    <!--这个 .passive 修饰符尤其能够提升移动端的性能。-->
  ```

  使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 v-on:click.prevent.self 会阻止所有的点击，而 v-on:click.self.prevent 只会阻止对元素自身的点击。

- 4、按键修饰符
 
  ```html
      <!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
      <input v-on:keyup.enter="submit">
      <!--处理函数只会在 $event.key 等于 PageDown 时被调用。-->
      <input v-on:keyup.page-down="onPageDown">
  ```
  > 在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名：
  >   - .enter
  >   - .tab
  >   - .delete (捕获“删除”和“退格”键)
  >   - .esc
  >   - .space
  >   - .up
  >   - .down
  >   - .left
  >   - .right
  > 你还可以通过全局 config.keyCodes 对象自定义按键修饰符别名：

  ```js
    // 可以使用 `v-on:keyup.f1`
    Vue.config.keyCodes.f1 = 112
  ```

- 5、系统修饰符
  
  可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。.ctrl  .alt  .shift .meta （meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。）

  ```html
    <!-- Alt + C -->
    <input v-on:keyup.alt.67="clear">

    <!-- Ctrl + Click -->
    <div v-on:click.ctrl="doSomething">Do something</div>
  ```

  **exact 修饰符允许你控制由精确的系统修饰符组合触发的事件。**

  ```html
    <!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
    <button v-on:click.ctrl="onClick">A</button>

    <!-- 有且只有 Ctrl 被按下的时候才触发 -->
    <button v-on:click.ctrl.exact="onCtrlClick">A</button>

    <!-- 没有任何系统修饰符被按下的时候才触发 -->
    <button v-on:click.exact="onClick">A</button>
  ```

  **鼠标按钮修饰符**
    .left  .right  .middle 这些修饰符会限制处理函数仅响应特定的鼠标按钮。

### 表单输入绑定

 v-model 会忽略所有表单元素的 value、checked、selected attribute 的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 data 选项中声明初始值。
 
 v-model 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：
    - text 和 textarea 元素使用 value property 和 input 事件；
    - checkbox 和 radio 使用 checked property 和 change 事件；
    - select 字段将 value 作为 prop 并将 change 作为事件。
  
- 1、各种表单控件绑定示例

```html
  <!--文本框-->
  <input v-model="message" placeholder="edit me">
  <!--多行文件框-->
  <textarea v-model="message" placeholder="add multiple lines"></textarea>
  <!--checkbox-->
  <input type="checkbox" id="checkbox" v-model="checked">
  <!--radio group-->
  <div id="example-4">
    <input type="radio" id="one" value="One" v-model="picked">
    <label for="one">One</label>
    <br>
    <input type="radio" id="two" value="Two" v-model="picked">
    <label for="two">Two</label>
    <br>
    <span>Picked: {{ picked }}</span>
  </div>

```

  **值绑定**

```html
    <!-- 当选中时，`picked` 为字符串 "a" -->
    <input type="radio" v-model="picked" value="a">

    <!-- `toggle` 为 true 或 false -->
    <input type="checkbox" v-model="toggle">

    <!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
    <select v-model="selected">
      <option value="abc">ABC</option>
    </select>

    <input type="checkbox" v-model="toggle"  true-value="yes"  false-value="no">
    <!-- // 当选中时-->
    vm.toggle === 'yes'
    <!-- // 当没有选中时 -->
    vm.toggle === 'no'

    <!--选择框的选项-->
    <select v-model="selected">
        <!-- 内联对象字面量 -->
      <option v-bind:value="{ number: 123 }">123</option>
    </select>
    <!--// 当选中时-->
    typeof vm.selected // => 'object'
    vm.selected.number // => 123
```

- 2、修饰符

  - .lazy
     model 在每次 input 事件触发后将输入框的值与数据进行同步。你可以添加 lazy 修饰符，从而转为在 change 事件_之后_进行同步。
     
     ```html
        <!-- 在“change”时而非“input”时更新 -->
        <input v-model.lazy="msg">
     ```
  - .number
     
     ```html
      <input v-model.number="age" type="number"> 
     ```
     将用户的输入值转为数值类型，如果这个值无法被 parseFloat() 解析，则会返回原始的值。
  - .trim
     
     ```html
      <input v-model.trim="msg">
     ```


