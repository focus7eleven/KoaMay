#### 组件编码规范

##### 组件命名

- 有意义的
- 简短
- 具有可读性



##### 组件表达式简单化

- 减少行内表达式
- 多用method或者computed



##### 组件props原子化

- 避免使用复杂的对象
- 尽量只使用js原始类型（string，number，boolean）
- 就是把一个复杂的configObj都拆分开来，作为多个独立属性传入组件



##### 组件结构化

```jsx
<template lang="html">
  <div class="Ranger__Wrapper">
    <!-- ... -->
  </div>
</template>

<script type="text/javascript">
  export default {
    // 不要忘记了 name 属性
    name: 'RangeSlider',
    // 组合其它组件
    extends: {},
    // 组件属性、变量
    props: {
      bar: {}, // 按字母顺序
      foo: {},
      fooBar: {},
    },
    // 变量
    data() {},
    computed: {},
    // 使用其它组件
    components: {},
    // 方法
    watch: {},
    methods: {},
    // 生命周期函数
    beforeCreate() {},
    mounted() {},
  };
</script>

<style scoped>
  .Ranger__Wrapper { /* ... */ }
</style>
```



##### 谨慎使用this.$refs

- Vue.js 支持通过 `ref` 属性来访问其它组件和 HTML 元素。并通过 `this.$refs` 可以得到组件或 HTML 元素的上下文。在大多数情况下，通过 `this.$refs`来访问其它组件的上下文是可以避免的。
- 尽量使用props和事件传递来实现相应的功能。



##### 提供组件API文档，demo

- 每个组件都要写一个README.md
- 文档应包括，组件的功能，组件的接口描述等



##### 校验组件代码

- ESLint解析.vue文件：`eslint src/xx/xxx.vue`

- JSHint解析HTML：

  `jshint --config modules/.jshintrc --extra-ext=html --extract=auto modules/`





---

#### 组件化应用构建

```javascript
Vue.component('todo-item', {
  // todo-item 组件现在接受一个
  // "props"，类似于一个自定义属性
  // 这个属性名为 todo。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { text: '蔬菜' },
      { text: '奶酪' },
      { text: '随便其他什么人吃的东西' }
    ]
  }
})
```

```html
<div id="app-7">
  <ol>
    <!-- 现在我们为每个todo-item提供待办项对象    -->
    <!-- 待办项对象是变量，即其内容可以是动态的 -->
    <todo-item v-for="item in groceryList" v-bind:todo="item"></todo-item>
  </ol>
</div>
```



#### 构造器

```javascript
// 创建一个Vue实例

var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$el === document.getElementById('example') // -> true
vm.$data === data // -> true

vm.a === data.a // -> true

// 设置属性也会影响到原始数据
vm.a = 2
data.a // -> 2

// 只有这些被代理的属性是响应的。如果在实例创建之后添加新的属性到实例上，它不会触发视图更新。
```

可以扩展 `Vue` 构造器，从而用预定义选项创建可复用的**组件构造器**：

```javascript
var MyComponent = Vue.extend({
  // 扩展选项
})
// 所有的 `MyComponent` 实例都将以预定义的扩展选项被创建
var myComponentInstance = new MyComponent()
```



#### 实例生命周期

下面的created方法在实例被创建时被调用

```javascript
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// -> "a is: 1"
```

beforeCreate - created - beforeMount - mounted - beforeUpdate - updated - beforeDestroy - destroyed



#### 模板语法

##### 插值

- html标签的属性插值，用**v-bind**。`<a v-bind:href="someUrl" />`
- 其余文本用mustache语法：{{ todo.text }}
- 所有的插值都支持js的单一表达式



##### 指令：带有 `v-` 前缀的特殊属性。

- 修饰符：监听submit事件时，调用event.preventDefault()

  `<form v-on:submit.prevent="onSubmit"></form>`

- 属性：v-bind和v-on来监听标签的某个属性的值或属性对应的事件



##### 过滤器

- 只能用于两个地方：**mustache插值**和**v-bind**
- 过滤器设计目的就是用于**文本转换**。为了在其他指令中实现更复杂的数据变换，应该使用**计算属性**。
- 过滤器可以串联

```javascript
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})

<!-- in mustaches -->
{{ message | capitalize }}

{{ message | filterA('arg1', arg2) }} // 过滤器是function，所以可以接受参数

<!-- in v-bind -->
<div v-bind:id="rawId | formatId"></div>
```





#### 计算属性

存在于vue实例中的一系列function

