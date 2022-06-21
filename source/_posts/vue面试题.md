---
title: vue面试题
toc: true
date: 2022-06-20 17:44:24
tags: [Vue]
urlname:
categories: [Vue]
recommend:
cover:
keywords:  vue面试题
top: 3
---
# props、methods、data、computed、watch初始化顺序

Vue在初始化事会调用`initState`方法
<!--more-->
```typescript
export function initState(vm: Component) {
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)

  // Composition API
  initSetup(vm)
	// methods->data->computed->watch
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    const ob = observe((vm._data = {}))
    ob && ob.vmCount++
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

- 从改方法中可以看出Vue初始化顺序：props->methods->data->computed->watch

# v-if、v-for优先级

```js
export function genElement(el: ASTElement, state: CodegenState): string {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre
  }

  if (el.staticRoot && !el.staticProcessed) {
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    return genSlot(el, state)
  } else {
    // component or element
    let code
    if (el.component) {
      code = genComponent(el.component, el, state)
    } else {
      let data
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData(el, state)
      }

      const children = el.inlineTemplate ? null : genChildren(el, state, true)
      code = `_c('${el.tag}'${
        data ? `,${data}` : '' // data
      }${
        children ? `,${children}` : '' // children
      })`
    }
    // module transforms
    for (let i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code)
    }
    return code
  }
}
```

- 在源码中发现 先处理静态节点（staticRoot） 然后处理once 最后才会处理for 代码显示for优先于if 断点调试之后也是证实for优先于if 结论：

**1、v-for优先于v-if被解析（把你怎么知道的告诉面试官，看过源码）**
 **2、如果同时出现，每次渲染都会先执行循环在判断条件，无论如何循环都不可避免，浪费了性能**
 **3、要避免出现这种情况，在外层嵌套template，在这一层进行v-if判断，然后在内部进行v-for循环**

# Vue组件之间的通信

1.组件间常用方式有以下8种：

- props
- $emits/props
- ~~$children~~/$parent
- $attrs/~~$listeners~~
- ref
- $root
- ~~eventbus~~
- vuex

注意vue3种废弃的几个api [$children](https://v3.cn.vuejs.org/guide/migration/children.html#%E6%A6%82%E8%A7%88)、[$lisnters](https://v3.cn.vuejs.org/guide/migration/listeners-removed.html#%E6%A6%82%E8%A7%88)、[$eventbus](https://v3.cn.vuejs.org/guide/migration/events-api.html#%E6%A6%82%E8%A7%88)

父子通信：props、$emits、$parent、ref、$attrs

兄弟组件：$parent、$root、eventsbus、vuex

跨层级：eventbus、vuex、provide/inject

# Vue生命周期以及每个阶段做的事

1.每个Vue组件被创建后都会经过一系列初始化步聚，比如，他需要数据观测，模版编译，挂载实例到dom上，以及数据变化时更新dom。

2.Vue生命周期总共分为8个阶段：创建前后，载入前后，更新前后，销毁前后，以及一些特殊的生命周期。Vue3中新增了3个用于调试和服务端渲染场景。

| Vue2          | Vue2            | 描述                                     | 结合实践                                                     |
| ------------- | --------------- | ---------------------------------------- | ------------------------------------------------------------ |
| beforeCreate  | beforeCreate    | 组件实例被创建之初                       | 通常用于插件开发中执行一些初始化任务                         |
| created       | created         | 组件实例已经完全创建                     | 组件初始化完毕，可以访问各种数据，获取接口数据等             |
| beforeMounted | beforeMounted   | 组件挂载之前                             |                                                              |
| mounted       | mounted         | 组件挂载到实例上去之后                   | dom已创建，可用于获取访问数据和dom元素；访问子组件等。       |
| beforeUpdate  | beforeUpdate    | 组件数据发生变化，更新之前               | 此时`view`层的还未更新，可用于更新前获取各种状态。           |
| updated       | updated         | 组件数据更新之后                         | 完成view`层的更新，更新后，所有状态已是最新。                |
| beforeDestroy | beforeUnmounted | 组件实例销毁之前                         | 实例销毁前调用，可用于一些定时器或订阅的取消。               |
| destroyed     | unmounted       | 组件实例销毁之后                         | 销毁一个实例。可清理它与其他实例的连接，解绑它的全部指令及时间监听器 |
| activated     | activated       | keep-alive缓存的组件激活时               |                                                              |
| deactivated   | deactivated     | keep-alive缓存的组件停用时调用           |                                                              |
| errorCaptured | errorCaptured   | 捕获一个来自子孙组件的错误时被调用       |                                                              |
| --            | renderTracked   | 调试钩子，响应式依赖被收集时调用         |                                                              |
| --            | renderTriggered | 调试钩子，响应式依赖被触发时调用         |                                                              |
| --            | serverPrefetch  | Sir only，组件实例在服务器上被渲染前调用 |                                                              |

