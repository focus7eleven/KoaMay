#### 核心概念

- 核心是store
- store中的存储是响应式的
- 改变sotre中的状态的唯一途径是显式地 **commit mutations** 。
- 由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在**计算属性**中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutations。








---

### Vue-Router

```jsx
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>


// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')
```



##### 动态路由

```javascript
routes: [
  { path: '/user/:id', component: User}
]
```

- id等参数可以在`this.$route.params`中取到
- 路由渲染同个组件，只是参数不一样时，原来的组件实例 **会被复用**，意味着组件的 **生命周期钩子不会再被调用**
- 支持 **正则** 匹配



##### 嵌套路由

```jsx
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}

const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```



##### 编程时导航

router.push(location)

- 点击 `<router-link>` 时，这个方法会在内部调用，所以说，点击 `<router-link :to="...">` 等同于调用 `router.push(...)`。

- 参数：

  ```javascript
  // 字符串
  router.push('home')

  // 对象
  router.push({ path: 'home' })

  // 命名的路由
  router.push({ name: 'user', params: { userId: 123 }})

  // 带查询参数，变成 /register?plan=private
  router.push({ path: 'register', query: { plan: 'private' }})
  ```



router.replace(loaction)

- 与 `router.push` 很像，但不会向history添加新纪录，只会体会history栈顶的那条记录



router.go(n)

- 类似 `window.history.go(n)`

  ```javascript
  // 在浏览器记录中前进一步，等同于 history.forward()
  router.go(1)

  // 后退一步记录，等同于 history.back()
  router.go(-1)
  ```



##### 命名路由

通过名称来标识路由。

```jsx
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})

<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```





##### 命名视图

如果想同时在同级展示多个视图，例如有sidebar和main两个视图，就需用到命名视图。

默认没有设置名字的情况下，视图名为default。

```jsx
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>

const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```





##### 导航钩子

用来拦截导航，让它完成跳转或取消。

###### 全局钩子

可以使用 `router.beforeEach` 注册一个全局的before钩子：

```javascript
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

钩子是一部解析执行，导航在所有钩子resolve完之前一直处于 **等待中**

next参数：

- 一定要调用该方法来resolve钩子。
- next()：进行管道中的下一个钩子。如果全部钩子执行完了，导航的状态就是confirmed
- next(false)：中断当前的导航。
- next('/') 或者next({ path: '/' })，跳转到一个不同的地址，当前的导航被中断，然后进行一个新的导航。



