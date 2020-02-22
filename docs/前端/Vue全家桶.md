---
title: Vue全家桶
---

## Vue

### 基本语法



### 组件化



### 复用



### 原理

#### MVVM

MVVM（Model View ViewModel）

![MVVM](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/image-20200221205703849.png)

- View，即前端的DOM
- Model，数据模型
- ViewModel，视图模型层。通过Data Bindings（数据绑定），将Model的改变同步到View。通过DOM Listeners（DOM监听），监听DOM的事件（点击、touch），并通过回调修改Model数据。

#### Vue版本

在 NPM 包的 `dist/` 目录下多个不同的 Vue.js 构建版本。

Vue包含了**运行时（runtime）**和**编译器（compiler）**两部分：编译器用来将模版（如template、el）编译为render函数。运行时则是用来创建Vue实例、渲染和处理虚拟DOM等。**完整版（Full）**就是runtime+compiler。另外，根据不同的模块化方案，提供了不同的版本：如UMD、CommonJS、ES Module。

如果在客户端编译模版，则需要使用完整版（包含编译器）；而如果通过打包器（如webpack）构建项目，则应该在打包时使用vue-loader来将模版预编译成render函数，然后通过运行版运行。运行版的体积更小，运行速度更快。

#### 生命周期

![生命周期](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/image-20200221211645286.png)

```js
var vm = new Vue({
  el: "#app",
  data: {
    message: "hello world"
  },
  beforeCreate() {
    showData("beforeCreate", this);
  },
  created() {
    showData("created", this);
  },
  beforeMount() {
    showData("beforeMount", this);
  },
  mounted() {
    showData("mounted", this);
  },
  beforeUpdate() {
    showData("beforeUpdate", this);
  },
  updated() {
    showData("updated", this);
  },
  beforeDestroy() {
    showData("beforeDestroy", this);
  },
  destroyed() {
    showData("destroyed", this);
  }
});

function showData(process, obj) {
  console.log(process);
  console.log("data数据对象：" + obj.message);
  console.log("$el挂载对象：" + obj.$el);
  console.log(
    "$el挂载对象的内容：" +
      (obj.$el === undefined ? "undefined" : obj.$el.innerHTML)
  );
  console.log("实际的DOM Element：" + document.getElementById("app").innerHTML);
  console.log("----------");
}

vm.message = "update data";
setTimeout(() => {
  vm.$destroy();
}, 1000);

// 输出结果：
beforeCreate
data数据对象：undefined
$el挂载对象：undefined
$el挂载对象的内容：undefined
实际的DOM Element：{{message}}
----------
created
data数据对象：hello world
$el挂载对象：undefined
$el挂载对象的内容：undefined
实际的DOM Element：{{message}}
----------
beforeMount
data数据对象：hello world
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：{{message}}
实际的DOM Element：{{message}}
----------
mounted
data数据对象：hello world
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：hello world
实际的DOM Element：hello world
----------
beforeUpdate
data数据对象：update data
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：hello world
实际的DOM Element：hello world
----------
updated
data数据对象：update data
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：update data
实际的DOM Element：update data
----------
beforeDestroy
data数据对象：update data
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：update data
实际的DOM Element：update data
----------
destroyed
data数据对象：update data
$el挂载对象：[object HTMLDivElement]
$el挂载对象的内容：update data
实际的DOM Element：update data
----------
```

#### 响应式

响应式，即检测到数据对象变化后，将自动同步/渲染到DOM树上。

> **Vue是如何检测Model变化的**

Vue遍历data配置项中的所有属性，通过`Object.defineProperty`把这些属性全部转为 getter/setter。当Model发生修改时，会自动调用setter方法。

每个组件实例都对应一个 watcher 实例，它会在组件渲染的过程中把“接触”过的数据属性记录为依赖。之后当依赖项的 setter 触发时，会通知 watcher，从而使它关联的组件重新渲染。

![响应式流程](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/image-20200221215216257.png)

