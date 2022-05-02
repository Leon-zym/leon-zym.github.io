---
layout: mypost
title: Vue Router的使用（一）
categories: [Vue]
---

Vue Router是Vue官方的一个插件库，用来实现SPA（single page web application）。

一个路由（route）就是一组映射关系（key - value），多个路由需要路由器（router）进行管理。对于前端，key是路径，value是组件。

## 基本使用

npm安装好vue-router包后，创建 ./src/router/index.js 文件：

```js
//引入VueRouter
import VueRouter from "vue-router";
//引入需要路由的组件
import About from "../components/About";
import Home from "../components/Home";

//创建router实例对象，去管理一组一组的路由规则
export default new VueRouter({
  routes: [
    {
      path: "/about",
      component: About,
    },
    {
      path: "/home",
      component: Home,
    },
  ],
});
```

在组件的导航部分中实现切换：

```vue
<router-link active-class="active" to="/about">About</router-link>
```

- 使用`<router-link>`代替`<a>`标签，Vue处理后会在html中还原成`<a>`标签
- 可通过`active-class`类名配置高亮样式

在组件的指定位置展示切换后的内容：

```vue
<router-view></router-view>
```

## 注意点

- 路由组件通常存放在 ./src/pages文件夹里，一般组件通常存放在 ./src/components 文件夹里
- 通过切换，不再显示了的路由组件，默认是被销毁掉的，需要的时候再去挂载
- 每个组件身上都有自己独有的`$route`属性，里面存储着自己的路由信息。而整个应用只有唯一一个 router，可以通过组件身上的`$router`属性获取到

## 多级路由

多级路由也叫嵌套路由。

在 ./src/router/index.js 中配置路由规则：

```js
routes: [
  {
    path: "/about",
    component: About,
  },
  {
    path: "/home",
    component: Home,
    children: [
    //通过children配置子级路由
      {
        path: "news",  // 此处一定不要写成 /news
        component: News,
      },
      {
        path: "message",  // 此处一定不要写成 /message
        component: Message,
      },
    ],
  },
];
```

在组件的导航部分中实现切换跳转：

```vue
<router-link to="/home/news">News</router-link>
<!-- 路径要写完整路径 -->
```

## 路由的query参数

在组件中传递query参数：

```vue
<!-- 跳转并携带query参数，to的字符串写法 -->
<router-link :to="`/home/message/detail?id=${m.id}&title=${m.title}`"
>跳转</router-link>

<!-- 跳转并携带query参数，to的对象写法 -->
<router-link
  :to="{
    path: '/home/message/detail',
    query: {id: m.id, title: m.title},
  }"
>跳转</router-link>
```

- 若传递的query参数是表达式，to需要变成v-bind绑定形式。

- to的字符串写法，需要配合模板字符串。而对象写法更加结构化，更推荐

在组件中接收query参数：

```js
$route.query.id;
$route.query.title;
```

## 命名路由

在 ./src/router/index.js 中给路由规则命名：

```js
// 三级路由嵌套
{
  path: '/demo',
  component: Demo,
  children: [
    {
      path: 'test',
      component: Test,
      children: [
        {
          name:'hello'  //给路由命名
          path:'welcome',
          component: Hello,
        }
      ]
    }
  ]
}
```

在组件中简化跳转：

```vue
<!--简化前，需要写完整的路径 -->
<router-link to="/demo/test/welcome">跳转</router-link>

<!--简化后，直接通过名字跳转 -->
<router-link :to="{name: 'hello'}">跳转</router-link>

<!--简化写法配合传递query参数 -->
<router-link
  :to="{
    name: 'hello',
    query: {
      id: 666,
      title: '你好',
    },
  }"
>跳转</router-link>
```

## 路由的params参数

先在 ./src/router/index.js 中声明接收params参数：

```js
{
path: '/home',
component: Home,
children: [
  {
    path: 'message',
    component: Message,
    children: [
      {
        name: 'detail',
        path: 'detail/:id/:title',  //使用占位符声明接收params参数
        component: Detail
      }
    ]
  }
]
}
```

在组件中传递params参数：

```vue
<!-- 跳转并携带params参数，to的字符串写法 -->
<router-link :to="/home/message/detail/666/你好">跳转</router-link>

<!-- 跳转并携带params参数，to的对象写法 -->
<router-link
  :to="{
    name: 'detail',
    params: {
      id: 666,
      title: '你好',
    },
  }"
>跳转</router-link>
```

- 路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置

在组件中接收params参数：

```js
$route.params.id;
$route.params.title;
```