<div style="display:flex;width:100%;background:#fff;">
  <img src='https://cn.vuejs.org/images/lifecycle.png' style="display:block;width:50%;border-right:1px solid #000;object-fit:contain">
  <img src='https://staging-cn.vuejs.org/assets/lifecycle.16e4c08e.png' style="display:block;width:50%;object-fit:contain">
</div>



# 双向绑定使用和原理

1.vue中双向绑定是一个指令`v-model`，可以绑定一个响应式数据到视图，同时视图中变化能改变该值。

2.`v-model`是语法糖，默认情况下相当于:`value`和`@input`。使用`v-model`可以减少大量繁琐的事件处理代码，提高开发效率。

3.通常在表单项使用`v-model`，还可以在自定义组件上使用，表示某个值的输入和输出控制。

4.通过`<input v-model="xxx"/>`的方式绑定到表单元素value上，对于checkbox，可以使用`true-value`和`false-value`指定特殊的值，对于radio可以使用value指定特殊的值；对于select可以通过options元素的value设置特殊的值，还可以结合.lazy，.number，.trim对v-model的行为做进一步的限制；`v-model`用在自定义组件上时又有很大的不同，vue3中它类似于`.sync`修饰符，最终展开的结果时modelValue属性和update:modelValue事件；vue3中我们甚至可以用参数形式指定多个不同的绑定，例如：v-model:foo和v-model:bar。

5.我做过测试，输出包含v-model模板的组件渲染函数，发现它会被转换为value属性的绑定以及一个事件监听，事件回调函数中会做相应变量更新操作，这说明神奇魔法实际上是vue的编译器完成的。

# Vue中如何扩展一个组件

<font size="4">答题思路：</font>

按照逻辑扩展和内容扩展来列举，逻辑扩展有：mixins、extends、composition api；内容扩展有slots；

分别说出他们使用方法、场景差异和问题。

作为扩展，还可以说说vue3中新引入的composition api带来的变化。

1. 常见的组件扩展方法有：mixins，slots，extends等
2. 混入mixins是分发 Vue 组件中可复用功能的非常灵活的方式。混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被混入该组件本身的选项。
3. 插槽主要用于vue组件中的内容分发，也可以用于组件扩展。如果要精确分发到不同位置可以使用具名插槽，如果要使用子组件中的数据可以使用作用域插槽。
4. 组件选项中还有一个不太常用的选项extends，也可以起到扩展组件的目的。
5. 混入的数据和方法不能明确判断来源且可能和当前组件内变量产生命名冲突，vue3中引入的composition api，可以很好解决这些问题，利用独立出来的响应式模块可以很方便的编写独立逻辑并提供响应式的数据，然后在setup选项中有机组合使用。

<font size="4">可能追问：</font>

Vue.extend方法你用过吗？它能用来做组件扩展吗？

# 子组件可以直接改变父组件的数据么?

### 分析

这是一个实践知识点，组件化开发过程中有个单项数据流原则，不在子组件中修改父组件是个常识问题。

参考文档：https://staging.vuejs.org/guide/components/props.html#one-way-data-flow

------

### 思路

1. 讲讲单项数据流原则，表明为何不能这么做
2. 举几个常见场景的例子说说解决方案
3. 结合实践讲讲如果需要修改父组件状态应该如何做

------

### 回答范例

1. 所有的 prop 都使得其父子之间形成了一个单向下行绑定：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外变更父级组件的状态，从而导致你的应用的数据流向难以理解。另外，每次父级组件发生变更时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器控制台中发出警告。 const props = defineProps(['foo']) // ❌ 下面行为会被警告, props是只读的! props.foo = 'bar'

------

1. 实际开发过程中有两个场景会想要修改一个属性：
   - 这个 prop 用来传递一个初始值；这个子组件接下来希望将其作为一个本地的 prop 数据来使用。在这种情况下，最好定义一个本地的 data，并将这个 prop 用作其初始值： const props = defineProps(['initialCounter']) const counter = ref(props.initialCounter)
   - 这个 prop 以一种原始的值传入且需要进行转换。在这种情况下，最好使用这个 prop 的值来定义一个计算属性： const props = defineProps(['size']) // prop变化，计算属性自动更新 const normalizedSize = computed(() => props.size.trim().toLowerCase())
2. 实践中如果确实想要改变父组件属性应该emit一个事件让父组件去做这个变更。注意虽然我们不能直接修改一个传入的对象或者数组类型的prop，但是我们还是能够直接改内嵌的对象或属性。

# Vue要做权限管理该怎么做？控制到按钮级别的权限怎么做？

### 分析

综合实践题目，实际开发中经常需要面临权限管理的需求，考查实际应用能力。

权限管理一般需求是两个：页面权限和按钮权限，从这两个方面论述即可。