```javascript
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

- 当 `vm.message` 发生改变时，所有依赖于 `vm.reversedMessage` 的绑定也会更新。

- 计算属性的 getter 是没有副作用，这使得它易于测试和推理。

- 计算属性是一种缓存，只要所依赖的属性不变化，就**不会重新渲染**。如果不需要缓存，可以用method替代。

- 计算属性提供getter，也可以自定义setter

  ```javascript
  // ...
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
  // ...
  ```

  运行 `vm.fullName = 'John Doe'` 时， setter 会被调用， `vm.firstName` 和 `vm.lastName` 也相应地会被更新。



##### 观察watchers

使用 `watch` 选项允许我们执行**异步操作**（访问一个 API），限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。**这是计算属性无法做到的**。

```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```

```javascript
<script src="https://unpkg.com/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://unpkg.com/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 question 发生改变，这个函数就会运行
    question: function (newQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce 是一个通过 lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问yesno.wtf/api的频率
    // ajax请求直到用户输入完毕才会发出
    getAnswer: _.debounce(
      function () {
        var vm = this
        if (this.question.indexOf('?') === -1) {
          vm.answer = 'Questions usually contain a question mark. ;-)'
          return
        }
        vm.answer = 'Thinking...'
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Error! Could not reach the API. ' + error
          })
      },
      // 这是我们为用户停止输入等待的毫秒数
      500
    )
  }
})
</script>
```







#### Class和Style绑定

##### 绑定class的对象写法

```html
<div class="static"
	v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>

// 本质上，bind的是一个object
```

当isActive为true，hasError为false时，render成如下html：

```html
<div class="static active"></div>
```



类似的写法如下：

```jsx
<div v-bind:class="classObject"></div>

data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}

// 这里的classObject还可以是一个计算属性
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal',
    }
  }
}
```



##### 绑定class的数组写法

```jsx
<div v-bind:class="[activeClass, errorClass]">

data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
  
// 渲染为：
<div class="active text-danger"></div>
```

条件切换写法：

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
```

数组写法中嵌套对象写法：

```html
<div v-bind:class="[{ active: isActive }, errorClass]">
```





##### 绑定style

