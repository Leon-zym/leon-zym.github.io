---
layout: mypost
title: Vue Router的使用（二）
categories: [Vue]
---

## 路由的props配置

为了让路由组件更方便地接收参数（query、params）。

在 ./src/router/index.js 中给路由规则配置props：

```js
{
	name: 'detail',
	path: 'detail',
	component: Detail,

	//第一种写法：props值为对象，该对象中所有的key-value组合最终都会通过props传给Detail组件
	props: {a:900},

	//第二种写法：props值为布尔值，布尔值为true，则把路由收到的所有params参数通过props传给Detail组件
	props:true,

	//第三种写法：props值为函数，该函数返回的对象中每一组key-value都会通过props传给Detail组件
	props($route) {
		return {
			id: $route.query.id,
			title: $route.query.title
		}
	}
}
```

在组件中接收props：

```js
props:['id','title']
```

## < router-link > 的replace属性

replace属性的作用是控制路由跳转时控制浏览器历史记录的模式。

浏览器的历史记录有两种写入方式：分别为`push`和`replace`。`push`是追加历史记录，`replace`是替换当前历史记录。路由跳转时候默认为`push`模式。

在组件中更改浏览器历史记录模式为replace：

```vue
<router-link replace ......>News</router-link>
```

## 编程式路由导航

为了不借助`<router-link>`实现路由跳转，让路由跳转更加灵活。

在组件中定义相关methods：

```js
methods: {
  pushShow(m) {
    this.$router.push({
      name: 'detail',
      query: {
        id: m.id,
        title: m.title
      }
    })
  },
  replaceShow(m) {
    this.$router.replace({
      name: 'detail',
      query: {
        id: m.id,
        title: m.title
      }
    })
  },
  
  // this.$router.forward(); //前进
  // this.$router.back(); //后退
  // this.$router.go(n); //n为正，则前进n步。n为负，则后退n步
}
```

## 缓存路由组件

为了让不展示的路由组件保持挂载，不被销毁。

在组件中将用于展示内容的`<router-view>`标签用`<keep-alive>`包裹，并使用`include`属性指定其中要缓存的路由组件名。

```vue
<keep-alive :include="['News', 'Message']"> 
  <router-view></router-view>
</keep-alive>
```

## 两个新的生命周期钩子

`activated()`和`deactivated()`是路由组件所独有的两个钩子，用于捕获路由组件的激活状态。

应先缓存路由组件，不让组件被销毁，再在两个钩子中写回调函数。

## 路由守卫

为了对路由切换跳转进行权限控制。路由守卫分为全局守卫、独享守卫、组件内守卫。

### 全局路由守卫：

在 ./src/router/index.js 中需要先用变量接住router实例对象，然后执行路由守卫的代码，最后再将router暴露出去。

配置全局路由守卫：

```js
//全局前置守卫：初始化时执行、每次路由切换 前 执行
router.beforeEach((to, from, next) => {
//传入三个参数，to为目标组件的路由信息，from为原组件的路由信息，next为允许放行的方法
  if (to.meta.isAuth) {
  //判断当前路由是否需要进行权限控制。可在需要权限控制的组件路由配置中添加meta配置项
    if (localStorage.getItem("school") === "atguigu") {
    //权限控制的具体规则。此处为从localStorage中读取权限信息
      next();  //放行
    } else {
      alert("暂无权限查看");
    }
  } else {
    //不需要权限控制的组件直接放行
    next(); //放行
  }
});

//全局后置守卫：初始化时执行、每次路由切换 后 执行
router.afterEach((to, from) => {
//只有两个参数，没有next方法
  if (to.meta.title) {
  //判断组件的路由信息的meta中是否配置了title
    document.title = to.meta.title;  //修改页面的title
  } else {
    //否则为默认的页面title
    document.title = "vue_test";
  }
});
```

### 独享守卫

在单独组件的路由信息中配置独享守卫：

```js
{
  name: 'news',
  path: 'news',
  component: News,
  meta: {isAuth:true, title:'新闻'},
  //独享守卫的配置项
  beforeEnter(to,from,next) {
    if (to.meta.isAuth) {  //判断当前路由是否需要进行权限控制
      if (localStorage.getItem('school') === 'atguigu') {
        next()
      } else {
        alert('暂无权限查看')
      }
    } else {
      next()
    }
  }
}
```

### 组件内守卫

在组件内配置项中配置守卫：

```js
//进入守卫：通过路由规则，进入该组件时被调用
beforeRouteEnter (to, from, next) {
  ......
},
//离开守卫：通过路由规则，离开该组件时被调用
beforeRouteLeave (to, from, next) {
  ......
}
```

## 路由器的两种工作模式

 对于一个URL来说，`#`及其后面的内容就是hash值。hash值不会包含在HTTP请求中，即hash值不会带给服务器。

路由的默认模式为`hash`模式。地址中永远带着#号，不美观，且URL可能会被标记为不合法。但其兼容性较好。

路由的模式还可以改为`history`模式。地址中没有#号，干净美观。但兼容性和hash模式相比略差，且为了避免页面刷新后404的问题，需要应用部署上线时后端人员提供支持。

在 ./src/router/index.js 中创建路由器时，添加一条工作模式的配置项：

```js
mode: 'history'
```

