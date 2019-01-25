---
title: Vue 2.5入门以及一个 TodoList 的例子
date: 2019-01-23 10:56:45
tags:
  - Vue
  - TodoList
categories:
  - Vue
---

本文讲解几个知识点：

- Vue 实例的创建
- 挂载点、模板与实例之间的关系
- Vue 实例中的数据、事件和方法
- 属性绑定与双向数据绑定
- 计算属性与侦听器
- 三个常见的指令：v-if、v-show、v-for
- 组件拆分
- 组件与实例之间的关系

<!-- more -->

## 创建一个 Vue 实例

引入 Vue.js v2.5.22，添加一个 div

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue入门</title>
    <script src="./vue.js"></script>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

创建一个 Vue 实例，让实例通过 `el` 接管页面上某一个内容。这样这个实例就和页面上的某一个 DOM 做了绑定

```js
new Vue({
    el: '#root'
});
```

现在要给 `div` 标签添加内容，如果用原生的 JS 来写的话，需要手动去操作 DOM

```js
var dom = document.getElmentById('root');
dom.innerHTML = 'hello world';
```

但是通过 Vue 实例的 `data` 属性加上“插值表达”，可以这么写

```js
<div id="root">{{msg}}</div>

<script>
    new Vue({
        el: '#root',
        data: {
            msg: 'hello world'
        }
    });
</script>
```

这样就不需要去手动操作 DOM，这个工作由 Vue 来帮我们完成

## 挂载点、模板与实例之间的关系

### 挂载点

`<div id="root"></div>` 的 `id` 和 Vue 实例的 `el` 属性值对应上了，所以 `<div id="root"></div>` 就叫做 Vue 实例的“挂载点”

### 模板

挂载点里的内容就是模板，比如

```html
<div id="root">
    <h1>{{msg}}</h1>
</div>
```

其中 `<h1>{{msg}}<h1>` 就叫做“模板”。模板不光可以写在挂载点里，也可以写在 Vue 实例的 `template` 属性里，比如

```html
<div id="root"></div>

<script>
    new Vue({
        el: '#root',
        template: '<h1>{{msg}}</h1>',
        data: {
            msg: 'hello world'
        }
    });
</script>
```

### 实例

这就是个 Vue 实例

```js
new Vue({
    el: '#root',
    template: '<h1>{{msg}}</h1>',
    data: {
        msg: 'hello world'
    }
});
```

## Vue 实例中的数据、事件和方法

### 数据

Vue 属性的 `data` 中的内容，就是数据。数据加上插值表达式（`{{data}}`） 就可以把数据显示到模板里

比如

```html
<div id="root">{{msg}}</div>

<script>
    new Vue({
        el: '#root',
        data: {
            msg: 'Hello Vue.js'
        }
    });
</script>
```

显示数据，不光可以利用插值表达式，还可以利用这两个 Vue 指令

- `v-text`
- `v-html`

`v-text` 和 `v-html` 的区别在于，`v-text` 原样输出 `data` 属性的内容，`v-html` 会渲染成 HTML 再输出。看下面这个例子就明白了

```html
<div id="root">
    <div v-text="content"></div>
    <div v-html="content"></div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            content: '<h1>Hello Vue.js</h1>'
        }
    });
</script>
```

效果如图

{% asset_img Snipaste_2019-01-22_10-41-27 %}

### 事件

如何给某一个 div 标签绑定一个事件？利用一个模板指令：`v-on` 来实现

比如点击一个 div，把内容变换一下要怎么实现？

1. 使用 `v-on:click` 指令指定一个方法
2. 把方法定义写在 Vue 实例的 `methods` 属性里

```html
<div id="root">
    <div v-on:click="handleClick">{{content}}</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            content: 'Hello'
        },
        methods: {
            handleClick: function() {
                this.content = 'world'
            }
        }
    });
</script>
```

注意一下编程思路的转变

> 要改变 DOM 的内容，不要去修改 DOM，在 Vue 里应该是去修改数据，Vue 会帮我们更新 DOM。不是面向 DOM 编程，而是**面向数据编程**

通过 `this.xxx` 就可以修改 `data` 里的数据了


`v-on:` 可以简写成 `@`，所以 `v-on:click` 可以简写成 `@click`

## Vue 中的属性绑定和双向数据绑定

### 属性绑定

这样一个模板，有一个 `title` 属性，当鼠标放在这个 div 上面时，会显示 “this is hello world”

<div title="this is hello world">hello world</div>

如果要 `title` 属性的值是 Vue 实例里 `data` 里的值，应该使用 `v-bind` 指令指定 Vue 实例里 `data` 的值