- 对象写法

  ```jsx
  <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

  data: {
    activeColor: 'red',
    fontSize: 30
  }
  ```


  // 直接绑定一个object会更好
  <div v-bind:style="styleObject"></div>

  data: {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
  ```

- 数组写法

  ```html
  <div v-bind:style="[baseStyles, overridingStyles]">
  ```

- 当 `v-bind:style` 使用需要特定前缀的 CSS 属性时，如 `transform` ，Vue会自动侦测并添加相应的前缀。在runtime检测是否需要添加。





#### 条件渲染

- `v-else`需要紧跟在`v-if`后面

- 多个需要条件渲染的html块可以用`template`包裹

  ```html
  <template v-if="ok">
    <h1>Title</h1>
    <p>Paragraph 1</p>
    <p>Paragraph 2</p>
  </template>
  ```

- 通过添加key值，来告诉Vue是否需要复用元素。默认情况下是会复用的。

  ```html
  <template v-if="loginType === 'username'">
    <label>Username</label>
    <input placeholder="Enter your username" key="username-input">
  </template>
  <template v-else>
    <label>Email</label>
    <input placeholder="Enter your email address" key="email-input">
  </template>

  // 该例子下，loginType变化之后，会渲染新的input标签
  ```

- `v-show`的元素始终会被渲染并保留在 DOM 中。只是简单地切换了元素的display属性。

- `v-if` 是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

- `v-if` 是“真正的”条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被**销毁和重建**。





#### 列表渲染

- 支持index参数

  ```jsx
  <ul id="example-2">
    <li v-for="(item, index) in items">
      {{ parentMessage }} - {{ index }} - {{ item.message }}
    </li>
  </ul>

  var example2 = new Vue({
    el: '#example-2',
    data: {
      parentMessage: 'Parent',
      items: [
        { message: 'Foo' },
        { message: 'Bar' }
      ]
    }
  })
  ```

- 可以用`of`替代`in`。

- 可以使用`<template>`标签

  ```
  <ul>
    <template v-for="item in items">
      <li>{{ item.msg }}</li>
      <li class="divider"></li>
    </template>
  </ul>
  ```

- 对象的属性迭代。

  ```jsx
  <ul id="repeat-object" class="demo">
    <li v-for="value in object">
      {{ value }}
    </li>
  </ul>

  new Vue({
    el: '#repeat-object',
    data: {
      object: {
        FirstName: 'John',
        LastName: 'Doe',
        Age: 30
      }
    }
  })

  // 也可以提供第二个的参数为键名，第三个参数为索引：
  <div v-for="(value, key, index) in object">
    {{ index }}. {{ key }} : {{ value }}
  </div>
  ```

- 整数迭代

  ```html
  <div>
    <span v-for="n in 10">{{ n }}</span>
  </div>
  ```

- key值：为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素。

  > 当 Vue.js 用 `v-for` 正在更新已渲染过的元素列表时，它默认用 “就地复用” 策略。如果数据项的顺序被改变，Vue将不是移动 DOM 元素来匹配数据项的顺序， 而是简单复用此处每个元素，并且确保它在特定索引下显示已被渲染过的每个元素。

  ```html
  <div v-for="item in items" :key="item.id">
    <!-- 内容 -->
  </div>
  ```



##### 数组更新检测

- 变异方法：Vue 包含一组观察数组的变异方法，所以它们也将会触发视图更新。

  - push()
  - pop()
  - shift()
  - unshift()
  - splice()
  - sort()
  - reverse()

- `filter()`, `concat()`, `slice()` 。这些不会改变原始数组，但总是返回一个新数组。当使用非变异方法时，可以用新数组替换旧数组：

  ```javascript
  example1.items = example1.items.filter(function (item) {
    return item.message.match(/Foo/)
  })
  ```

- Vue不能检测以下变动：

  - 当你利用索引直接设置一个项时，例如： `vm.items[indexOfItem] = newValue`

  - 当你修改数组的长度时，例如： `vm.items.length = newLength`

  - 解决方法：

    ```javascript
    // Array.prototype.splice`
    example1.items.splice(indexOfItem, 1, newValue)

    // Vue.set
    Vue.set(example1.items, indexOfItem, newValue)
    ```



##### 过滤/排序数组内容

- 使用计算属性

  ```jsx
  <li v-for="n in evenNumbers">{{ n }}</li>

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
  ```

- 使用method

  ```jsx
  <li v-for="n in even(numbers)">{{ n }}</li>

  data: {
    numbers: [ 1, 2, 3, 4, 5 ]
  },
  methods: {
    even: function (numbers) {
      return numbers.filter(function (number) {
        return number % 2 === 0
      })
    }
  }
  ```

  ​

#### 事件处理器

##### 事件监听

v-on:click="someFunction(props, $event)"

**methods**中，props作为someFunction的第一个参数，event作为第二个参数（不会自动传入）。



##### 修饰符

Vue.js 为 `v-on` 提供了 **事件修饰符**。

- .stop：等同于调用了event.stopPropagation()，阻止事件冒泡。
- .prevent
- .capture：使用捕获模式。
- .self：元素本身（而非子元素）触发时调用。
- .once：事件只触发一次。

```html
<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联  -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>

<!-- 添加事件侦听器时使用事件捕获模式 -->
<div v-on:click.capture="doThis">...</div>

<!-- 只当事件在该元素本身（而不是子元素）触发时触发回调 -->
<div v-on:click.self="doThat">...</div>
```



此外还有若干个**按键修饰符**，可以通过全局 `config.keyCodes` 对象自定义按键修饰符别名。

```jsx
<input v-on:keyup.enter="submit">

<!-- 缩写语法 -->
<input @keyup.enter="submit">

// 可以使用 v-on:keyup.f1
Vue.config.keyCodes.f1 = 112
```



#### 表单控件绑定

##### ❓在文本区域插值( `<textarea></textarea>` ) 并不会生效。

##### 修饰符

- .lazy

  ```html
  <!-- 在 "change" 而不是 "input" 事件中更新 -->
  <input v-model.lazy="msg" >  // onBlur的时候才会更新
  ```

- .number

- .trim

修饰符可以串联。



---

#### 组件

自定义标签名，Vue.js 不强制要求遵循 [W3C规则](https://www.w3.org/TR/custom-elements/#concepts) （小写，并且包含一个短杠），尽管遵循这个规则比较好。

##### 局部注册

Vue.component('my-component', {template: `<div>test</div>`})是一种全局注册。

不必在全局注册每个组件。通过使用组件实例选项注册，可以使组件仅在另一个实例/组件的作用域中可用：

```jsx
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  // ...
  components: {
    // <my-component> 将只在父模板可用
    'my-component': Child
  }
})
```



##### props

- camelCase vs. kebab-case

  HTML 特性是不区分大小写的。所以，当使用的不是字符串模版，camelCased (驼峰式) 命名的 prop 需要转换为相对应的 kebab-case (短横线隔开式) 命名：

  ```jsx
  Vue.component('child', {
    // camelCase in JavaScript
    props: ['myMessage'],
    template: '<span>{{ myMessage }}</span>'
  })

  <!-- kebab-case in HTML -->
  <child my-message="hello!"></child>
  ```

- 动态传递参数到子组件：使用 **v-bind**

  ```jsx
  <div>
    <input v-model="parentMsg">
    <br>
    <child :my-message="parentMsg"></child>
  </div>
  ```

- 字面量语法

  如果想传递一个实际的number，需要使用 `v-bind` ，从而让它的值被当作 JavaScript 表达式计算：

  ```html
  <!-- 传递实际的 number -->
  <comp v-bind:some-prop="1"></comp>
  ```

- 单向数据流

  如果要对props进行修改，应在子组件内初始化局部的data或computed属性。但是，如果props是数组或对象时，单纯地赋值会出问题。应使用深拷贝。

- props验证

  ```javascript
  Vue.component('example', {
    props: {
      // 基础类型检测 （`null` 意思是任何类型都可以）
      propA: Number,
      // 多种类型
      propB: [String, Number],
      // 必传且是字符串
      propC: {
        type: String,
        required: true
      },
      // 数字，有默认值
      propD: {
        type: Number,
        default: 100
      },
      // 数组／对象的默认值应当由一个工厂函数返回
      propE: {
        type: Object,
        default: function () {
          return { message: 'hello' }
        }
      },
      // 自定义验证函数
      propF: {
        validator: function (value) {
          return value > 10
        }
      }
    }
  })

  // type 可以是String, Number, Boolean, Function, Object, Array
  // type 也可以是自定义构造函数，用instanceOf检测
  ```



##### 自定义事件

父组件可以在使用子组件的地方直接用 `v-on` 来监听子组件触发的事件。子组件通过 `this.$emit('eventName')` 提交事件。



##### 非父子组件通信

创建一个 `bus`，它是一个空的vue实例，来完成A和B之间的通信。

```javascript
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('id-selected', 1)

// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```



##### 使用slot分发内容



```html
<!-- 子组件 -->
<div>
  <h2>我是子组件的标题</h2>
  <slot>
    只有在没有要分发的内容时才会显示。 <!-- 此处为备用内容 -->
  </slot>
</div>

<!-- 父组件 -->
<div>
  <h1>我是父组件的标题</h1>
  <my-component>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </my-component>
</div>

<!-- 渲染结果 -->
<div>
  <h1>我是父组件的标题</h1>
  <div>
    <h2>我是子组件的标题</h2>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </div>
</div>
```

`<slot>` 元素可以用一个特殊的属性 `name` 来配置如何分发内容。多个 slot 可以有不同的名字。具名 slot 将匹配内容片段中有对应 `slot` 特性的元素。

可以有一个匿名 slot ，它是**默认 slot** ，作为找不到匹配的内容片段的备用插槽。如果没有默认的 slot ，这些找不到匹配的内容片段将被抛弃。



###### 具名插槽

```html
<!-- 子组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

<!-- 父组件 -->
<app-layout>
  <h1 slot="header">这里可能是一个页面标题</h1>
  <p>主要内容的一个段落。</p>
  <p>另一个主要段落。</p>
  <p slot="footer">这里有一些联系信息</p>
</app-layout>

<!-- 渲染结果 -->
<div class="container">
  <header>
    <h1>这里可能是一个页面标题</h1>
  </header>
  <main>
    <p>主要内容的一个段落。</p>
    <p>另一个主要段落。</p>
  </main>
  <footer>
    <p>这里有一些联系信息</p>
  </footer>
</div>
```





###### 作用域插槽

一种特殊类型的插槽，用作使用一个（能够传递数据到）可重用模板替换已渲染元素。

典型例子：

```jsx
<my-awesome-list :items="items">
  <!-- 作用域插槽也可以是具名的 -->
  <template slot="item" scope="props">
    <li class="my-fancy-item">{{ props.text }}</li>
  </template>
</my-awesome-list>

<!-- my-awesome-list的模板 -->
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- 这里写入备用内容 -->
  </slot>
</ul>
```



##### 动态组件

使用保留的 `<component>` 元素，动态地绑定到它的 `is` 特性，让多个组件可以使用同一个挂载点，并动态切换。

```jsx
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})

<component v-bind:is="currentView">
  <!-- 组件在 vm.currentview 变化时改变！ -->
</component>
```



###### keep-alive

换出去的组件会保留在内存中，可以保留它的状态或避免重新渲染。

