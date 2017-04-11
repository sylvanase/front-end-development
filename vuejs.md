# Vue.js
## 一：基本使用
### 1.数据渲染
{{参数名}} 此种形式显示的为文本，如果添加了 `v-once` 指令，当数据改变时，插值处的内容不会更新，只会在初始执行一次性地插值。
```html
<div id="app">
  文本：{{ message }}
</div>
```
```javascript
var app = new Vue({
  el: '#app', //容器的id，或类名
  data: {
    message: 'Hello Vue!'
  }
})
```
```html
<div id="app" v-once>
  文本：{{ message }}
</div>
```
需要输出的是html，则使用 `v-html`指令，此时被插入的内容都会被当做 HTML。
```html
<div id="app" v-html="message">
```
可以对数据进行简单的JavaScript 表达式计算，但是限制为，每个绑定都只能包含单个表达式。
```html
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}
```
> 扩展：过滤器，可以对文本进行格式化。
串联的过滤器：{{ message | filterA | filterB }}
JavaScript形式的过滤器：{{ message | filterA('arg1', arg2) }} ，字符串 'arg1' 将传给过滤器作为第二个参数， arg2 表达式的值将被求值然后传给过滤器作为第三个参数。

```html
<!-- in mustaches -->
{{ message | capitalize }}
<!-- in v-bind -->
<div v-bind:id="rawId | formatId"></div>
```
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
```

### 2.为dom绑定属性
为属性赋值时需要使用vue的语法 v-bild:属性名，布尔值的属性也适用，eg：v-bind:disabled
```html
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>
```
```javascript
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date()
  }
})
```
> 以 `v-` 开头的是Vue提供的属性，不同的属性是不同的指令。例如 `v-bind` 作用为将绑定的dom属性值与data中的值保持一致。

指令的修饰符，用于指出一个指令应该以特殊方式绑定。例如，.prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()
```html
<form v-on:submit.prevent="onSubmit"></form>
```
#### 计算属性
基础的属性计算，计算缓存与method。
```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>声明的计算属性reversedMessage: "{{ reversedMessage }}"</p>
  <p>使用method进行计算: "{{ reversedMessage() }}"</p>
</div>
```
```javascript
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
  },
  methods: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
```
> 两种形式对比：如果计算属性的依赖没有发生改变，就不会再次执行函数。无论message有没有发生变化，只要重新渲染，method 调用就会执行函数。
#### watch 属性：观察和响应 Vue 实例上的数据变动
当需要监测多个值时，且多个值直接有依赖关系时，建议使用computed 属性，而不是watch属性，如下：
```javascript
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
```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: { //使用computed只需要为fullname设置方法即可
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```
watcher扩展：自定义的 watcher ，想要在数据变化响应时，执行异步操作或开销较大的操作。
```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```
```html
<!-- Since there is already a rich ecosystem of ajax libraries    -->
<!-- and collections of general-purpose utility methods, Vue core -->
<!-- is able to remain small by not reinventing them. This also   -->
<!-- gives you the freedom to just use what you're familiar with. -->
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
    // 学习更多关于 _.debounce function (and its cousin
    // _.throttle), 参考: https://lodash.com/docs#debounce
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
#### setter
计算属性默认只有getter，可以手动为其添加setter
```javascript
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
fullName的值是根据firstName和lastName，通过get函数计算得出，如果为fullName赋值 vm.fullName = 'John Doe'，此时将会调用fullName下的set，为firstName和lastName赋值。
### 3.Class 与 Style 绑定
使用 `v-bind` 绑定class或style时，表达式的结果类型除了字符串之外，还可以是对象或数组。
#### 绑定class
```html
<div class="static"
     v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>
<!-- 结果为 -->
<div class="static active"></div>
```
```javascript
data: {
  isActive: true,
  hasError: false
}
```
直接绑定一个对象：
```html
<div v-bind:class="classObject"></div>
```
```javascript
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```
绑定返回对象的计算属性，常用且功能强大：
```html
<div v-bind:class="classObject"></div>
```
```javascript
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
绑定数组：
```html
<div v-bind:class="[activeClass, errorClass]">
<!-- 结果 -->
<div class="active text-danger"></div>
```
```javascript
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```
条件判断：
```html
<!-- 三元表达式 -->
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
<div v-bind:class="[{ active: isActive }, errorClass]">
```
> 以上形式均适用于为组件绑定class
```html
<my-component v-bind:class="{ active: isActive }"></my-component>
```
#### 绑定内联样式style
 CSS 属性名可以用驼峰式（camelCase）或短横分隔命名（kebab-case）。
 对象语法：
```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
<!-- 或者直接绑定一个样式对象 -->
<div v-bind:style="styleObject"></div>
```
```javascript
data: {
  activeColor: 'red',
  fontSize: 30,
  //将样式都放在一个对象中
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```
数组语法：
```html
<div v-bind:style="[baseStyles, overridingStyles]">
```
> 当 `v-bind:style` 使用需要特定前缀的 CSS 属性时，如 transform ，Vue.js 会自动侦测并添加相应的前缀。

### 4.条件与循环
#### 条件判断：if ~ else
```html
<div id="app-3">
  <p v-if="seen">能看到我么？</p>
  <p v-if="diff == 1">我是条件判断</p>
  <p v-else-if="diff == 2">不满足条件1</p>
  <p v-else>不满足条件1和2，且必须跟在 v-if 或 v-else-if 后才能被正常识别</p>
</div>
```
```javascript
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true,
    diff: 1
  }
})
```

#### for循环
```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```
```javascript
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '学习 CSS' }
    ]
  }
})
```
可以动态的为循环添加数据
```javascript
app4.todos.push({ text: '新项目' })
```
添加了索引的 `v-for` ：
```html
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
`v-for` 中的 in 可以使用 of 代替 (item, index) of items，
> 当 `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级。

在template中使用 `v-for`：
```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider"></li>
  </template>
