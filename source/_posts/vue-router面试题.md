---
title: vue-router面试题
toc: true
date: 2022-07-18 17:26:32
tags: [vue-router]
urlname:
categories: [vue-router]
recommend:
cover:
keywords: vue-router面试题
top: 3
---

# 路由重定向的方法

<!-- more -->

- 第一种

  在路由列表中添加redirect，redirect为重定向的路径名

  ```js
  const router = new VueRouter({
      routes: [
          { path: '/a', redirect: '/b' }
      ]
  })
  ```

- 第二种

  在路由列表中添加redirect，redirect为一个对象`{name:'xxx'}`

  ```js
  const router = new VueRouter({
      routes: [
          { path: '/a', redirect: { name: 'foo' }}
      ]
  })
  ```

- 第三种方法

  在路由列表中添加redirect，redirect为一个方法，方法有一个`to`参数，表示跳转路由的信息

  ```js
  const router = new VueRouter({
      routes: [
          { 
              path: '/a', 
              redirect: to =>{
                  // 方法接收 目标路由 作为参数
        					// return 重定向的 字符串路径/路径对象
                  const { hash, params, query } = to
                  if (query.to === 'foo') {
                      return { path: '/foo', query: null }
                  }else{
                     return '/b' 
                  }
              }
              
          }
      ]
  })
  ```

# 如何配置404

  - Vue-router 3.x

    ```js
    const router = new VueRouter({
        routes: [
            {
                path: '*', redirect: {path: '/404'}
            }
        ]
    })
    ```

  - Vue-router 4.x

    ```js
    const routes = [
      // 将匹配所有内容并将其放在 `$route.params.pathMatch` 下
      { path: '/:pathMatch(.*)*', name: 'NotFound', component: NotFound },
      // 将匹配以 `/user-` 开头的所有内容，并将其放在 `$route.params.afterUser` 下
      { path: '/user-:afterUser(.*)', component: UserGeneric },
    ]
    ```

# 切换路由时，需要保存草稿的功能，怎么实现呢？

  ```vue
  <keep-alive :include="include">
      <router-view></router-view>
   </keep-alive>
  ```

  其中include可以是个数组，数组内容为路由的name选项的值。

# 路由有几种模式？说说它们的区别？

  `hash`: 兼容所有浏览器，包括不支持 HTML5 History Api 的浏览器，例http://www.abc.com/#/index，hash值为#/index， hash的改变会触发hashchange事件，通过监听hashchange事件来完成操作实现前端路由。hash值变化不会让浏览器向服务器请求。

  ```csharp
  // 监听hash变化，点击浏览器的前进后退会触发
  window.addEventListener('hashchange', function(event){ 
      let newURL = event.newURL; // hash 改变后的新 url
      let oldURL = event.oldURL; // hash 改变前的旧 url
  },false)
  复制代码
  ```

 ` history`: 兼容能支持 HTML5 History Api 的浏览器，依赖HTML5 History API来实现前端路由。没有`#`，路由地址跟正常的url一样，但是初次访问或者刷新都会向服务器请求，如果没有请求到对应的资源就会返回404，所以路由地址匹配不到任何静态资源，则应该返回同一个index.html 页面，需要在nginx中配置。

  `abstract`: 支持所有 JavaScript 运行环境，如 Node.js 服务器端。如果发现没有浏览器的 API，路由会自动强制进入这个模式。

# 路由守卫

- 全局前置守卫`beforeEach`

  ```js
  router.beforeEach((to, from, next) => {
    // ...
    next()
  })
  ```

- 全局解析守卫`beforeResolve`

  ```js
  router.beforeEach((to, from, next) => {
    // ...
    next()
  })
  ```

- 全局后置钩子`afterEach`

  你也可以注册全局后置钩子，然而和守卫不同的是，这些钩子不会接受 `next` 函数也不会改变导航本身：

  ```js
  router.afterEach((to, from) => {
    // ...
    next()
  })
  ```

