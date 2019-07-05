# Vuex


## Vuex是什么

Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。


## 什么是“状态管理模式”？
让我们从一个简单的 Vue 计数应用开始

```

new Vue({
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})

```
state：驱动应用的数据源
view: state映射的视图
actions：view层面影响state的操作，如用户输入、点击等。


![MacDown logo](https://vuex.vuejs.org/flow.png)

## 什么条件下使用
但是，当我们的应用遇到多个组件共享状态时，单向数据流的简洁性很容易被破坏

多个视图依赖于同一状态(多个组件共用一个状态做驱动机制时，如用户登陆加载相应信息)。

来自不同视图的行为需要变更同一状态(详情页面购买商品后，可在我的订单、购物车、结算等多个页面查看)

非单页大型应用不推荐。多页数据量较大推荐使用。

案例：见mis-manager /#/vue-demo


![MacDown logo](https://vuex.vuejs.org/vuex.png)



![MacDown logo](https://github.com/bailicangdu/pxq/raw/master/screenshot/simple_redux.jpg)




![MacDown logo](https://github.com/bailicangdu/pxq/raw/master/screenshot/all_redux.png)





## 特性
一个全局单例模式管理，沙盒模式。通过暴露相关api，可对共享状态进行保护性操作(闭包特性)。

借鉴了 Flux、Redux 和 The Elm Architecture。

flux：可多个store共存，处理多个store比较麻烦。

redux：共享一个store，非惰性加载，学习成本高(尤其是中间件，洋葱)。

vuex：可按modules加载，一个或多个store都可实现，api学习成本低，数据驱动视图部分mvvm底层封装用户不可见，用户只需要了解dispatch到mutations对state修改步骤即可。

vue-emit： emit管理关系乱，dispatch&actions都放在view中。


## 辅助函数
Getter：访问store 或 view中的state。用的不多

mapGetters：映射 store中的state。

mapState：

actions：commit函数 type parmas 此处可以异步api操作之类。async await 搞起来，不同返回值不同commit。

mapActions：可以在视图中直接调用actions的方法。如果不用actions或以store.dispatch。

Mutation：只可以在此处修改store中的state，在其它地方修改的。

被store.commit(type,parmas)调用。

对象的替换不要 state.module = { newProp: 123 } 。

正确姿势更新值：state.module = { ... state.module, newProp: 123 }  用新值解构以前的老值

正确姿势添加新值：Vue.set(state.module, 'newProp', 123);

事件类型最好用常量代替
mutations事件必须是同步函数(修改数据混乱)

```
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```





## 相关注意知识点
computed：有计算缓存，如果你需要每次都更新view，建议使用watch。

对象展开运算符：... Mutation中常用的，不要直接替换旧state，不能异步，否则devtools会报错，这个工具会有新旧快照对比。store.state必须在此处操作。

```
let arr = ["蚂蚁部落", 6, "softwhy.com"];
console.log([1,2,...arr]);
// [1, 2, "蚂蚁部落", 6, "softwhy.com"]





let str = "蚂蚁部落";
console.log([...str])
// ["蚂", "蚁", "部", "落"]




let infor={
  target:"分享互助",
  url:"www.softwhy.com"
}
let obj={
  address:"青岛市南区",
  age:4,
  ...infor
};
console.log(obj);

// {address: "青岛市南区", age: 4, target: "分享互助", url: "www.softwhy.com"}


```


## 严格模式
```
const store = new Vuex.Store({
  // ...
  strict: true
})
```
在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，

将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到



## 表单处理

这块是个麻烦的问题.

当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 v-model 会比较棘手。

由于这个修改不是在 mutation 函数中执行的, 这里会抛出一个错误。

官方给出了两个解决办法。但都不是很理想.

```
// 方法1

<input :value="message" @input="updateMessage">


computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
  	 // 直接commit调用mutations同步这么写很好
  	 // 但是异步呢？ 还是需要执行 dispatch(actionsTypes)
    this.$store.commit('updateMessage', e.target.value)
  }
}




mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}


方法2
<input v-model="message">

computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
方法3 iview vuex
监听方式dispatch||actions=>commitstore下面的值改变

方法3 iview+vuex
校验通过后进行dispatch||actions=>commit
```