</ul>
```
对象迭代 `v-for` ：
```html
<ul id="repeat-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
<!-- value为参数值，key为键名，index为索引 -->
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }} : {{ value }}
</div>
```
```javascript
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
```
`v-for`循环时也可以取整：
```html
<div>
  <span v-for="n in 10">{{ n }}</span>
</div>
```
组件中的 `v-for` ，需要使用props来传递数据
```html
<div id="todo-list-example">
 <!-- 输入框，输入待办事项，回车后添加到下方的ul中 -->
  <input
    v-model="newTodoText"
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <!-- 显示待办列表，随着input更新，也可进行删除操作 -->
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:title="todo"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```
```javascript
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\  //取props传递过来的title值
      <button v-on:click="$emit(\'remove\')">X</button>\ 
    </li>\  
    //使用vue自带的emit方法删除元素
  ',
  props: ['title']
})
new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      'Do the dishes',
      'Take out the trash',
      'Mow the lawn'
    ]
  },
  methods: {
    addNewTodo: function () {
      this.todos.push(this.newTodoText)
      this.newTodoText = ''
    }
  }
})
```
#### 用key管理复用元素
下面的例子中，虽然是不同的模板，但是使用的是同样的input元素，如果切换的话，input不会被替换，仅替换placeholder值。
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
使用 key 来声明元素是独立的，不复用元素。
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
此时，label仍是复用的，因为label中并没有添加key属性。

在`v-for`中使用key：
```html
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```
#### v-show
带有 `v-show` 的元素始终会被渲染并保留在 DOM 中，不管初始条件是什么，元素总是会被渲染， 它是简单地切换元素的 CSS 属性 display 。`v-show` 不支持 <template> 语法，也不支持 `v-else`。
```html
<h1 v-show="ok">Hello!</h1>
```
与 `v-if` 相比，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件不太可能改变，则使用 `v-if` 较好。
#### 数组更新
vue中包含对数组的操作方法，调用方法时，更新数组，触发视图更新。与javascript中操作数组的方法类似：push()、pop()、shift()、unshift()、splice()、sort()、reverse()。
```javascript
app.items.push({ message: 'Baz' }); //app：vue实例名称，items--data中的数据名称。
```
上述举例的方法均为变异方法，会更改原始数组。以下方法则会返回一个新数组：filter()、concat()、slice()。
> 以下数组的变动vue监测不到：
1.利用索引直接设置一个项时，例如： vm.items[indexOfItem] = newValue。
2.修改数组的长度时，例如： vm.items.length = newLength。

如果想只改变数组中的一个值时，直接设置vue监测不到，可采用set方法或者splice方法。
```javascript
Vue.set(example1.items, indexOfItem, newValue)
//或者第二种方法
example1.items.splice(indexOfItem, 1, newValue)
```
想改变数组长度时，vue可以监测到，则采用splice
```javascript
example1.items.splice(newLength)
```

#### 显示过滤/排序结果
```html
<li v-for="n in evenNumbers">{{ n }}</li>
```
```javascript
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
或者使用method方法进行过滤：
```html
<li v-for="n in even(numbers)">{{ n }}</li>
```
```javascript
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
### 5.事件监听
使用 `v-on` 绑定事件监听器
```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">逆转消息</button>
  <button v-on:click="say('hi')">传递参数</button>
  <button v-on:click="warn('Form cannot be submitted yet.', $event)">传递参数和dom</button>
