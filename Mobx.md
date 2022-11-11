## Mobx基本概念



### 介绍

- 简单，可拓展的状态管理工具

- 通过运用透明的函数式响应式编程使状态管理变得简单和可拓展




### Mobx 和 Redux 对比

#### Redux

- 有严格的工作流程，需要写大量的模板代码

- 需要保证数据不可变（替换）

- Redux 需要中间件处理异步thunk、suga

- Redux 约束强，更适合大型多人协作开发




#### Mobx

- 无模板代码，非常简洁

- 数据是响应式的，可直接修改数据（Proxy）

- Mobx可直接处理异步

- Mobx灵活简单，适合规模不大的应用




![image-20220307234950214](C:\Users\lingu\AppData\Roaming\Typora\typora-user-images\image-20220307234950214.png)

安装mobx、mobx-react(支持类组件 函数组件)、mobx-react-lite(支持函数组件，轻量)



### 核心概念



#### 核心API

- obeservable 定义一个存储 state 的可追踪字段（proxy）（响应式状态
- action 将一个方法标记为可以修改 state 的action （方法干啥的，修改状态的
- computed 标记一个可以由 state 派生出新值并且缓存其输出的计算属性

工作流程图 actions -- obeservable -- 计算值 -- render



#### 创建 store 

- 新建文件 store/Counter.ts 通过 class 创建一个 Counter 类
- 使用 makeObservable 将类的属性和方法变成响应式的
- 到处 Counter 实例，注意：mobx 中每一个 store 都只初始化一次（因此导出实例）
- 用 observer（高阶组件） 包裹组件，和 store 产生连接



#### this 的指向问题

默认 class 中的方法不会绑定 this，this 指向取决于如何调用

~~~react
<button onClick={() => counter.increment()}>increment</button> 
<button onClick={counter.increment}>increment</button> // 有问题
~~~

解决方法：在使用 `makeObservable` 时可以通过 `action.bound` 绑定 this 指向，使其指向当前类

~~~js
constructor() {
    makeObservable(this, { 
        count: observable, 
        increment: action.bound,
        reset: action.bound
    }) 
}
~~~



#### 计算属性的使用

- computed 可以用来从其他可观察对象中派生信息
- 计算值采用惰性求值，会缓存其输出，并且只有当其依赖的可观察对象被改变时才会重新计算
- 计算属性是一个方法，且方法前面必须使用 get 进行修饰
- 计算属性还需要通过 makeObservable 方法指定

~~~js
constructor() {
    makeObservable(this, {
        double: computed
    }) 
}
get double() {
    return this.count * 2
}
~~~



#### makeAutoObservable 的使用

- makeAutoObservable 就像加强版的 makeObservable，默认情况下，它将推断所有属性
- 推断规则：
  - 所有属性都为 observable
  - 所有方法都为 action
  - 所有 get 都为 computed
- 通过 overrides 排除不需要被观察的属性和方法
- 通过 autoBind 可以绑定 this 指向

~~~js
// makeObservable(this, { 
//   count: observable, 
//   increment: action.bound,
//   reset: action.bound,
//   double: computed
// }) 
makeAutoObservable(this, { reset: false }, { autoBind: true })
~~~



#### Mobx 监听属性



##### autorun 的使用

- autorun 函数接收一个函数作为参数，每当该函数所观察的值发生变化时，它都应该运行

- autorun 创建时，会执行一次

- Mobx 会自动收集并订阅所有的可观察属性，一旦发生改变，autoRun 自动触发




##### reaction 的使用

- reaction 类似于 autorun，可你让开发者更加精确去控制和跟踪可观察对象

- 接收两个参数：

  1. data 函数，其返回值作为第二个函数的依赖值

  2. 回调函数

- 与 autorun 区别：reaction 在创建时不会自动执行



### Mobx 处理异步



如何处理

异步进程在 Mobx 中不需要任何特殊处理，因为不论是何时引发的所有 reactions 都将会自动更新

因为可观察对象是可变的，因此在 action 执行过程中修改一般是安全的

如果可观察对象的修改不是 action 函数中，控制台会报警告（可关闭但不推荐）



### Mobx 模块化



多个 store 场景

- 当项目规模变大后，不能将所有的状态和方法都放到一个 Store 中
- 我们可以根据业务模块定义多个 Store
- 通过一个根 Store 统一管理所有 Store

缺陷：同个组件引用多个 store 的数据需要逐一引入



store 合并

- 拆分 Counter 和 Cart 两个 Store，每个 Store 都有自己的 state / action / computed
- 在 store/index 中导入所有的 Store，组合成一个 Store
- 使用 useContext 机制，自定义 useStore  hook，统一导出 Store