```html
<keep-alive>
  <component :is="currentView">
    <!-- 非活动组件将被缓存！ -->
  </component>
</keep-alive>
```





##### 杂项

###### 可复用组件

组件的API主要来自三个接口：props, events, slots

```html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"       <!-- v-on的缩写 -->
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```



###### 子组件索引

使用 `ref` 为子组件指定一个索引 ID 。例如：

```jsx
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>

var parent = new Vue({ el: '#parent' })
// 访问子组件
var child = parent.$refs.profile
```



###### 异步组件

一种按需加载。webpack的例子：

```javascript
Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 require 语法告诉 webpack
  // 自动将编译后的代码分割成不同的块，
  // 这些块将通过 Ajax 请求自动下载。
  require(['./my-async-component'], resolve)
})
```



###### 组件命名约定

- HTML模板中，使用kebab-case
- 字符串模板中，kebab-case、camelCase、TitleCase都可以



###### 组件的循环调用

在其中一个组件中（比如 A ）告诉webpack，“A 虽然需要 B ，但是不需要优先导入 B”。在A的`beforeCreate` 生命周期钩子中去注册B。

```javascript
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue')
}
```



###### ❓内联模板

如果子组件有 inline-template 特性，组件将把它的内容当作它的模板，而不是把它当作分发内容。

```html
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```





###### 对低开销的组件使用v-once

一些静态模板可以缓存起来，提高效率

```jsx
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ... a lot of static content ...\
    </div>\
  '
})
```



---



#### 响应式原理

Vue 将遍历component的`data`参数上所有的属性，并使用 [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 把这些属性全部转为 getter/setter。Object.defineProperty 是仅 ES5 支持，且无法 shim 的特性，这也就是为什么 Vue 不支持 IE8 以及更低版本浏览器的原因。

每个组件实例都有相应的 **watcher** 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 `setter` 被调用时，会通知 `watcher` 重新计算，从而致使它关联的组件得以更新。

Vue **不能检测到对象属性的添加或删除**。由于 Vue 会在初始化实例时对属性执行 `getter/setter` 转化过程，所以属性必须在 `data` 对象上存在才能让 Vue 转换它。

```javascript
var vm = new Vue({
  data:{
  a:1
  }
})
// `vm.a` 是响应的

vm.b = 2
// `vm.b` 是非响应的
```



##### 异步更新队列

data的更新是异步的，如果同一个 watcher 被多次触发，只会一次推入到队列中。跟react的`this.setState({})`是一个道理。

为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 `Vue.nextTick(callback)` 。这样回调函数在 DOM 更新完成后就会调用。例如：

```javascript
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => '没有更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '更新完成'
      })
    }
  }
})
```





#### 自定义指令

全局注册

```javascript
// 注册一个全局自定义指令 v-focus
Vue.directive('focus', {
  // 当绑定元素插入到 DOM 中。
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```

局部注册

```javascript

directives: {
  focus: {
    // 指令的定义---
  }
}
```



##### 钩子函数

指令定义函数提供了几个钩子函数（可选）：

- `bind`: 只调用一次，指令第一次绑定到元素时调用，用这个钩子函数可以定义一个在绑定时执行一次的初始化动作。
- `inserted`: 被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于 document 中）。
- `update`: 被绑定元素所在的模板更新时调用，而不论绑定值是否变化。通过比较更新前后的绑定值，可以忽略不必要的模板更新。
- `componentUpdated`: 被绑定元素所在模板完成一次更新周期时调用。
- `unbind`: 只调用一次， 指令与元素解绑时调用。



钩子函数参数

- el
- binding：name, value, oldValue, expression, arg, modifiers（修饰符）
- vnode：vue编译生成的虚拟节点
- oldNode：上一个虚拟节点，仅在update和componentUpdated钩子中可用
- ​



#### 混合

当组件和混合对象含有同名选项时，这些选项将以恰当的方式混合。同名钩子函数将混合为一个数组，因此都将被调用。混合对象的钩子将在组件自身钩子 **之前** 调用。

值为对象的选项，例如 `methods`, `components` 和 `directives`，将被混合为同一个对象。 两个对象键名冲突时，取**组件**对象的键值对。

```javascript
var mixin = {
  created: function () {
    console.log('混合对象的钩子被调用')
  }
}
new Vue({
  mixins: [mixin],
  created: function () {
    console.log('组件钩子被调用')
  }
})
// -> "混合对象的钩子被调用"
// -> "组件钩子被调用"

var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}
var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})
vm.foo() // -> "foo"
vm.bar() // -> "bar"
vm.conflicting() // -> "from self"
```