</div>
```
```javascript
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function (event) {
      // `this` 在方法里指当前 Vue 实例
      this.message = this.message.split('').reverse().join('')
      // `event` 是原生 DOM 事件
      alert(event.target.tagName)
    },
    say: function (message) {
      alert(message);
    },
    warn: function (message, event) {
      // 现在我们可以访问原生事件对象
      if (event) event.preventDefault()
      alert(message)
    }
  }
})

// 也可以用 JavaScript 直接调用方法
app5.reverseMessage()
```
> `v-on:click` 简写形式 `@click`

#### 事件修饰符
vue中提供了事件修饰符，可以方便实现 event.stopPropagation() 一类的事件。
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
<!-- 点击事件将只会触发一次 -->
<a v-on:click.once="doThis"></a>
```
#### 按键修饰符
因为keyCode较多，vue提供的按键别名：.enter、.tab、.delete (捕获 “删除” 和 “退格” 键)、.esc、.space、.up、.down、
.left、.right、.ctrl、.alt、.shift、.meta
```html
<!-- 只有在 keyCode 是 13 时调用 vm.submit() -->
<input v-on:keyup.13="submit">
<!-- 同上 -->
<input v-on:keyup.enter="submit">
<!-- 缩写语法 -->
<input @keyup.enter="submit">
```
可以通过全局 config.keyCodes 对象自定义按键修饰符别名：
```javascript
// 可以使用 v-on:keyup.f1
Vue.config.keyCodes.f1 = 112
```
> 注意：在Mac系统键盘上，meta对应命令键 (⌘)。在Windows系统键盘meta对应windows徽标键(⊞)。在Sun操作系统键盘上，meta对应实心宝石键 (◆)。在其他特定键盘上，尤其在MIT和Lisp键盘及其后续，比如Knight键盘，space-cadet键盘，meta被标记为“META”。在Symbolics键盘上，meta被标记为“META” 或者 “Meta”。
例如:

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">
<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

#### 表单控件绑定
`v-model` 语法糖，负责监听用户的输入事件以更新数据，在表单控件元素上创建双向数据绑定。
文本：
```html
<!-- 文本 -->
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
<!-- 多行文本 -->
<span>多行文本:</span>
<p style="white-space: pre">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```
复选框：
```html
<!-- 单个复选框 -->
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<!-- 多个复选框 -->
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
<br>
<span>Checked names: {{ checkedNames }}</span>
```
```javascript
new Vue({
  el: '...',
  data: {
    checkedNames: [] //多个复选框，结果：["jack","mike"]
  }
})
```
单选按钮：
```html
<!-- 单选按钮，v-model值设定相同 -->
<div id="example-4" class="demo">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>
```
```javascript
new Vue({
  el: '#example-4',
  data: {
    picked: '' //picked值为选定的单选按钮的value值
  }
})
```
下拉列表：
```html
<!-- 单选下拉框 -->
<div id="example-5" class="demo">
  <select v-model="selected">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
<!-- 多选下拉框 -->
<div id="example-6" class="demo">
  <select v-model="selected" multiple style="width: 50px">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>
```
```javascript
new Vue({
  el: '#example-5',
  data: {
    selected: null //没有value值时，显示option文本
  }
})

new Vue({
  el: '#example-6',
  data: {
    selected: []
  }
})
```
> 对于单选按钮，勾选框及选择列表选项， v-model 绑定的 value 通常是静态字符串（对于勾选框是逻辑值）。如果想绑定 value 到 Vue 实例的一个动态属性上，可以用 `v-bind` 实现。