```html
<div id="root">
    <div v-bind:title="title">hello world</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            title: 'this is hello world'
        }
    });
</script>
```

`v-bind:` 可以简写成 `:`，比如

```html
<div id="root">
    <div :title="title">hello world</div>
</div>
```

### 双向数据绑定

属性绑定的进一步就是双向数据绑定 

一个 `div` 和一个 `input` 标签，都显示 content 的内容

```html
<div id="root">
    <div>{{content}}</div>
    <input :value="content" />
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            content: 'this is content'
        }
    });
</script>
```

但是，改变 `input` 的内容，`div` 的内容没有跟着改变。这就是数据决定了页面的显示，但是页面无法决定数据的内容，也就是单向绑定

如果要实现双向绑定：

> 数据决定页面的显示，页面也能决定数据的内容

应该使用 `v-model` 模板指令

```html
<div id="root">
    <div>{{content}}</div>
    <input v-model="content" />
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            content: 'this is content'
        }
    });
</script>
```

## Vue 中的计算属性和侦听器

### 计算属性

一个输入姓和名，显示全名的例子

```html
<div id="root">
    姓：<input v-model="firstName"/>
    名：<input v-model="lastName"/>
    全名：{{firstName}} {{lastName}}
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            firstName: '',
            lastName: ''
        }
    });
</script>
```

全名用两个变量组成不够好看，可以用另外一个变量 `fullName` 来表示。`fullName` = `firstName` + `lastName`。在 Vue 中，可以利用计算属性（一个属性由其他属性计算而来）来实现

```html
<div id="root">
    姓：<input v-model="firstName"/>
    名：<input v-model="lastName"/>
    全名：{{fullName}}
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            firstName: '',
            lastName: ''
        },
        computed: {
            fullName: function() {
                return this.firstName + ' ' + this.lastName;
            }
        }
    });
</script>
```

不过，注意一下坑。只有 `firstName` 和 `lastName` 变化了之后，`fullName` 才会重新计算。比如，`firstName` 和 `lastName` 都有一个初始值 `a` 和 `b`


```html
<div id="root">
    姓：<input v-model="firstName"/>
    名：<input v-model="lastName"/>
    全名：{{fullName}}
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            firstName: 'a',
            lastName: 'b'
        },
        computed: {
            fullName: function() {
                return this.firstName + ' ' + this.lastName;
            }
        }
    });
</script>
```

当页面显示出来的时候，`fullName` 是为空的，不会是 `a b`。这一点一定要注意

### 侦听器

要监听某一个数据发生了变化，可以使用侦听器 `watch`。比如 `fullName` 变化了，那么 `count` 就加 1

```html
<div id="root">
    姓：<input v-model="firstName"/>
    名：<input v-model="lastName"/>
    全名：{{fullName}}
    count：{{count}}
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            firstName: '',
            lastName: '',
            count: 0
        },
        computed: {
            fullName: function() {
                return this.firstName + ' ' + this.lastName;
            }
        },
        watch: {
            fullName: function() {
                this.count++;
            }
        }
    });
</script>
```

## 三个常见的指令：v-if、v-show、v-for

### v-if

控制模板的是否从 DOM 树中删除，也就是

> 控制 DOM 的存在与否

值为 `true` 就显示，`false` 就删除

```html
<div id="root">
    <div v-if="show">hell world</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            show: true
        }
    });
</script>
```

可以看到 `<div>hell world</div>` 存在与 DOM 树中

{% asset_img Snipaste_2019-01-22_14-56-18.png %}

```html
<div id="root">
    <div v-if="show">hell world</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            show: false
        }
    });
</script>
```

可以看到 `<div>hell world</div>` 从 DOM 树中中删除了，变成了一行注释 `<!-- -->`

{% asset_img Snipaste_2019-01-22_14-57-33.png %}

### v-show

`v-show` 与 `v-if` 不同，它不会把模板从 DOM 树中删除，而是改变模板的属性，也就是

> 控制 DOM 的显示与否

```
<div id="root">
    <div v-show="show">hell world</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            show: true
        }
    });
</script>
```

模板正常显示

{% asset_img Snipaste_2019-01-22_15-02-58.png %}

```html
<div id="root">
    <div v-show="show">hell world</div>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            show: false
        }
    });
</script>
```

模板增加一个属性 `style="display: none;"`

{% asset_img Snipaste_2019-01-22_15-03-30.png %}

### v-for

显示列表用的，把数据循环展示出来

```html
<div id="root">
    <ul>
        <li v-for="item of list">{{item}}</li>
    </ul>
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            list: [1, 2, 3]
        }
    });
</script>
```

## 一个 TodoList 的例子