> **响应式简单代码实现**

![](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/v-1.jpeg)

```js
// 发布者（依赖）
class Dependency {
  constructor() {
    this.subscribers = []; // 用来保存该发布者管理的所有订阅者的队列
  }
  addSub(watcher) {
    // 添加订阅者
    this.subscribers.push(watcher);
  }
  notify() {
    // 通知订阅者
    this.subscribers.forEach(sub => {
      sub.update();
    });
  }
}

// 订阅者
class Watcher {
  constructor(componentName, data) {
    this.name = componentName; // 组件名
    this.data = data; // 组件中绑定的数据
  }

  update() {
    // 监听到通知后的处理方法，让组件重新渲染页面
    console.log(`${this.name}中的${this.data}更新了`);
  }
}

// Vue中的$data
const obj = {
  message: "Hello World",
  name: "zhangsan"
};

// 遍历所有的数据模型
Object.keys(obj).forEach(prop => {
  let value = obj[prop];

  // 每个数据模型对应一个Dep依赖，依赖管理着与该数据模型绑定的所有组件实例
  const dep = new Dependency();
  const w1 = new Watcher("组件A", prop);
  const w2 = new Watcher("组件B", prop);
  const w3 = new Watcher("组件C", prop);
  dep.addSub(w1);
  dep.addSub(w2);
  dep.addSub(w3);

  // 将Vue.$data中的所有数据模型转为getter/setter
  Object.defineProperty(obj, prop, {
    set(newValue) {
      value = newValue;
      dep.notify(); // 当数据模型更新时，通知watcher/组件
    },
    get() {
      return value;
    }
  });
});
```

> **注意事项**

- 因为原生JS没有方法可以检测对象属性的添加或删除，所以Vue也就无法检测对象属性的添加或删除，即只有在data配置项中存在的Model才能被转为响应式的。
- 只有在创建Vue实例时通过data配置项添加的Model是响应式的。直接通过vm.xxx='val'添加的属性不是响应式的。
- Vue提供了`Vue.set(object, propertyName, value)`/`vm.$set`方法来向**嵌套对象**添加响应式属性。即可以对data配置项中已经声明了的响应式Mode添加新属性。

#### 异步更新队列

- Vue 在更新 DOM 时是**异步**执行的，即当Model变化时，Vue将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，会在缓冲时自动去重，避免不必要的计算和DOM刷新。
- 然后，在下一个的事件循环**“tick”**中，Vue 刷新队列并执行实际 (已去重的) 工作。
- Vue 在内部对异步队列尝试使用原生的 Promise.then、MutationObserver 和 setImmediate，如果执行环境不支持，则会采用 setTimeout(fn, 0) 代替。

因为Vue更新DOM是异步的，我们不知道何时DOM刷新完成，有时我们想在DOM刷新完成后再执行一个操作。可以通过`Vue.nextTick(callback)`/`vm.$nextTick`来设置一个DOM刷新完成后的回调函数。

```js
methods: {
  updateMessage: function () {
    this.message = '已更新'
    console.log(this.$el.textContent) // => '未更新'
    this.$nextTick(function () {
      console.log(this.$el.textContent) // => '已更新'
    })
  }
}
```

#### 渲染机制

![渲染机制](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/image-20200221214857385.png)

## Vue CLI





## Vue Router





## Vuex

### 概述
Vuex是专为Vue.js 应用程序开发的**状态管理**模式。它采用**集中式**存储管理应用的所有组件的状态。简单来说，状态管理就是让多个组件共享同一个数据。

![vuex](Vue%E5%85%A8%E5%AE%B6%E6%A1%B6.assets/vuex.png)

说明：

- Vuex 的状态存储是响应式的。当state发生变化，自动同步到组件的view上。
- 当view触发action事件来修改state时，要通过commit mutation的方式来提交修改。

需要用到状态管理的场景：

- 用户登陆状态token、用户名、头像、地理位置信息
- 商品收藏、购物车

