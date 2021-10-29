## 项目是怎么跑起来的
属于多页应用，这里面有很多子项目（`pages/`）：
- 在 webpack 配置的 entry 里可以看到这些子项目入口
- 对于每一个 page，都有对应的 `HtmlWebpackPlugin` 指定它的模板，并注入它需要的 chunks （对应每一个 entry 打包出的 js）
    - 指定 `chunks` 是因为项目是多 entry 会生成多个编译后的 js 文件，chunks 决定使用哪些 js 文件，如果没有指定默认会全部引用
    - `inject` 值为 true，表明 chunks js 会被注入到 html 文件的 body 底部
- 每一个 page 里的 js 文件会创建该子项目的 Vue 实例，指定对应的 component, router, store
- 每一个 page 有对应的 `router/`，这是子项目的路由，而且每个路由加载的 component 都是异步获取，在访问该路由时按需加载
- webpack 打包时（`dist/`）会 emit 出所有 HtmlWebpackPlugin 生成的 html 文件（这也是浏览器访问的入口），相对每个 entry 打包出的 js 文件（filename, `js/[name].[chunkhash].js`），所有异步加载的组件 js（chunkFilename, `js/[id].[chunkhash].js'`） 


## webpack 配置
### filename 和 chunkFilename
- `filename` 是对应于 entry 里面的输入文件，经过打包后输出文件的名称。`chunkFilename` 指未被列在 entry 中，却又需要被打包出来的 chunk 文件的名称，一般是要懒加载的代码。
- `output.filename` 的输出文件名是 `[name].[chunkhash].js`，`[name]` 根据 entry 的配置推断为 index，所以输出为 `index.[chunkhash].js`。`output.chunkFilename` 默认使用 `[id].js`, 会把 `[name]` 替换为 chunk 文件的 id 号。
- `chunkhash` 根据不同的入口文件构建对应的 chunk，生成对应的哈希值，来源于同一个 chunk，则 hash 值就一样。

### resolve
- extensions 数组，在 import 不带文件后缀时，webpack 会自动带上后缀去尝试访问文件是否存在。
- alias 配置别名，把导入路径映射成一个新的导入路径。Vue npm 包有不同的 Vue.js 构建版本（运行时 + 编译器 vs 只包含运行时），如果要使用完整版，则需要在打包工具里配置一个别名 `'vue$': 'vue/dist/vue.esm.js`，这样 import vue 是为打包工具提供的 ES Modules。
- modules 数组，tell webpack what directories should be searched when resolving modules, 默认是去 node_modules 目录下寻找。


## 路由相关
- 使用 `vue-router 3.x`，由于 VueRouter 是 default export 只有一个，所以在引入时可以任意起名字。
- 创建 Vue 时配置 router 参数传入一个 Router 实例。构建 Router 实例，路由有两种模式 hash 和 history，这里在 prod 使用 history mode，dev 使用 hash mode，`fallback` 属性的含义是当浏览器不支持 `history.pushState` 是否回退到 hash 模式，默认为 true。
- 嵌套路由配置 children，组件内 `this.$route` 可以访问到 params, query, hash，匹配多个路由时优先级就按照路由的定义顺序
- 每一个 route 有名字，在跳转时可以指定该 name，如果需要带上 params 和 query 对象 (`<router-link>` 的 `to` 属性和调用`this.$router.push` 或 `this.$router.replace` 传递参数是一回事）
- 配置路由懒加载，可以写成 `component: () => import('xx.vue')`，也可以写成 `resolve => require(['xx.vue'], resolve)`，这里 resolve 就是 promise 的 resolve 回调，组件加载成功后调用。因为 webpack 支持多种模块规范语法，所以有不同写法。
- 关于路由 guard 函数，可以在整个路由对象上定义 `beforeEach` 和 `afterEach`，也可以在组件内定义 `beforeRouteEnter`, `beforeRouteUpdate`, `beforeRouteLeave`，这些函数都有 `to`, `from`, `next` 三个参数，可以帮助判断是从哪个路由进入的或者要离开时给出弹窗提示
- 定义路由时配置 meta 字段，在所有可以访问到它对应的 `to` 或 `from` 参数中（Route 对象）读取该字段
- 创建路由时可以提供一个 `scrollBehavior` 方法返回滚动位置，`scrollBehavior (to, from, savedPosition) {}`，其中第三个参数 `savedPosition` 当且仅当通过浏览器的 前进/后退 按钮触发时才可用


## Vuex 相关
- Vuex store，主要包括 state，mutations，actions；从 store 中读取状态是在 computed 中返回某个 state，触发变化是在组件的 methods 中 commit mutation。在创建 Vue 实例时，注入一个 store 实例，从而在所有子组件可以访问 `this.$store`
- 不想在组件内重复的写 `this.$store.state.xx`，`this.$store.commit`，`this.$store.dispatch`，使用 `mapState`, `mapMutations`, `mapActions` 辅助函数把 store 中同名的状态或操作映射到组件内，相当于在组件内直接定义了这些计算属性和方法
- 提交 mutation 是更改状态的唯一方法，并且这个过程是同步的（类似于 reducer），action (接收一个 store，可以解构) 提交 mutation，`store.dispatch` 返回一个 Promise 可以组合下一个 action
- 可以把 store 分割成多个 module，每个 module 拥有自己的 state，mutations，actions，创建 Vuex.Store 时传入 modules 配置。可以给每个 module 添加 `namespaced: true` 使其成为带命名空间的模块，此时在组件内需将 module 的名字作为第一个参数传递给 `mapState`, `mapActions`，这样所有的映射都会自动将这个 module 作为上下文（也可以用 `createNamespacedHelpers('some/nested/module')` 函数，它会返回绑定在给定命名空间上的 `mapState`, `mapActions`）
- 如果 store 文件太大，可以将 mutations，actions 以及每个 module 分割到单独的文件


## Vue Loader