在 `input` 中填写内容，点击 `button`，就能把内容以列表的形式展示出来

```html
<div id="root">
    <input v-model="todo"/>
    <button @click="handleSubmit">提交</button>
    <ul>
        <li v-for="item of todoList">{{item}}</li>
    </ul>    
</div>

<script>
    new Vue({
        el: '#root',
        data: {
            todo: '',
            todoList: []
        },
        methods: {
            handleSubmit: function() {
                this.todoList.push(this.todo);
                this.todo = '';
            }
        }
    });
</script>
```

## 对 TodoList 进行组件拆分

所谓“组件”，就是页面的一部分。如果一个项目很大的话，把项目分成一个个小的部分进行开发，就会方便很多了

问题来了：

- 如何定义组件？
- 组件与组件之间如何做通信？

#### 如何定义组件？

现在把

```html
<li v-for="item of todoList">{{item}}</li>
```

抽取出来作为一个组件

组件有两种：

1. 全局组件
2. 局部组件

这就是定义了一个全局组件

```js
Vue.component('todo-item', {
    template: '<li>item</li>'
});
```

这是定义一个局部组件

```js
new Vue({
    el: '#root',
    components: {
        'todo-item': {
            template: '<li>item</li>'
        }
    }
});
</script>
```

### 组件与组件之间如何做通信？

可以直接套用 `v-for`，点击 `button` 就可以展示列表

```html
<div id="root">
    <input v-model="todo"/>
    <button @click="handleSubmit">提交</button>
    <ul>
        <todo-item v-for="item of todoList"></todo-item>
    </ul>    
</div>

<script>
    Vue.component('todo-item', {
        template: '<li>item</li>'
    });

    new Vue({
        el: '#root',
        data: {
            todo: '',
            todoList: []
        },
        methods: {
            handleSubmit: function() {
                this.todoList.push(this.todo);
                this.todo = '';
            }
        }
    });
</script>
```

{% asset_img Snipaste_2019-01-22_16-04-04.png %}

要像之前那样，输入什么就显示什么，需要给组件加一个属性 `props`，表示输入的内容，然后再使用插值表达式显示内容

因为 Vue 规定，父组件向子组件传值，需要使用**属性**的形式

```js
Vue.component('todo-item', {
    props: ['todo'],
    template: '<li>{{todo}}</li>'
});
```

然后模板改一下，使用属性绑定

```html
<ul>
    <todo-item v-for="item of todoList" :todo="item"></todo-item>
</ul> 
```

这样就没问题了

## 组件和实例的关系

每一个组件都是一个 Vue 的实例。比如上面的全局组件 `todo-item`，也可以定义 Vue 实例一样增加 `methods` 属性

```js
Vue.component('todo-item', {
    props: ['todo'],
    template: '<li @click="handleClick">{{todo}}</li>',
    methods: {
        handleClick: function() {
            alert(this.todo);
        }
    }
});
```

那为什么这个 Vue 实例没有 `template` 呢？

```js
new Vue({
    el: '#root',
    data: {
        todo: '',
        todoList: []
    },
    methods: {
        handleSubmit: function() {
            this.todoList.push(this.todo);
            this.todo = '';
        }
    }
});
```

如果 Vue 实例找不到 `template` 的话，会自己去找 `el` 指定的挂载点下的模板作为 `template` 的定义。所以，也可以把这个 Vue 实例叫做父组件，里面有子组件 `todo-item`

## 实现 TodoList 的删除功能

希望点击 TodoList 时，能被删除掉。也就是需要子组件告诉父组件，要把 `todoList` 的其中一个元素删除掉。在 Vue 中，子组件与父组件之间进行通信，需要通过“发布-订阅”模式来实现

传入 `item` 的索引 `index`

```html
<ul>
    <todo-item v-for="(item, index) of todoList" :todo="item" :index="index"></todo-item>
</ul> 

在子组件中发布一个事件

```js
Vue.component('todo-item', {
    props: ['todo', 'index'],
    template: '<li @click="handleClick">{{todo}}</li>',
    methods: {
        handleClick: function() {
            this.$emit('delete', this.index);  // 发布一个 delete 事件
        }
    }
});
```

使用 `v-on` 指令在父组件中监听这个 `delete` 事件

```html
<todo-item v-for="(item, index) of todoList" :todo="item" :index="index" @delete="deleteTodo"></todo-item>
```

```js
<script>
    new Vue({
        el: '#root',
        data: {
            todo: '',
            todoList: []
        },
        methods: {
            handleSubmit: function() {
                this.todoList.push(this.todo);
                this.todo = '';
            },
            deleteTodo: function(index) {
                this.todoList.splice(index, 1);
            }
        }
    });
</script>
```