### 快速入门

1. 安装： npm install vuex 
2. 创建Vuex.Store实例

```javascript
// src\store\index.js

import Vuex from "vuex"
import Vue from "vue"

// 1.安装插件
Vue.use(Vuex)

// 2.创建Store对象
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {// mutations内的方法，默认自动传入state对象
      state.count++
    }
  },
  actions: {},
  getters: {},
  modules: {}
})

export default store
```

3. 将Vuex.Store实例注入到Vue实例

```javascript
// src\main.js
import Vue from "vue"
import App from "./App.vue"
import store from "./store"

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
  store // 将Vuex.Store实例暴露给Vue示例的store选项，从而将Vuex.Store实例注入到Vue实例中，在Vue实例中可以通过this.$store访问到Vuex.Store实例
}).$mount("#app")
```

4. 使用Vuex

```vue
// src\App.vue
<template>
  <div id="app">
    <h2>{{$store.state.count}}</h2>
    <button @click="increase">+</button>
  </div>
</template>

<script>
export default {
  methods: {
    increase() {
      this.$store.commit("increment");//不要直接修改$store.state.count，而应该通过commit mutations来提交修改。这样Vuex可以记录状态的修改过程。
    }
  }
};
</script>
```

### 核心概念

#### State

> **单一状态树**

每个应用中只创建一个Vuex.Store对象来保存所有的状态。

> **在 Vue 组件中获得 Vuex 状态**

在组件中通过`this.$store.state.count`访问到state。

为了方便使用，通过将`this.$store.state.count`包裹为组件中的computed计算属性：

```javascript
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

> **绑定函数mapState**

vuex提供了mapState函数，可以方便地从Vuex实例中取出state：

```vue
<template>
  <div id="app">
    <h2>{{count}}</h2>
    <h2>{{name}}</h2>
    <h2>{{age}}</h2>
  </div>
</template>

<script>
import { mapState } from "vuex"; // 引入vuex的mapState函数
export default {
  name: "App",
  data() {
    return {
      namePrefix: "JN-"
    };
  },
  computed: {
    ...mapState({// mapState函数返回对象类型，这里使用对象展开运算符
      count: "count",// 使用字符串，直接取出对应的state
      name(state) {// 通过函数取出state，并且可以使用this
        return this.namePrefix + state.name;
      },
      age: state => state.age// 通过箭头函数取出state
    }),
    otherComputed() {}//组件中的其他计算属性
  },
  // computed: {
  // 当computed计算属性名与state名字相同时，可以给mapState传入一个字符串数组
  //  ...mapState(["name", "count", "age"])
  // }
};
</script>
```

#### Getters

getter可以看作是state的computed计算属性。getter的返回值会被缓存，当state改变时，getter会自动重新计算。

> **示例**

getter函数可以接收两个参数：(state, getters)。state即是Vuex.Store实例中的state状态对象，getters即使Vuex.Store实例中的getters计算状态。

在Vue组件中可以通过`this.$store.getters.xxx`访问getter计算状态。

```javascript
const store = new Vuex.Store({
  state: {
    students: [
      { id: 1, name: "zhangsan", age: 25 },
      { id: 2, name: "lisi", age: 19 },
      { id: 3, name: "wangwu", age: 23 }
    ]
  },
  getters: {
    // 计算年龄大于20的学生
    ageOver20: state => state.students.filter(stu => stu.age > 20),
    // 计算年龄大于20的学生个数
    ageOver20Count: (state, getters) => getters.ageOver20.length,
    // 获取指定id的学生
    getById(state) {
      return function(id) {//这个getter返回的是一个函数，调用时需要传入参数id
        return state.students.find(stu => stu.id === id)
      }
    }
  },
  mutations: {},
  actions: {},
  modules: {}
})
```

```vue
<template>
  <div id="app">
    <h2>{{ageOver20}}</h2>
    <h2>{{$store.getters.getById(2)}}</h2>
  </div>