```html
<!-- 指定复选框选中、未选中状态下所取的值 -->
<input
  type="checkbox" v-model="toggle" v-bind:true-value="a" v-bind:false-value="b" >
<!-- 单选框 -->
<input type="radio" v-model="pick" v-bind:value="a">
<!-- 下拉列表 -->
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```
```javascript
// 当复选框选中时
vm.toggle === vm.a
// 当没有选中时
vm.toggle === vm.b

// 当单选框选中时
vm.pick === vm.a

// 当下拉列表选中时
typeof vm.selected // -> 'object'
vm.selected.number // -> 123
```

修饰符：

.lazy：转变为在 change 事件中同步。
```html
<!-- 在 "change" 而不是 "input" 事件中更新 -->
<input v-model.lazy="msg" >
```
.number：自动将用户的输入值转为 Number 类型。
```html
<input v-model.number="age" type="number">
```
.trim：自动过滤用户输入的首尾空格。
```html
<input v-model.trim="msg">
```
 

## 二：Vue实例
### 构造函数
```javascript
var vm = new Vue({
  // 基本构造函数
})

var MyComponent = Vue.extend({
  // 扩展构造函数，扩展选项
})
// 所有的 `MyComponent` 实例都将以预定义的扩展选项被创建
var myComponentInstance = new MyComponent()
```
>Vue实例会代理data对象中的所有属性，有这些被代理的属性是响应的。如果在实例创建之后添加新的属性到实例上，它不会触发视图更新。
```javascript
var data = { a: 1 }
var vm = new Vue({
  data: data
})
vm.a === data.a // -> true
vm.a = 2 // 设置属性也会影响到原始数据
data.a // -> 2
data.a = 3 // 为原始数据赋值，也会影响属性值
vm.a // -> 3
```
>带有前缀$的属性和方法为Vue 实例暴露出来的，以便与data区分开来
```javascript
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})
vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true
// $watch 是一个实例方法
vm.$watch('a', function (newVal, oldVal) {
  // 这个回调将在 `vm.a`  改变后调用
})
```
生命周期
![生命周期][1]


  [1]: https://cn.vuejs.org/images/lifecycle.png


## 三：组件
### 1. 注册组件
```html
<div id="app-1">
  <ol>
    <!-- 创建一个 my-component 组件的实例 -->
    <my-component v-for="item in groceryList" v-bind:todo="item"></my-component>
  </ol>
</div>
```
```javasvript
// 定义名为 my-component 的组件，建议组件名命名规则为：小写，并且包含一个短杠
Vue.component('my-component', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
//确保在实例化前注册组件
var app7 = new Vue({
  el: '#app-1',
  data: {
    groceryList: [
      { text: '蔬菜' },
      { text: '奶酪' },
      { text: '随便其他什么人吃的东西' }
    ]
  }
})
```
局部注册：
```html
<div id="app">
   <my-component></my-component>
</div>
```
```javascript
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  el: '#app',
  components: { //使用components来声明模板
    // <my-component> 将只在父模板可用
    'my-component': Child
  }
})
```
> 在一些dom元素中，其包裹的子孙元素被限制，例如：ul、ol、table、select等，在这些元素中引用组件，会出现解析错误。

```html
<!-- 解析错误，my-row会被认为是无效的内容 -->
<table>
  <my-row>...</my-row>
</table>
<!-- 正确引用方式，使用is -->
<table>
  <tr is="my-row"></tr>
</table>
```
组件中的data必须是函数。
```html
<div id="app">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
```
```javascript
var data = { counter: 0 }
Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () { //在组件中，data只能为函数
    return {
      counter: 0
    }
  }
})
new Vue({
  el: '#app'
})
```