- 路由独享守卫`beforeEnter`

  ```js
    routes: [
      {
        path: '/foo',
        component: Foo,
        beforeEnter: (to, from, next) => {
          // ...
        }
      }
    ]
  ```

- 组件内的守卫

  - `beforeRouteEnter`
  - `beforeRouteUpdate` (2.2 新增)
  - `beforeRouteLeave`

  ```js
  const Foo = {
    template: `...`,
    beforeRouteEnter(to, from, next) {
      // 在渲染该组件的对应路由被 confirm 前调用
      // 不！能！获取组件实例 `this`
      // 因为当守卫执行前，组件实例还没被创建
    },
    beforeRouteUpdate(to, from, next) {
      // 在当前路由改变，但是该组件被复用时调用
      // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
      // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
      // 可以访问组件实例 `this`
    },
    beforeRouteLeave(to, from, next) {
      // 导航离开该组件的对应路由时调用
      // 可以访问组件实例 `this`
    }
  }
  ```

  `beforeRouteEnter` 守卫 **不能** 访问 `this`，因为守卫在导航确认前被调用，因此即将登场的新组件还没被创建。

  不过，你可以通过传一个回调给 `next`来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数。

  ```js
  beforeRouteEnter (to, from, next) {
    next(vm => {
      // 通过 `vm` 访问组件实例
    })
  }
  ```

  

# 讲一下导航守卫的三个参数的含义？

每个守卫方法接收三个参数：