</template>

<script>
export default {
  computed: {
    ageOver20() {
      return this.$store.getters.ageOver20;
    }
  }
};
</script>
```
> **绑定函数mapGetters**

```javascript
import { mapGetters } from "vuex";
export default {
  computed: {
    // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters(["ageOver20", "ageOver20Count", "getById"])
    // 将ageOver20Count重命名为stuCount计算属性
    // ...mapGetters({stuCount: "ageOver20Count"})
  }
};
```

#### Mutation

mutation是用来改变state的方式，类似于**事件**。它包含事件类型（type）和回调函数（handler）。

mutation的回调函数接收两个参数：(state[, payload])，state表示Vuex实例中的状态，payload是负载数据。

在调用mutation时，要通过`store.commit(type[, payload])`方法。

> 示例

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      state.count++
    },
    incrementN (state, n) {
    	state.count += n
    }
  }
})

// 调用mutation
this.$store.commit('increment')
this.$store.commit('incrementN',10)
```

> **Mutation要遵循响应式规则**

正如Vue中的响应式data，Vuex中的state状态也是响应式的（即将state的变化同步到view）。虽然mutation可以修改state，但Vuex只能监听state对象属性的改变，不能对新增或删除属性做出响应。

解决办法：

- 方法1：预先在state对象中初始化所有需要的属性；

- 方法2：通过Vue.set(state.obj, 'newProp', val)来添加新属性；
- 方法3：用新对象替换旧的state对象：state.obj = newObj；

```javascript
const store = new Vuex.Store({
  state: {
    cat: { name: "小白" } // 该state对象默认没有gender属性
  },
  getters: {},
  mutations: {
    addGenderProp(state, gender) {
      //state.cat["gender"] = gender // 直接新增属性，无法触发响应式
      Vue.set(state.cat, "gender", gender) // Vue.set
      //state.cat = { ...state.cat, gender: gender } // 以新对象替换老对象
    }
  },
  actions: {},
  modules: {}
})
```

> **Mutation 必须是同步函数**

mutation内禁止进行异步操作。mutation中进行异步操作，会导致Vuex无法**追踪状态的改变**。

如果要执行异步操作，应该使用action。

> **代码规范：建议使用常量来替代Mutation事件类型**

在`store.commit(type, playload)`提交mutation时，type是字符串，容易写错。建议将事件类型定义为常量来使用。

```
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'

// store\index.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})

// App.vue
import { SOME_MUTATION } from './mutation-types'

export default {
  methods: {
    update() {
      this.$store.commit(SOME_MUTATION);
    }
  }
};
```

> **绑定函数mapMutations**

```
import { mapMutations } from 'vuex'

export default {
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

#### Action

action也是用来修改state的，但**action不是直接修改state**，而是通过commit mutation。

action相比mutation，它**支持异步操作**。

action函数接收参数**context**，用**context.commit**来提交state更新。

通过`store.dispatch(type[,payload])`来分发action。

> **示例**

```javascript
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      setTimeout(() => {
      	context.commit('increment')
    	}, 1000)
      // 用context.state获取state
      // 用context.getters获取getters
    },
    increment2 ({commit, state, getters}) {// 使用参数解构，直接从context中抽取commit、state和getters
      commit('increment')
    },
    increment3 ({commit},n){} 
  }
})

// 分发action
this.$store.dispatch('increment');
this.$store.dispatch('increment',10);
```

> **多个Action顺序执行**

store.dispatch可以处理action返回的Promise，并且store.dispatch仍旧返回Promise。

```javascript
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {// 让action返回一个Promise对象
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  },
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {// 多个action顺序执行
      commit('someOtherMutation')
    })
  }
}
```

```javascript
// 假设 getData() 和 getOtherData() 返回的是 Promise
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())// 等待getData获取数据，然后提交mutation
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())// 等待getOtherData获取数据，然后提交mutation
  }
}
```

> **绑定函数mapActions**

```js
import { mapActions } from 'vuex'