### 2. prop 说明
> 组件中使用props来传递数据，在调用组件时，给props赋值，如果传递的是普通的字符串，在标签中直接使用props，不需要再`v-bind`。
```html
<child message="hello!"></child>
```
```javascript
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 可以用在模板内
  // 同样也可以在 vm 实例中像 “this.message” 这样使用
  template: '<span>{{ message }}</span>'
})
```
>驼峰式命名的prop在模板调用时，要改为短横线隔开
```html
<child my-message="hello!"></child>
```
```javascript
Vue.component('child', {
  props: ['myMessage'], //驼峰式
  template: '<span>{{ myMessage }}</span>'
})
```
>动态地绑定父组件的数据到子模板的props，需要使用 `v-bind` 和 `v-model`
```html
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```
```javascript
Vue.component('child', {
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```
>字面量语法与动态语法，字面的prop传递的都是字符串，使用 `v-bind` 让prop的值被当作 JavaScript 表达式计算
```html
<!-- 传递了一个字符串"1" -->
<comp some-prop="1"></comp>
```
```html
<!-- 传递实际的mumber -->
<comp v-bind:some-prop="1"></comp>
```
>prop 是单向绑定，父组件的属性变化时，将传导给子组件，所以子组件无法改变父组件的状态。如果需要改变子组件prop中的数据，有以下两种情况：

1.子组件想把prop当作局部数据来用。操作：定义一个局部变量，并用 prop 的值初始化它。
```javascript
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
```
2.由子组件处理成其它数据输出。操作：定义一个计算属性，处理 prop 的值并返回。
```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
>PS:如果 prop 是一个对象或数组，在子组件内部改变它会影响父组件的状态。

prop数据格式校验，此时传入props的值与设置的格式不符，vue将抛出警告
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
```
### 3. 自定义事件
父组件使用props将数据传给子组件，子组件利用自定义事件将数据传递回父组件。
使用 **\$on(eventName)** 监听事件，**\$emit(eventName)** 触发事件，父组件可以在使用子组件的地方直接用 `v-on` 来监听子组件触发的事件。

> ps: 不能用 **\$on** 侦听子组件抛出的事件，而必须在模板里直接用v-on绑定，如下例所示。

```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <!-- 调用组件时绑定v-on，监听子组件事件 -->
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```
```javascript
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: { //组件按钮自己的点击事件increment
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {//监听到子组件抛出的事件后，触发该事件
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```
监听组件的根元素上的个原生事件，使用 `.native`：
```html
<my-component v-on:click.native="doTheThing"></my-component>
```
使组件中的 `v-model` 生效，需满足条件：接受一个 value 属性，在有新的 value 时触发 input 事件。如下：
```html
<div id="app">
  <currency-input 
    label="Price" 
    v-model="price"
  ></currency-input>
  <currency-input 
    label="Shipping" 
    v-model="shipping"
  ></currency-input>
  <currency-input 
    label="Handling" 
    v-model="handling"
  ></currency-input>
  <currency-input 
    label="Discount" 
    v-model="discount"
  ></currency-input>
  
  <p>Total: ${{ total }}</p>
</div>
```
```javascript
Vue.component('currency-input', {
  template: '\
    <div>\
      <label v-if="label">{{ label }}</label>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
        v-on:focus="selectAll"\
        v-on:blur="formatValue"\
      >\
    </div>\
  ',
  props: {
    value: {
      type: Number,
      default: 0
    },
    label: {
      type: String,
      default: ''
    }
  },
  mounted: function () {
    this.formatValue()
  },
  methods: {
    updateValue: function (value) {
      var result = currencyValidator.parse(value, this.value)
      if (result.warning) {
        this.$refs.input.value = result.value
      }
      this.$emit('input', result.value)
    },
    formatValue: function () {
      this.$refs.input.value = currencyValidator.format(this.value)
    },
    selectAll: function (event) {
      // Workaround for Safari bug
      // http://stackoverflow.com/questions/1269722/selecting-text-on-focus-using-jquery-not-working-in-safari-and-chrome
      setTimeout(function () {
        event.target.select()
      }, 0)
    }
  }
})

new Vue({
  el: '#app',
  data: {
    price: 0,
    shipping: 0,
    handling: 0,
    discount: 0
  },
  computed: {
    total: function () {
      return ((
        this.price * 100 + 
        this.shipping * 100 + 
        this.handling * 100 - 
        this.discount * 100
      ) / 100).toFixed(2)
    }
  }
})
```