[![img](https://camo.githubusercontent.com/a8d70aafa46e5ee1afca79506eff90acf3824c757ddb8118c8a90370543db99b/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f65366339643234656c79316830776a7a69623478316a3231376b30736f3736782e6a7067)](https://camo.githubusercontent.com/a8d70aafa46e5ee1afca79506eff90acf3824c757ddb8118c8a90370543db99b/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f65366339643234656c79316830776a7a69623478316a3231376b30736f3736782e6a7067)

------

### 思路

1. 权限管理需求分析：页面和按钮权限
2. 权限管理的实现方案：分后端方案和前端方案阐述
3. 说说各自的优缺点

------

### 回答范例

1. 权限管理一般需求是**页面权限**和**按钮权限**的管理

2. 具体实现的时候分后端和前端两种方案：

   前端方案会**把所有路由信息在前端配置**，通过路由守卫要求用户登录，用户**登录后根据角色过滤出路由表**。比如我会配置一个`asyncRoutes`数组，需要认证的页面在其路由的`meta`中添加一个`roles`字段，等获取用户角色之后取两者的交集，若结果不为空则说明可以访问。此过滤过程结束，剩下的路由就是该用户能访问的页面，**最后通过`router.addRoutes(accessRoutes)`方式动态添加路由**即可。

   后端方案会**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息

   按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

3. 纯前端方案的优点是实现简单，不需要额外权限管理页面，但是维护起来问题比较大，有新的页面和角色需求就要修改前端代码重新打包部署；服务端方案就不存在这个问题，通过专门的角色和权限管理页面，配置页面和按钮权限信息到数据库，应用每次登陆时获取的都是最新的路由信息，可谓一劳永逸！

------

### 知其所以然

路由守卫

https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L13-L14

路由生成

https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/store/modules/permission.js#L50-L51

动态追加路由

https://github1s.com/PanJiaChen/vue-element-admin/blob/HEAD/src/permission.js#L43-L44

------

### 可能的追问

1. 类似`Tabs`这类组件能不能使用`v-permission`指令实现按钮权限控制？

   ```
   <el-tabs> 
     <el-tab-pane label="⽤户管理" name="first">⽤户管理</el-tab-pane> 
   	<el-tab-pane label="⻆⾊管理" name="third">⻆⾊管理</el-tab-pane>
   </el-tabs>
   ```

------

1. 服务端返回的路由信息如何添加到路由器中？

   ```
   // 前端组件名和组件映射表
   const map = {
     //xx: require('@/views/xx.vue').default // 同步的⽅式
     xx: () => import('@/views/xx.vue') // 异步的⽅式
   }
   // 服务端返回的asyncRoutes
   const asyncRoutes = [
     { path: '/xx', component: 'xx',... }
   ]
   // 遍历asyncRoutes，将component替换为map[component]
   function mapComponent(asyncRoutes) {
     asyncRoutes.forEach(route => {
       route.component = map[route.component];
       if(route.children) {
         route.children.map(child => mapComponent(child))
       }
   	})
   }
   mapComponent(asyncRoutes)
   ```

# vue响应式理解

### 思路

- 什么是响应式？

  > 所谓数据响应式就是能够使数据发生变化可以被检测并对这种变化做出响应式的机制。

- 为什么vue需要响应式？

  > MVVM框架中要解决的一个核心问题就是连接数据层和视图层，通过数据驱动应用，数据变化，视图更新，要做到这点就需要对数据做响应式处理，这样一旦数据发生了变化就可以立即做出更新处理。

- 它能给我们带来什么好处？

  >  以vue为例说明，通过数据响应式加上虚拟DOM和patch算法，开发人员只需要操作数据，关心业务，完全不用接触频繁的DOM操作，从而大大提升开发效率，降低开发难度。

- vue的响应式是怎么实现的？有哪些优缺点？

  >  vue2中数据响应式会根据数据类型来做不同的处理，如果是对象则采用Object.defineProperty()的方式定义数据拦截，当数据被访问或发生变化时，我们感知并做出响应；如果是数组则通过覆盖数组对象原型的7个数组变更方法，使这些方法可以做额外的更新通知，从而做出响应。这种机制很好的解决了数据响应化的问题，但在实际使用中也存在一些缺点：比如初始化时需要递归遍历整个数据造成的性能损失；新增或删除属性时需要用户使用Vue.set/delete这样特殊的api才生效；对于es6中新产生的Map、Set这些数据结构不支持等问题。

- vue3中的响应式的新变化

  > 为了解决这些问题，vue3重新编写了这一部分的实现：利用ES6点Proxy代理要响应化的数据，它有很多好处，编程体验是一致的，不需要使用特殊的api，初始化和内存消耗都得到了大幅改善；另外由于响应化的实现代码抽取为独立的reactivity包，使我们可以更灵活的shying它，第三方的扩展开发起来更加灵活，缺点就是在兼容性上没有Object.defineProperty高，在一些考虑老设备的开发上需要关注这方面。

### 相关源码

vue2

- https://github1s.com/vuejs/vue/blob/HEAD/src/core/observer/index.ts#L135-L136

vue3

- https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/reactive.ts#L89-L90
- https://github1s.com/vuejs/core/blob/HEAD/packages/reactivity/src/ref.ts#L67-L68