export default {
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
      'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

#### Module

由于Vuex.Store是**单一状态树**，当state太多时，为了方便管理，Vuex支持将Store分割成模块，且模块可以嵌套子模块。每个模块拥有自己的state、getter、mutation、action。

> **示例**

```javascript
const moduleA = {
  state: {
    cat: { name: "大花", age: 8 }
  },
  mutations: { ... },
  actions: { ... },
  getters: { ... },
	modules:{//子模块}
}

const moduleB = {
  state: {
		student: { name: "张三", age: 18 }
  },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

> **模块下的State**

模块中的state成为**局部state**。通过`this.$store.state.moduleName.obj`来调用。全局state仍通过`this.$store.state.obj`来调用。

```javascript
// 声明state
const mouduleA = {
  state: {name: "大花"}//局部state
}

const mouduleB = {
  state: {name: "张三"}//局部state
}

const store = new Vuex.Store({
  modules: {//注册模块
    a: mouduleA,
    b: mouduleB
  },
  state: state: {name: "Niki"}//全局state
})

// 在组件中调用state
this.$store.state.a.name //调用a模块中的state，返回"大花"
this.$store.state.b.name //调用b模块中的state，返回"张三"
this.$store.state.name //调用全局state，返回"Niki"
```

> **模块下的Getter**

1. 模块化后，getter函数的可以接收4个入参：
   - state：局部state
   - getters：局部getters
   - rootState：全局state
   - rootGetters：全局getters

```javascript
const mouduleA = {
  state: { name: "大花" },
  getters: {
    getName(state, getters, rootState, rootGetters) {
      state.name // 返回"大花"
      rootState.name // 返回"Niki"
    }
  }
}

const store = new Vuex.Store({
  modules: {
    a: mouduleA
  },
  state: { name: "Niki" }
})
```

2. 子模块中的getter**默认注册在全局命名空间**，可以在模块中配置`namespaced: true`来使其成为**带命名空间的模块**。

```javascript
// getter默认注册在全局命名空间
// 下面的示例中，模块a和模块b都注册了getName同名getter
const mouduleA = {
  state: { name: "大花" },
  getters: {
    getName(state, getters, rootState, rootGetters) {
      return state.name
    }
  }
}

const mouduleB = {
  state: { name: "张三" },
  getters: {
    getName(state, getters, rootState, rootGetters) {
      return state.name
    }
  }
}

const store = new Vuex.Store({
  modules: {
    a: mouduleA,
    b: mouduleB
  }
})

// 在组件中调用getter
this.$store.getters.getName //返回'大花'，并且控制台报错：duplicate getter key: getName
```

```javascript
// 使用`namespaced: true`使getter自动添加命名空间，从而防止注册重名getter
const mouduleA = {
  namespaced: true,
  state: { name: "大花" },
  getters: {
    getName(state, getters, rootState, rootGetters) {
      return state.name
    }
  },
  modules: {// a模块下的子模块
    a1: {
      // 没有声明namespaced: true的子模块将继承父模块的命名空间
      getters: {
        profile() {return "Adam"} // -> getters['a/profile']
      }
    },
    a2: {
      namespaced: true,
      getters: {
        profile() {return "Bob"} // -> getters['a/a2/profile']
      }
    }
  }
}

const mouduleB = {
  namespaced: true,
  state: { name: "张三" },
  getters: {
    getName(state, getters, rootState, rootGetters) {
      return state.name
    }
  }
}

const store = new Vuex.Store({
  modules: {
    a: mouduleA,
    b: mouduleB
  }
})

// 在组件中调用
this.$store.getters['a/getName']//输出"大花"
this.$store.getters['b/getName']//输出"张三"
this.$store.getters['a/profile']//输出"Adam"
this.$store.getters['a/a2/profile']//输出"Bob"
```

> **模块下的Mutations**

1. 模块中的mutation函数的入参state是局部state。
2. 模块中的mutation默认注册在**全局命名空间**，使用`namespaced: true`可以使mutation也变为带有命名空间

```javascript
const mouduleA = {
  namespaced: true,
  state: { name: "大花" },
  mutations: {
    updateName: (state, name) => (state.name = name)
  }
}

const mouduleB = {
  namespaced: true,
  state: { name: "张三" },
  mutations: {
    updateName: (state, name) => (state.name = name)
  }
}

const store = new Vuex.Store({
  modules: {
    a: mouduleA,
    b: mouduleB
  }
})

//在组件中调用
this.$store.commit("a/updateName", "Niki");
this.$store.commit("b/updateName", "Bon");
```

> **模块下的Actions**

1. 模块中的action函数的入参context的结构发生变化：

   - context.state：局部state
   - context.getters：局部getter
   - context.commit：局部commit
   - context.dispatch：局部dispatch
   - context.rootState：全局state
   - context.rootGetters：全局getter

```javascript
actions: {
	updateNameAsync({state,getters,commit,dispatch,rootState,rootGetters}) {...}
}
```

2. 模块中的action默认注册在**全局命名空间**，使用`namespaced: true`可以使action也变为带有命名空间
3. 模块中的action函数内，`commit` 和`dispatch` 被局部化了，即这两个方法只能触发该模块内的mutation和其他action。如果想要在action内触发全局的mutation、action，则将`{ root: true }` 作为第三参数传给 `dispatch` 或 `commit` 即可。

```javascript
actions: {
	someAction ({ dispatch, commit, getters, rootGetters }) {
		getters.someGetter // -> 'foo/someGetter'
		rootGetters.someGetter // -> 'someGetter'

		dispatch('someOtherAction') // -> 'foo/someOtherAction'
		dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

		commit('someMutation') // -> 'foo/someMutation'
		commit('someMutation', null, { root: true }) // -> 'someMutation'
	}
}
```

> **模块下的绑定函数**

```javascript
//基本使用
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  })
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}

