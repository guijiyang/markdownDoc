#<center>vue学习</center>

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [vue学习](#centervue学习center)
  - [1. Vue实例](#1-vue实例)
    - [1.1. 数据与方法](#11-数据与方法)
    - [1.2. 生命周期和钩子](#12-生命周期和钩子)
  - [2. 模板语法](#2-模板语法)
    - [2.1 指令](#21-指令)
    - [2.2. 插值](#22-插值)
  - [3. 计算属性和监听器](#3-计算属性和监听器)
    - [3.1. 计算属性](#31-计算属性)
    - [3.2. 监听器](#32-监听器)
  - [4. HTML Class和Style绑定](#4-html-class和style绑定)
    - [4.1. 绑定class](#41-绑定class)
    - [4.2. 绑定内联样式](#42-绑定内联样式)
  - [5. 条件渲染](#5-条件渲染)
    - [5.1. v-if](#51-v-if)
    - [5.2. v-show](#52-v-show)
  - [6. 列表渲染](#6-列表渲染)
    - [6.1. 数组更新检测](#61-数组更新检测)
    - [6.2. 对象变更检测](#62-对象变更检测)
  - [7. 事件处理](#7-事件处理)
  - [8. 表单输入绑定](#8-表单输入绑定)
    - [8.1. 基本用法](#81-基本用法)
    - [8.2. 值绑定](#82-值绑定)
  - [9. 组件](#9-组件)
    - [9.1. 组件基础](#91-组件基础)
    - [9.2 组件注册](#92-组件注册)
    - [9.3. Prop](#93-prop)
      - [9.3.1. Prop的大小写(camelCase vs kebab-case)](#931-prop的大小写camelcase-vs-kebab-case)
      - [9.3.2. Prop类型](#932-prop类型)
      - [9.3.3. 传递静态或动态Prop](#933-传递静态或动态prop)
      - [9.3.4. 单向数据流](#934-单向数据流)
    - [9.4 自定义事件](#94-自定义事件)
      - [9.4.1. 事件名](#941-事件名)
      - [9.4.2. 自定义组件的v-model](#942-自定义组件的v-model)
      - [9.4.3. 将原生事件绑定到组件](#943-将原生事件绑定到组件)
      - [9.4.4. 修饰符.sync](#944-修饰符sync)
    - [9.5. 插槽](#95-插槽)
      - [9.5.1. 编译作用域](#951-编译作用域)
      - [9.5.2. 具名插槽](#952-具名插槽)
      - [9.5.3. 作用域插槽](#953-作用域插槽)
      - [9.5.4. 动态插槽名](#954-动态插槽名)
      - [9.5.5. 具名插槽缩写](#955-具名插槽缩写)
    - [9.6. 动态组件 & 异步组件](#96-动态组件-异步组件)

<!-- /code_chunk_output -->

## 1. Vue实例

每个 Vue 应用都是通过用 Vue 函数创建一个新的 Vue 实例开始的：
```js
var vm=new Vue({
    //选项
})
```
通常会使用 `vm` (ViewModel)这个变量名表示Vue实例.
当创建一个 Vue 实例时，你可以传入一个选项对象。如 `el` 和 `data`.

一个 Vue 应用由一个通过 `new Vue` 创建的根 Vue 实例，以及可选的嵌套的、可复用的组件树组成。举个例子，一个 todo 应用的组件树可以是这样的：
```
根实例
└─ TodoList
   ├─ TodoItem
   │  ├─ DeleteTodoButton
   │  └─ EditTodoButton
   └─ TodoListFooter
      ├─ ClearTodosButton
      └─ TodoListStatistics
```

### 1.1. 数据与方法

Vue实例中最常见的 选项是 `data`:
- 类型: Object |Function
- 限制:组件的定义只接受Function
- 详细:
    Vue 实例的数据对象。Vue 将会递归将 data 的属性转换为 getter/setter，从而让 data 的属性能够响应数据变化。对象必须是纯粹的对象 (含有零个或多个的 key/value 对)：浏览器 API 创建的原生对象，原型上的属性会被忽略。大概来说，data 应该只能是数据 - 不推荐观察拥有状态行为的对象。

    一旦观察过，不需要再次在数据对象上添加响应式属性。因此推荐在创建实例之前，就声明所有的根级响应式属性。

    实例创建之后，可以通过 `vm.$data` 访问原始数据对象。Vue 实例也代理了 data 对象上所有的属性，因此访问 `vm.a` 等价于访问 `vm.$data.a`。

    以 `_` 或 `$` 开头的属性 不会 被 Vue 实例代理，因为它们可能和 Vue 内置的属性、API 方法冲突。你可以使用例如 `vm.$data._property` 的方式访问这些属性.

    当一个组件被定义，data 必须声明为返回一个初始数据对象的函数，因为组件可能被用来创建多个实例。如果 data 仍然是一个纯粹的对象，则所有的实例将**共享引用**同一个数据对象！通过提供 data 函数，每次创建一个新实例后，我们能够调用 data 函数，从而返回初始数据的一个全新副本数据对象。


当Vue实例创建时,它将选项对象 `data` 中的所有的属性加入到Vue的响应式系统中.因此当选项对象中的属性改变时,实例也将同步变化.
```js
var data = {a:1}

var vm = new Vue({
    data : data
})

vm.a == data.a // => true

vm.a=2
data.a // => 2

data.a=3
vm.a //=>3
```

当实例中的数据改变时,视图会进行重渲染.但是只有创建实例时候的 `data` 中存在的属性才是响应式的.也就是说新添加一个属性:
```js
vm.b='hi'
```
是不会触发任何视图更新.因此对于后期需要的属性,需要在创建实例的时候就设置初始值:
```js
data: {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}
```

另一个常见的选项是 `el`:
- 类型: string | Element
- 限制: 只在由 `new` 创建的实例中遵守.
- 详细:
    提供一个在页面上已存在的 DOM 元素作为Vue实例的挂载目标.可以是CSS选择器,也可以是一个HTMLElement实例.
    在实例挂载之后,元素可以用 `vm.$el` 访问.
    如果在实例化时存在这个选项,实例将立即进入编译过程,否则,需要显式调用 `vm$mount()`手动开启编译.

通过 `Object.freeze(obj)`,将阻止修改现有的属性,这意味着响应系统无法再追踪变化.
```js
var obj={foo:'bar}

Object.freeze(obj)

new Vue({
    el:'#app'
    data:obj
})
```
```html
<div id='app'>
    <p>{{foo}}</p>
    <button v-on:click="foo = 'baz'">Change it</button>
</div>
```
除了数据属性，Vue 实例还暴露了一些有用的实例属性与方法。它们都有前缀 $，以便与用户定义的属性区分开来。例如：

```js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

### 1.2. 生命周期和钩子

下图展示了实例的整个生命周期.
![lifeciycle](/img/lifecycle.png)

从上图可以看出每个Vue实例在被创建时都要经过一系列的初始化过程--例如,需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做生命周期钩子的函数，这给了用户在不同阶段添加自己的代码的机会。

比如 `created` 钩子可以用来在一个实例被创建之后执行代码：
```js
new Vue({
    data: {
        a:1
    },
    created:function () {
        //this 指向vm实例
        console.log('a is : ' +this.a)
    }
})
```
实例生命周期在不同阶段也有其他的钩子,上图中红色的圆角矩形框中的就是所有的钩子.在生命周期钩子中的 `this` 指向的调用它的Vue实例.

不要在选项属性或回调上使用箭头函数，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 this，this 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。

## 2. 模板语法

Vue.js使用了基于HTML的模板语法,允许开发者声明式地将DOM绑定到底层的Vue实例的数据.所有的Vue.js的模板都是合法的HTML,所以能被遵循规范的浏览器和HTML解析器解析.

在底层的实现上,Vue将模板编译成虚拟DOM渲染函数.结合响应系统,Vue能够计算出最少需要重新渲染多少组件,并把DOM操作次数减到最少.

如果你熟悉虚拟 DOM 并且偏爱 JavaScript 的原始力量，你也可以不用模板，直接写渲染 (render) 函数，使用可选的 JSX 语法。
### 2.1 指令
模板中常常用到指令,指令是带有 `v-`前缀的特殊特性.指令特性的值预期是单个js表达式(`v-for`例外).指令的职责是,当表达式的值变更时,将其产生的连带影响,响应式地作用于DOM.如下:
```html
<p v-if="see">现在可见</p>
```
这里 `v-if` 指令将根据表达式 `see` 的真假来插入/移除`<p>`元素.

<p style="font-weight:bold">参数</p>
一些指令能够接收一个"参数",在指令名称之后以冒号表示.例如,  v-bind 指令可以用于响应式地更新 HTML 特性:
```html
<a v-bind:href="url">...</a>
```
在这里 `href` 是参数,告知 `v-bind`指令将该元素的`href`特性与表达式`url`的值绑定.
另一个例子是 `v-on`指令,它用于监听DOM事件:
```html
<a v-on:click="doSomething">...</a>
```
在这里参数是监听的事件名.
<p style="font-weight:bold">动态参数</p>
指令中的参数也可以用方括号下动态的js表达式:
```html
<a v-bind:[attributeName]='url'>...</a>
```
如果这里的绑定的实例数据有属性 `atrributeName`,其值为`href`,那么这个绑定等价于`v-bin:href`.

动态参数预期会求出一个字符串,异常情况下值为`null`.这个特殊的`null`值可以被显性地用于移除绑定.任何其他非字符串类型的值都将会触发一个警告.
<p style="font-weight:bold">修饰符</p>
修饰符(modifier)是以半角句号.指明的特殊后缀,用于指出一个指令应该以特殊方式绑定.例如,.prevent修饰符告诉`v-on`指令对于触发的事件调用`event.preventDefault()`:
```html
<form v-on:submit.prevent="onSubmit">...</form>
```

<p style="font-weight:bold">指令缩写</p>
Vue为`v-bind`和`v-on`提供缩写:
```html
<!-- v-bind缩写 -->
<a :href="url">...</a>

<!-- v-on缩写 -->
<a @click="doSomething">...</a>
```
### 2.2. 插值

数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：
```html
<span>Message L {{ msg }}</span>
```

Mustache 标签将会被替代为对应数据对象上 msg 属性的值。无论何时，绑定的数据对象上 msg 属性发生了改变，插值处的内容都会更新。

也可以通过`v-once`指令一次性的插入值,然后当数据改变时,插值处的内容不会更新.这将会影响到该节点的所有所有数据绑定:
```html
<span v-once>这个数据属性不会改变 {{msg}}</span>
```

<p style="font-weight:bold">原始HTML</p>
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：
```html
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
这个`span`的内容将会被替换成属性值 `rawHTML`,直接作为HTML.
需要注意的是不能使用`v-html`指令复合局部模板,因为Vue不是基于字符串的模板引擎.
<p style="font-weight:bold">特性</p>
Mustache语法不同作用到HTML特性上,遇到这种情况应该使用`v-bind`指令:
```html
<div v-bind:id="dynamicId"></div>
```
对于bool特性(只要存在就意味着true):
```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```
如果表达式的值是`null`,`undefined`或`false`,则`disabled`特性甚至不会被包含在渲染出来的`<button>`元素中.

<p style="font-weight:bold">使用js表达式</p>
Vue支持js表达式:
```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```
这些表达式会在所属Vue实例的数据作用域下作为javascript被解析.有个限制就是,每个绑定都只能包含单个表达式,所以下面的例子都不会生效:
```html
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

## 3. 计算属性和监听器

### 3.1. 计算属性

在模板中放入太多的逻辑会让模板过重且难以维护.例如:
```html
<div id="example">
{{ message.split('').reverse().join('')}}
</div>
```
所以对于任何复杂逻辑,都应该使用计算实行.如下所示:
```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```
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
结果：
```
Original message: "Hello"

Computed reversed message: "olleH"
```
这里我们声明了一个计算属性`reverseMessage`.我们提供的函数将用作属性`vm.reverseMessage`的 `getter`函数:
```js
console.log(vm.reverseMessage) //=>'olleH'
vm.message='Goodbye'
console.log(vm.reverseMessage)//=>'eybdooG'
```
这里Vue知道计算属性`vm.reverseMessage`依赖于属性`vm.message`,因此当属性改变时,所有依赖计算属性的绑定也会更新.

<p style="font-weight:bold">计算属性缓存VS方法</p>
除了计算属性,我们也可以使用实例中的方法来达到同样的效果:
```html
<p>Reversed message: "{{ reversedMessage() }}"</p>
```
```js
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```
这里我们将定义了一个同名函数而不是计算属性.这两者结果完全相同.然而,不同的是计算属性是基于它们的响应式依赖就行缓存的.也就是说如果这里的属性`vm.message`没有改变,已经求过值的计算属性可以直接使用缓存.

这也意味着下面的计算属性不再更新,因为`Date.now()`不是响应式依赖:
```js
computed: {
    now:function(){
        return Date.now()
    }
}
```
相比之下,每次重新渲染时,调用方法总是再次执行函数.所以获取当前时间必须使用方法.

<p style="font-weight:bold">计算属性缓存VS监听属性</p>
Vue提供另一种更通用的方式来观察和响应Vue实例上的数据变动:监听属性.然而,通常更好的做法是使用计算属性而不是命令式的`watch`回调.如下:
```html
<div id="demo">{{ fullName }}</div>
```
```js
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
可见使用watch监听代码重复较多.对比计算属性:
```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```
明显计算属性要好很多.

<p style="font-weight:bold">计算属性setter</p>
计算属性默认只有`getter`,不过在需要时可以提供一个setter:
```js
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
现在再运行 vm.fullName = 'John Doe' 时，setter 会被调用，vm.firstName 和 vm.lastName 也会相应地被更新。

### 3.2. 监听器

有时候也需要一个自定义的监听器.当需要的数据变化时执行异步或开销较大的操作时,这种方式是最有用的.
例如:
```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
<!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
<!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
    // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
    // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
    // 请参考：https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```

在这个示例中,使用`watch`选项允许我们执行异步操作(访问一个API),限制我们执行该操作的频率,并在我们得到最终结果前,设置中间状态.这些都是计算属性无法做到的.

## 4. HTML Class和Style绑定

对HTML元素按照class列表和内联样式进行数据绑定的一个常见需求.因为它们都是属性,所以我们可以用`v-bind`处理它们.在将`v-bind`用于`class`和`style`时,Vue做了专门的增强.表达式结果的类型除了字符串之外,还可以是对象或数组.

### 4.1. 绑定class

我们可以传给`v-bind:class`一个对象,以动态地切换class:
```html
<div v-bind:class="{active: isAcitve}"></div>
```
上面的语法表示`active`这个class存在与否将取决于数据属性`isActive`的`truthiness`.
你可以在对象中传入更多的属性来动态切换多个class.此外,`v-bind:class`指令也可以与普通的class共存:
```html
<div
    class="static"
    :class="{ active:isActive,'text-danger':hasError}">
</div>
```
```js
data:{
    isActive:true,
    hasError:false
}
```
这样渲染的结果是:
```html
<div class=" static active"></div>
```
当数据属性变化时,class列表也相应的更新.
绑定的数据对象不必内联定义在模板里:
```html
<div v-bind:class="classObject"></div>
```
```js
data: {
    classObject: {
        active: true,
        'text-danger':false
    }
}
```
渲染的结果和上面一样.我们也可以在这里帮一个返回对象的计算属性:
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

<p style="font-weight:bold">绑定数组</p>
```html
<div v-bind:class="[activeClass, errorClass]"></div>
```
```js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```
渲染为:
```html
<div class="active text-danger"></div>
```
也可以:
```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```
或者;
```html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

### 4.2. 绑定内联样式
<p style="font-weight:bold">对象语法</p>
```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```js
data: {
  activeColor: 'red',
  fontSize: 30
}
```
也可以直接绑定到一个样式对象:
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
同样的,对象语法常常结合返回对象的计算属性使用.

<p style="font-weight:bold">数组语法</p>
```html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

## 5. 条件渲染

### 5.1. v-if

`v-if`指令用于条件性地渲染一块内容.这块内容只有在指令表达式为truthy值的时候才被渲染.
```html
<h1 v-if="awesome">Vue is awesome!</h1>
```
添加一个else分支:
```html
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```
使用 `v-else-if`:
```html
<div v-if="type === 'A'">
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
```
如果需要控制多个元素,则使用`<template>`元素当做不可见的包裹元素,并在上面使用`v-if`.最终得到的渲染结果不包含`<template>`元素:
```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

<p style="font-weight:bold">复用元素</p>

Vue会尽可能搞笑地渲染元素,通常会复用已有元素而不是从头开始渲染.如下:
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
因为两个模板使用了相同的元素,`<input>`不会被替换掉,只替换了`placeholder`.

如果希望这两个元素是完全的独立的,只需要添加一个具有唯一值的`key`属性:
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
这里的`input`将是两个独立的元素.而`label`不是.

### 5.2. v-show
`v-show`也是根据条件展示元素.用法如下:
```html
<h1 v-show="ok">Hello!</h1>
```
不同于`v-if`的地方是`v-show`的元素始终都被渲染并保留在DOM中.`v-show`只是简单地切换元素的css属性 `display`.
<font color=red>注意,`v-show `不支持`<template>`元素.</font>

## 6. 列表渲染

我们使用`v-for`指令基于一个数组来渲染一个列表.

```html
<ul id="example-1">
  <li v-for="item in items">
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
结果：
- Foo
- Bar

`v-for`还可以获取当前项的索引:
```html
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
```js
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
结果:
- Parent-0-Foo
- Parent-1-Bar

这里的`in`也可以使用`of`代替,更接近js语法:
```html
<div v-for="item of items"></div>
```

`v-for`也可以用来遍历一个对象的属性:
```html
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
```
```js
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```
结果:
- How to do lists in Vue
- Jane Doe
- 2016-04-10

也可以:
```html
<div v-for="(value, name) in object">
  {{ name }}: {{ value }}
</div>
```
还可以:
```html
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>
```

为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 key 属性：
```html
<div v-for="item in items" v-bind:key="item.id">
  <!-- 内容 -->
</div>
```

### 6.1. 数组更新检测
 
 <p style="font-weight:bold">变异方法(mutation method)</p>

 Vue将被监听的数组的变异方法进行了包裹,所以它们将会触发视图的更新.这些方法包括:

 - push()
 - pop()
 - shift()
 - unshift()
 - splice()
 - sort()
 - reverse()

 调用方式如下:
```js
example1.items.push({message: 'Baz'})
```
变异方法会改变原始数组.通过`filter()`,`concat()`和`slice()`这些非变异方法,可以在不改变原始数组的情况下返回一个新的数组:
```js
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

由于js的限制,Vue不能检测一下数组的变动:
- 当你利用索引直接设置一个数组项时;
- 修改数组的长度时.
```js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```
可以使用一下方式触发视图更新:
```js
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
//实例方法
vm.$set(vm.items, indexOfItem, newValue)
// vm.items.splice(newLength)
```

### 6.2. 对象变更检测
还是由于js的限制,Vue不能检测对象属性的添加或者删除:
```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```
对于这种创建好的实例,Vue不允许动态添加根级别的响应式属性.但是对于嵌套对象则可以添加响应式属性,主要是通过全局API `Vue.set( object, propertyName, value)`,如下:
```js
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})

//global API
Vue.set(vm.userProfile, 'age', 27)
//实例方法
// vm.$set(vm.userProfile, 'age', 27)
```

如果需要给对象添加多个属性,应该用两个对象的属性创建一个新的对象:
```js
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
```

## 7. 事件处理

通过`v-on`指令监听DOM事件,并在触发时运行一些js代码.
```js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```
```html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

<p style="font-weight:bold">事件处理方法</p>
由于许多事件处理逻辑比较复杂,通常在表达式中使用方法:
```html
<div id="example-2">
  <!-- `greet` 是在下面定义的方法名 -->
  <button v-on:click="greet">Greet</button>
</div>
```
```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (event) {
      // `this` 在方法里指向当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
```
有时也需要在内联语句处理器中访问原始的DOM事件.可以用特殊变量`$event`传入方法:
```html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```
```js
// ...
methods: {
  warn: function (message, event) {
    // 现在我们可以访问原生事件对象
    if (event) event.preventDefault()
    alert(message)
  }
}
```
<p style="font-weight:bold">事件修饰符</p>

- .stop
- .prevent
- .capture
- .self
- .once
- .passive
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
<!-- 即元素自身触发的事件先在此处理，然后才交由内部元素进行处理 -->
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
```

<p style="font-weight:bold">按键修饰符</p>

Vue允许为`v-on`在监听键盘事件时添加按键修饰符:
```html
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input v-on:keyup.enter="submit">
<input v-on:keyup.page-down="onPageDown">
```
在上述示例中，处理函数只会在 $event.key 等于 PageDown 时被调用。

## 8. 表单输入绑定

### 8.1. 基本用法

`v-model`可以在表单元素`<input>`,`<textarea>`和`<select>`上创建双向数据绑定.它负责监听用户的输入事件以更新数据,并对一些极端场景进行一些特殊处理.

`v-model`在内部为不同的输入元素使用不同的属性并抛出不同的事件:
- `text`和`textarea`元素使用`value`属性和`input`事件;
- `checkbox`和`radio`使用`checked`属性和`change`事件;
- `select`字段将`value`作为属性并将`change`作为事件.

<p style="font-weight:bold">文本</p>
```html
<!-- 单行 -->
<input v-model="message" placeholder="edit me">
<p>Message is : {{ message }}</p>
<!-- 多行 -->
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```
<p style="font-weight:bold">复选框(checkbox)</p>
```html
<!-- 单个 -->
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
<!-- 多个 -->
<div id='example-3'>
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
```
```js
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
```
<p style="font-weight:bold">单选按钮(radio)</p>
```html
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
```js
new Vue({
  el: '#example-4',
  data: {
    picked: ''
  }
})
```

<p style="font-weight:bold">选择框(select)</p>
```html
<div id="example-5">
  <select v-model="selected">
    <option disabled value="">请选择</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
```
```js
new Vue({
  el: '...',
  data: {
    selected: ''
  }
})
```
选择框也可以多选:
```html
<div id="example-6">
  <select v-model="selected" multiple style="width: 50px;">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>
```
```js
new Vue({
  el: '#example-6',
  data: {
    selected: []
  }
})
```

### 8.2. 值绑定

`v-model`绑定的值通常是静态字符串(对于复选框可以是bool值):
```html
<!-- 当选中时，`picked` 为字符串 "a" -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` 为 true 或 false -->
<input type="checkbox" v-model="toggle">

<!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```
复选框的值也可以使用字符串:
```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```
```js
// 当选中时
vm.toggle === 'yes'
// 当没有选中时
vm.toggle === 'no'
```

单选按钮:
```html
<input type="radio" v-model="pick" v-bind:value="a">
```
```js
// 当选中时
vm.pick === vm.a
```

选择框:
```html
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```
```js
// 当选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

## 9. 组件

### 9.1. 组件基础

Vue中组件本质上是一个拥有预定义选项的Vue实例,注册一个新的组件类似如下:
```js
Vue.component('button-counter' , {
    data:function() {
        return {
            count:0
        }
    },
    template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```
组件是可复用的Vue实例,带有一个名字.我们可以在根实例中把这个组件作为自定义元素来使用:
```html
<div id="components-demo">
  <button-counter></button-counter>
</div>
```
```js
new Vue({ el : "components-demo"})
```
因为组件是可复用的Vue实例,所以它们与`new Vue`接受相同的选项,例如 `data`, `computed`,`watch`,`methods`以及生命周期钩子等.但没有 `el`.

组件可以任意复用,每个组件对应一个实例.

<font color=red>需要注意的是 `data`必须是一个函数,而非一个对象.这样每个实例可以维护一份被返回对象的独立的拷贝.</font>

<p style="font-weight:bold">组件的组织</p>
通常一个应用会以一颗嵌套的组件谁的形式来组织:

![components](/img/components.png)

例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。
为了能在模板中使用，这些组件必须先注册以便 Vue 能够识别。这里有两种组件的注册类型：全局注册和局部注册。至此，我们的组件都只是通过 Vue.component 全局注册的：
```js
Vue.component('my-component-name', {
  // ... options ...
})
```
全局注册的组件可以用在其被注册之后的任何新创建的Vue根实例,也包括其组织树中的所有子组件的模板中.

<p style="font-weight:bold">Prop传递数据给子组件</p>
`props`是可以在组件上注册的一些自定义特性.当一个值传递给一个`props`特性时,他就变成一个组件实例的一个属性.例如:
```js
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})
```
根实例如下:
```js
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```
为每篇博文渲染一个组件:
```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:title="post.title"
></blog-post>
```
这里通过`v-bind`绑定传递prop.

每个组件必须只有一个根元素(single root element).对于需要使用多个元素的,可以将模板内容包裹到一个父元素中:
```html
<!-- 组件实例化 -->
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-post>
```
```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```
现在，不论何时为 `post` 对象添加一个新的属性，它都会自动地在 `<blog-post>` 内可用。

<p style="font-weight:bold">监听子组件事件</p>

父级组件可以像处理 native DOM 事件一样通过 `v-on` 监听子组件实例的任意事件：
```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button v-on:click="$emit('enlarge-text')">
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```
这里子组件通过内建的`$emit`方法来触发一个事件.
```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += 0.1"
></blog-post>
```
这里父组件监听事件`enlarge-text`,在接收到该事件时更新`postFontSize`值.

内建方法`$emit`还提供了第二个参数以供父组件使用:
```html
<button v-on:click="$emit('enlarge-text', 0.1)">
  Enlarge text
</button>
```
在父组件中可以通过`$event`访问到这个被抛出的值:
```html
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```
或者使用一个事件处理方法:
```html
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```
这个值将作为第一个参数传入这个方法:
```js
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

<font color=red>自定义事件也可以用于创建支持 v-model 的自定义输入组件</font>。记住：
```html
<input v-model="searchText">
```
等价于:
```html
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
```
当用在组件上时, v-model 则会这样:
```html
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event"
></custom-input>
```
为了让它正常工作，这个组件内的 `<input>` 必须：

- 将其 value 特性绑定到一个名叫 value 的 prop 上
- 在其 input 事件被触发时，将新的值通过自定义的 input 事件抛出

写成代码就是这样的:
```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```
现在 v-model 就应该可以在这个组件上完美地工作起来了：
```html
<custom-input v-model="searchText"></custom-input>
```

### 9.2 组件注册

到目前为止,我们只用过`Vue.component`来创建组件,这些组件是全局注册的.这些组件在注册后可用于任何新创建的Vue根实例模板中.

全局注册往往是不够理想的.当你使用webpack这样的构建系统,全局注册所有的组件意味着该组件不再使用,仍然会被包含在最终的构建结果中.

这种情况下,可以使用局部注册,通过一个普通的js对象来定义组件:
```js
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }
```
然后在实例中的`components`选项定义需要的组件:
```js
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

使用 ES2015 模块,如果从vue文件中引入组件:
```js
import ComponentA from './ComponentA.vue'

export default {
  components: {
    ComponentA
  },
  // ...
}
```

注意在 ES2015+ 中，在对象中放一个类似 ComponentA 的变量名其实是 ComponentA: ComponentA 的缩写，即这个变量名同时是：

- 用在模板中的自定义元素的名称
- 包含了这个组件选项的变量名

<p style="font-weight:bold">模块系统</p>

- 在模块系统中局部注册

```js
// ComponentB.vue或者Component.js中
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
  // ...
}
```
这样在ComponentB 模板中就可以使用 ComponentA和 ComponentC.

- 基础组件的自动化全局注册

许多组件只是包裹了一个输入框或按钮之类的元素,是相对通用的.这种被称为基础组件,它们在各个组件中被频繁的使用.

这些基础组件如果要通过局部注册将会非常麻烦.这里可以在应用入口文件(如`src/main.js`)使用`require.context`全局注册.如下:
```js
// Globally register all base components for convenience, because they
// will be used very frequently. Components are registered using the
// PascalCased version of their file name.

import Vue from 'vue'

// https://webpack.js.org/guides/dependency-management/#require-context
const requireComponent = require.context(
  // Look for files in the current directory
  '.',
  // Do not look in subdirectories
  false,
  // Only include "_base-" prefixed .vue files
  /_base-[\w-]+\.vue$/
)

// For each matching file name...
requireComponent.keys().forEach((fileName) => {
  // Get the component config
  const componentConfig = requireComponent(fileName)
  // Get the PascalCase version of the component name
  const componentName = fileName
    // Remove the "./_" from the beginning
    .replace(/^\.\/_/, '')
    // Remove the file extension from the end
    .replace(/\.\w+$/, '')
    // Split up kebabs
    .split('-')
    // Upper case
    .map((kebab) => kebab.charAt(0).toUpperCase() + kebab.slice(1))
    // Concatenated
    .join('')

  // Globally register the component
  Vue.component(componentName, componentConfig.default || componentConfig)
})
```
<font color=red size=4px>记住全局注册的行为必须在根 Vue 实例 (通过 new Vue) 创建之前发生</font>。

### 9.3. Prop

#### 9.3.1. Prop的大小写(camelCase vs kebab-case)
HTML 中的特性名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着当你使用 DOM 中的模板时，camelCase (驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名：
```js
Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```
```html
<!-- 在 HTML 中是 kebab-case 的 -->
<blog-post post-title="hello!"></blog-post>
```
如果你使用字符串模板,就没有这个限制.

#### 9.3.2. Prop类型

前面我们已经介绍的prop都是字符串数组形式.如果需要指定每个prop的值类型,可以以对象形式列出prop:
```js
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```
#### 9.3.3. 传递静态或动态Prop

前面介绍过给组件prop传入静态值:
```html
<blog-post title="My Vue"></blog-post>
```
prop也可以通过`v-bind`动态赋值:
```html
<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title"></blog-post>

<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

传入不同类型的prop:
```html
<!-- 即便 `42` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:likes="42"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:likes="post.likes"></blog-post>

<!-- 包含该 prop 没有值的情况在内，都意味着 `true`。-->
<blog-post is-published></blog-post>

<!-- 即便 `false` 是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:is-published="false"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:is-published="post.isPublished"></blog-post>

<!-- 即便数组是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>

<!-- 即便对象是静态的，我们仍然需要 `v-bind` 来告诉 Vue -->
<!-- 这是一个 JavaScript 表达式而不是一个字符串。-->
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>

<!-- 用一个变量进行动态赋值。-->
<blog-post v-bind:author="post.author"></blog-post>
```
当然,从前面我们直到可以传递一个对象的所有属性,且不带`v-bind`指令:
```js
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```
```html
<blog-post v-bind="post"></blog-post>
```

#### 9.3.4. 单向数据流

所有的prop都使得其父子prop之间形成一个<font color=red>单向下行绑定</font>.每次父级组件发生更新时,子组件中所有的prop都会刷新.最好不要在子组件内部改变prop,这样会引起浏览器Vue警告.
这里有两种常见的试图改变一个 prop 的情形：
- 这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用.这种情况适合定义一个本地data属性.
```js
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```
- 这个 prop 以一种原始的值传入且需要进行转换。这种情况适合定义一个计算属性.
```html
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
### 9.4 自定义事件

#### 9.4.1. 事件名

不同于组件和prop,事件名不存在任何自动化的大小写转换.然而模板中`v-on`事件监听器会被自动转换为全小写(HTML大小写不敏感),所以如果事件名是camelCase,则会导致不能被监听到.
所以应该始终使用kebab-case事件名.例如:
```js
this.$emit('my-event')
```
#### 9.4.2. 自定义组件的v-model

一个组件上的`v-model`默认会利用名为 `value` 的 `prop` 和名为 `input` 的事件,但是想单选框,复选框等类型的输入控件可能会将 `value`特性用于不同的目的.通过`model`选项可以用来避免这样的冲突:
```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
现在在这个组件上使用 `v-model`的时候:
```html
<base-checkbox v-model="lovingVue"></base-checkbox>
```
这里的 `lovingVue` 的值将会传入这个名为 `checked` 的prop.同时当`<base-checkbox>` 触发一个 change 事件并附带一个新的值的时候,这个lovingVue的属性将会被更新.

#### 9.4.3. 将原生事件绑定到组件

Vue提供了一个 `$listeners`属性,它是一个对象,里面包含了作用在这个组件上的所有监听器:
```js
{
  focus : function (event) { ...}
  input : function (value) { ...}
}
```
```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // `Object.assign` 将所有的对象合并为一个新对象
      return Object.assign({},
        // 我们从父级添加所有的监听器
        this.$listeners,
        // 然后我们添加自定义监听器，
        // 或覆写一些监听器的行为
        {
          // 这里确保组件配合 `v-model` 的工作
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners"
      >
    </label>
  `
})
```

#### 9.4.4. 修饰符.sync

在某些情况下,我们可能需要对一个prop进行"双向绑定".这里我们使用 `update:myPropName`的模式触发事件来实现.例如在一个包含`title` prop的假设的组件中,我们可以用以下方法表达对其赋新值的意图:
```js
this.$emit('update:title', newTitle)
```
然后父组件可以监听这个事件并根据需要更新一个本地的数据属性:
```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```
为了方便起见,我们为这种模式提供一个缩写,即 `.sync`修饰符:
```html
<text-document v-bind:title.sync="doc.title"></text-document>
```

<font color=#D8301B size=2px>注意带有 .sync 修饰符的 v-bind 不能和表达式一起使用</font>。

当我们用一个对象同时设置多个prop的时候,也可以将这个`.sync`修饰符和`v-bind`配合使用:
```html
<text-document v-bind.sync="doc"></text-document>
```
这样会把`doc`对象中的每一个属性都作为一个独立的prop传进去,然后各自添加用于更新的`v-on`监视器.

### 9.5. 插槽

Vue实现了一套内容分发的API,将 `<slot>`元素作为承载分发内容的出口.
它允许你像这样合成组件:
```html
<navigation-link url="/profile">
  Your Profile
</navigation-link>
```
然后在`<navigation-link>`的模板中可能会写为:
```html
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot></slot>
</a>
```
当组件渲染的时候,`<slot></slot>`将会被替换为"Your Profile".插槽内可以包含任何模板代码,包括HTML:
```html
<navigation-link url="/profile">
  <!-- 添加一个 font awesome 图标 -->
  <span class="fa fa-user"></span>
  Your Profile
</navigation-link>
```
甚至其它的组件:
```html
<navigation-link url="/profile">
  <!-- 添加一个图标的组件 -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Your Profile
</navigation-link>
```
如果`<navigation-link>`没有包含一个`<slot>`元素,则该组件起始标签和结束标签之间的任何内容都会被抛弃.

#### 9.5.1. 编译作用域
当你想在一个插槽中使用数据时,例如:
```html
<navigation-link url="/profile">
 Logged in as {{user.name}}
 </navigation-link>
```
该插槽跟模板的其它地方一样可以访问相同的实例属性(作用域),而不能访问`<navigation-link>`的作用域.例如`url`是访问不到的:
```html
<navigation-link url="/profile">
  Clicking here will send you to: {{ url }}
  <!--
  这里的 `url` 会是 undefined，因为 "/profile" 是
  _传递给_ <navigation-link> 的而不是
  在 <navigation-link> 组件*内部*定义的。
  -->
</navigation-link>
```
记住:
父级模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

#### 9.5.2. 具名插槽

有时会我们需要多个插槽.对于这种情况,`<slot>`元素有一个特殊的特性:`name`.这个特性可以用来定义额外的插槽:
```html
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
```
没有显式定义的slot带有隐含的名字`default`.
提供内容的时候:
```html
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```
现在`<template>`中的所有内容将会被传入相应的插槽.没有被包裹的内容会被视为默认插槽的内容.

当然也可以将默认的内容放入`<template>`.只需要指定默认的名称.

注意 v-slot 只能添加在一个 `<template> `上 (只有一种例外情况).

#### 9.5.3. 作用域插槽

有时让插槽内容能够访问子组件中才有的数据是很有用的。例如，设想一个带有如下模板的 `<current-user>` 组件：
```html
<span>
  <slot>{{ user.lastName }}</slot>
</span>
```
我们想让它的后备内容显示用户的名，以取代正常情况下用户的姓，如下：
```html
<current-user>
  {{ user.firstName }}
</current-user>
```
然而上述代码不会正常工作，因为只有 `<current-user>` 组件可以访问到 `user` 而我们提供的内容是在父级渲染的。

为了让 `user` 在父级的插槽内容可用，我们可以将 `user` 作为 `<slot>` 元素的一个特性绑定上去：
```html
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```
绑定在 `<slot>` 元素上的特性被称为插槽 `prop`。现在在父级作用域中，我们可以给 `v-slot` 带一个值来定义我们提供的插槽 `prop` 的名字：
```html
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```
对于默认插槽,可以把`v-slot`直接用在组件上:
```html
<current-user v-slot:default="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```
或者:
```html
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```
如果出现多个插槽,需要使用完整的模板语法:
```html
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>

  <template v-slot:other="otherSlotProps">
    ...
  </template>
</current-user>
```

作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里:
```js
function (slotProps) {
  //插槽内容
}
```
这意味着 v-slot 的值实际上可以是任何能够作为函数定义中的参数的 JavaScript 表达式。
```html
<current-user v-slot="{ user }">
  {{ user.firstName }}
</current-user>
```
#### 9.5.4. 动态插槽名

动态指令参数也可以用在`v-slot`上,来定义动态的插槽名:
```html
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>
</base-layout>
```
#### 9.5.5. 具名插槽缩写

跟 v-on 和 v-bind 一样，v-slot 也有缩写，即把参数之前的所有内容 (v-slot:) 替换为字符 #。例如 v-slot:header 可以被重写为 #header：
```html
<base-layout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>
  <template #default>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </template>
  <template #footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

### 9.6. 动态组件 & 异步组件