- **`to: Route`**: 即将要进入的目标 [路由对象](https://v3.router.vuejs.org/zh/api/#路由对象)
- **`from: Route`**: 当前导航正要离开的路由
- **`next: Function`**: 一定要调用该方法来 **resolve** 这个钩子。执行效果依赖 `next` 方法的调用参数。
  - **`next()`**: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 **confirmed** (确认的)。
  - **`next(false)`**: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。
  - **`next('/')` 或者 `next({ path: '/' })`**: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 [`router-link` 的 `to` prop](https://v3.router.vuejs.org/zh/api/#to) 或 [`router.push`](https://v3.router.vuejs.org/zh/api/#router-push) 中的选项。
  - **`next(error)`**: (2.4.0+) 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 [`router.onError()`](https://v3.router.vuejs.org/zh/api/#router-onerror) 注册过的回调。

**确保 `next` 函数在任何给定的导航守卫中都被严格调用一次。它可以出现多于一次，但是只能在所有的逻辑路径都不重叠的情况下，否则钩子永远都不会被解析或报错**。

# 路由导航的解析流程

- 导航被触发。
- 在失活的组件里调用 `beforeRouteLeave` 守卫。
- 调用全局的 `beforeEach` 守卫。
- 在重用的组件里调用 `beforeRouteUpdate` 守卫(2.2+)。
- 在路由配置里调用 `beforeEnter`。
- 解析异步路由组件。
- 在被激活的组件里调用 `beforeRouteEnter`。
- 调用全局的 `beforeResolve` 守卫(2.5+)。
- 导航被确认。
- 调用全局的 `afterEach` 钩子。
- 触发 DOM 更新。
- 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

```js
beforeRouteEnter(to, from, next) {
    next(vm => {
        //通过vm访问组件实例
    })
},
```

# 路由导航守卫和Vue实例生命周期钩子函数的执行顺序？

路由导航守卫都是在Vue实例生命周期钩子函数之前执行的。

# router-link

`<router-link>`是Vue-Router的内置组件，在具有路由功能的应用中作为声明式的导航使用。

`<router-link>`有8个props，其作用是：

- `to`：必填，表示目标路由的链接。当被点击后，内部会立刻把`to`的值传到`router.push()`

  ，所以这个值可以是一个字符串或者是描述目标位置的对象。

  ```html
  <router-link to="home">Home</router-link>
  <router-link :to="'home'">Home</router-link>
  <router-link :to="{ path: 'home' }">Home</router-link>
  <router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
  <router-link :to="{ path: 'user', query: { userId: 123 }}">User</router-link>
  复制代码
  ```

  注意path存在时params不起作用，只能用query

- `replace`：默认值为false，若设置的话，当点击时，会调用`router.replace()`而不是`router.push()`，于是导航后不会留下 history 记录。

- `append`：设置 append 属性后，则在当前 (相对) 路径前添加基路径。

- `tag`：让`<router-link>`渲染成`tag`设置的标签，如`tag:'li`,渲染结果为`<li>foo</li>`。

- `active-class`：默认值为`router-link-active`,设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

- `exact-active-class`：默认值为`router-link-exact-active`,设置链接被精确匹配的时候应该激活的 class。默认值可以通过路由构造函数选项 linkExactActiveClass 进行全局配置的。

- ```
  exact
  ```

  ：是否精确匹配，默认为false。

  ```xml
  <!-- 这个链接只会在地址为 / 的时候被激活 -->
  <router-link to="/" exact></router-link>
  复制代码
  ```

- `event`：声明可以用来触发导航的事件。可以是一个字符串或是一个包含字符串的数组，默认是`click`。

# 如何在组件中监听路由参数变化？

有两种方法可以监听路由参数的变化，**但是只能用在包含`<router-view />`的组件内。**

- 第一种

  ```js
  watch:{
    '$route':function(to, from){
      ....
    }
  }
  ```

- 第二种

  ```js
  beforeRouteUpdate(to, from, next){
    ....
  }
  ```

# 切换路由后，新页面要滚动到顶部或者原来的位置

在创建router实例时，可以传入`scrollBehavir`一个函数

```js
const router = new Router({
    mode: 'history',
    base: process.env.BASE_URL,
    routes,
    scrollBehavior(to, from, savedPosition) {
        if (savedPosition) {
            return savedPosition;
        } else {
            return { x: 0, y: 0 };
        }
    }
});
```

# 嵌套组件的使用场景

做个管理系统，顶部栏和左侧菜单栏是全局通用的，那就应该放在父路由，而右下的页面内容部分放在子路由。

比如在app.vue文件中

```xml
<template>
  <div>
    <router-view/>
  </div>
</template>
复制代码
```

在layout.vue文件中

```less
<template>
  <div>
    <div>
        //...头部导航
    </div>
    <div>
        //...侧边栏导航
    </div>
    <div>
        //...主内容
        <router-view/>
    </div>
    
  </div>
</template
复制代码
```

在routes.js文件中

```javascript
function load(component) {
    return resolve => require([`views/${component}`], resolve);
}
const routes=[
    {
        path: '/',
        redirect: '/home',
        name: 'layout',
        component: load('layout'),
        children: [
            {
                path: '/home',
                name: 'home',
                component: load('home'),
                meta: {
                    title: '首页'
                },
            },
        ]
    }
]
复制代码
```

然后layout页面就渲染在app.vue文件中的`<router-view/>`上。home页面就渲染在layout.vue文件夹中的`<router-view/>`上。

# 什么是命名视图？

有时候想同时 (同级) 展示多个视图，而不是嵌套展示，例如创建一个布局，有 `sidebar` (侧导航) 和 `main` (主内容) 两个视图，这个时候命名视图就派上用场了。你可以在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。如果 `router-view` 没有设置名字，那么默认为 `default`。

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置 (带上 s)：

```js
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

# 如何获取路由传过来的参数？

路由有三种传参方式，获取方式各不相同。

- `meta`：路由元信息，写在routes配置文件中。

  ```js
  {
      path: '/home',
      name: 'home',
      component: load('home'),
      meta: {
          title: '首页'
      },
  },
  ```

  通过`this.$route.meta.title`获取

- query

  ```js
  this.$route.push({
      path:'/home',
      query:{
          userId:123
      }
  })
  复制代码
  ```

  浏览器地址：`http://localhost:8036/home?userId=123` 获取方式：`this.$route.query.userId`

- `params`:这个方式比较麻烦。

  - 首先要在地址上做配置

    ```js
    {
        path: '/home/:userId',
        name: 'home',
        component: load('home'),
        meta: {
            title: '首页'
        },
    },
    ```

    

    访问传参

    ```js
    const userId = '123'
    this.$router.push({ name: 'home', params: { userId } })
    ```

    

    注意用params传参，只能用命名的路由（用name访问），如果用path，params不起作用。 `this.$router.push({ path: '/home', params: { userId }})`不生效。

    浏览器地址：`http://localhost:8036/home/123`

    获取方式：`this.$route.params.userId`

  

# 路由组件和路由为什么解耦，怎么解耦？

因为在组件中使用 $route 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性，所有要解耦。

- 耦合如以下代码所示。Home组件只有在

  ```
  http://localhost:8036/home/123
  ```

  URL上才能使用。

  ```arduino
  const Home = {
      template: '<div>User {{ $route.params.id }}</div>'
  }
  const router = new VueRouter({
      routes: [
          { path: '/home/:id', component: Home }
      ]
  })
  复制代码
  ```

- 使用 props 来解耦

  - props为true，`route.params`将会被设置为组件属性。
  - props为对象，则按原样设置为组件属性。
  - props为函数，`http://localhost:8036/home?id=123`,会把123传给组件Home的props的id。

  ```yaml
  const Home = {
      props: ['id'],
      template: '<div>User {{ id }}</div>'
  }
  const router = new VueRouter({
      routes: [
          { path: '/home/:id', component: Home, props: true},
          // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
          {
              path: '/home/:id',
              components: { default: Home, sidebar: Sidebar },
              props: { default: true, sidebar: false }
          }
          { path: '/home', component: Home, props: {id:123} },
          { path: '/home', component: Home, props: (route) => ({ id: route.query.id }) },
      ]
  })
  ```

# active-class是那个组件的属性

`<router-link/>`组件的属性，设置链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

# 在vue组件中怎么获取到当前的路由信息？

通过`this.$route`来获取

# 动态加载路由？

使用Router的实例方法addRoutes来实现动态加载路由，一般用来实现菜单权限。

使用时要注意，静态路由文件中不能有404路由，而要通过addRoutes一起动态添加进去。

```js
const routes = [
    {
        path: '/overview',
        name: 'overview',
        component: () => import('@/views/account/overview/index'),
        meta: {
            title: '账户概览',
            pid: 869,
            nid: 877
        },
    },
    {
        path: '*',
        redirect: {
            path: '/'
        }
    }
]
vm.$router.options.routes.push(...routes);
vm.$router.addRoutes(routes);
```

# 怎么实现路由懒加载呢？

```js
function load(component) {
    //return resolve => require([`views/${component}`], resolve);
    return () => import(`views/${component}`);
}

const routes = [
    {
        path: '/home',
        name: 'home',
        component: load('home'),
        meta: {
            title: '首页'
        },
    },
]
```

# 路由之间是怎么跳转的？有哪些方式？

声明式  通过使用内置组件`<router-link :to="/home">`来跳转

编程式  通过调用router实例的push方法`router.push({ path: '/home' })`或replace方法`router.replace({ path: '/home' })`

# 如果vue-router使用history模式，部署时要注意什么？

要注意404的问题，因为在history模式下，只是动态的通过js操作window.history来改变浏览器地址栏里的路径，并没有发起http请求，当直接在浏览器里输入这个地址的时候，就一定要对服务器发起http请求，但是这个目标在服务器上又不存在，所以会返回404。

所以要在Ngnix中将所有请求都转发到index.html上就可以了。

```nginx
location / {
    try_files  $uri $uri/ @router index index.html;
}
location @router {
    rewrite ^.*$ /index.html last;
}
```

# route和router有什么区别？

route是“路由信息对象”，包括path，params，hash，query，fullPath，matched，name等路由信息参数。 而router是“路由实例对象”，包括了路由的跳转方法，钩子函数等。

# Vue路由怎么跳转打开新窗口？

```js
const obj = {
    path: xxx,//路由地址
    query: {
       mid: data.id//可以带参数
    }
};
const {href} = this.$router.resolve(obj);
window.open(href, '_blank');
```