// 将模块的空间名称字符串作为第一个参数传递给绑定函数，这样所有绑定都会自动将该模块作为上下文。
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}

// 可以通过使用 createNamespacedHelpers 创建基于某个命名空间辅助函数。它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数
import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    ...mapState({// 在 `some/nested/module` 中查找
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    ...mapActions([// 在 `some/nested/module` 中查找
      'foo',
      'bar'
    ])
  }
}
```

## Axios

### 快速入门

1. 安装：npm install axios
2. 引入axios：import axios from "axios"

3. 使用axios

```javascript
import axios from "axios"
axios({
  method: "get",
  url: "https://httpbin.org/get"
}).then(resp => console.log(resp))
```

### 全局axios

以下axios方法在调用后会**立即**发送请求：

- axios(config)
- axios.get(url[, config])
- axios.post(url[, data[, config]])

axios发送请求后返回**Promise**。

#### GET请求

```javascript
axios.get('/user', {
    params: {ID: 12345}
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .then(function () {
    // always executed
  });

//使用async/await
async function getUser() {
  try {
    const response = await axios.get('/user?ID=12345');
    console.log(response);
  } catch (error) {
    console.error(error);
  }
}
```

#### POST请求

```javascript
axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

#### 并发请求

- axios.all(iterable)：并发请求，传入iterable形式的多个axios实例，then的回调中收到的是数组形式的response
- axios.spread(callback)：展开并发请求的多个response

```javascript
function getUserAccount() {
  return axios.get('/user/12345');
}

function getUserPermissions() {
  return axios.get('/user/12345/permissions');
}

axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // Both requests are now complete
  }));
```

### 实例axios

> **创建axios实例**

- axios.create([config])：只创建axios实例，不会立即发送请求

```javascript
const instance = axios.create({
  baseURL: 'https://some-domain.com/api/',
  timeout: 1000,
  headers: {'X-Custom-Header': 'foobar'}
});
```

> **实例请求发送方法**

- instance(config)
- instance#get(url[, config])

- instance#post(url[, data[, config]])

### 请求配置

> **请求配置项**

```javascript
{
   // `url` 是用于请求的服务器 URL
  url: '/user',

  // `method` 是创建请求时使用的方法，如果不配置则默认使用GET请求
  method: 'get', // default

  // `baseURL` 将自动加在 `url` 前面，除非 `url` 是一个绝对 URL。
  baseURL: 'https://some-domain.com/api/',

  // `headers` 是即将被发送的自定义请求头
  headers: {'X-Requested-With': 'XMLHttpRequest'},

  // `params` 是即将与请求一起发送的 URL 参数
  // 必须是一个无格式对象(plain object)或 URLSearchParams 对象
  params: {
    ID: 12345
  },

  // `data` 是作为请求主体被发送的数据
  // 只适用于这些请求方法 'PUT', 'POST', 和 'PATCH'
  // 在没有设置 `transformRequest` 时，必须是以下类型之一：
  // - string, plain object, ArrayBuffer, ArrayBufferView, URLSearchParams
  // - 浏览器专属：FormData, File, Blob
  // - Node 专属： Stream
  data: {
    firstName: 'Fred'
  },

  // `timeout` 指定请求超时的毫秒数(0 表示无超时时间)
  // 如果请求话费了超过 `timeout` 的时间，请求将被中断
  timeout: 1000,

   // `responseType` 表示服务器响应的数据类型，可以是 'arraybuffer', 'blob', 'document', 'json', 'text', 'stream'
  responseType: 'json', // default

  // `responseEncoding` indicates encoding to use for decoding responses
  // Note: Ignored for `responseType` of 'stream' or client-side requests
  responseEncoding: 'utf8', // default

   // `onUploadProgress` 允许为上传处理进度事件
  onUploadProgress: function (progressEvent) {
    // Do whatever you want with the native progress event
  },

  // `onDownloadProgress` 允许为下载处理进度事件
  onDownloadProgress: function (progressEvent) {
    // 对原生进度事件的处理
  },

   // `maxContentLength` 定义允许的响应内容的最大尺寸
  maxContentLength: 2000,
}
```

> **配置全局axios默认值**

```javascript
axios.defaults.baseURL = 'https://api.example.com';
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

> **配置axios实例的默认值**

```javascript
const instance = axios.create({
  baseURL: 'https://api.example.com'
});
```

> **配置优先级**

请求方法的config > 实例config > 全局config

### 响应结构

```javascript
{
  // `data` 由服务器提供的响应
  data: {},

  // `status` 来自服务器响应的 HTTP 状态码
  status: 200,

  // `statusText` 来自服务器响应的 HTTP 状态信息
  statusText: 'OK',

  // `headers` 服务器响应的头
  headers: {},

  // `config` 是axios请求时的配置信息
  config: {},

  // `request` 表示产生该响应的请求对象，在浏览器环境下是一个XMLHttpRequest实例
  request: {}
}
```

### 拦截器

拦截器在请求发送前和响应被 `then` 或 `catch` 处理前拦截它们。

```javascript
// 添加请求拦截器
axios.interceptors.request.use(function (config) {
    // 在发送请求之前修改config
    return config;
  }, function (error) {
  	// 处理请求错误
    return Promise.reject(error);
  });

// 添加响应拦截器
axios.interceptors.response.use(function (response) {
    // 对响应进行处理
    return response;
  }, function (error) {
    // 处理响应错误
    return Promise.reject(error);
  });
```

### 封装

```javascript
import axios from "axios"
const instance = axios.create({
  baseURL: "http://123.207.32.32:8000/",
  timeout: 3000
})
export const request = config => instance(config)

// 调用
import { request } from "./network/request";
request({ url: "home/multidata" })
  .then(res => (this.a = res.data))
  .catch(err => console.log(err));
```

