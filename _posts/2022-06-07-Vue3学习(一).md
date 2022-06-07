---
layout: mypost
title: Vue3学习(一)
categories: [Vue3]
---

> Vue3 One Piece
>
> 因为有Vue2的基础，所以仅记录Vue3改变和新增的地方~

## vue-cli脚手架的小变化

Vue3在main.js中的引入方式发生变化，引入的不再是Vue构造函数，而是一个名为createApp的工厂函数。

创建的实例对象app也要比Vue2的vm更“轻”，少了许多属性和方法。

```js
// 引入Vue工厂函数
import { createApp } from 'vue'
// 引入根组件App
import App from './App.vue'

// 创建实例对象
const app = createApp(App)

//挂载到#app的DOM元素上
app.mount('#app')
```

Vue3的template模板可以没有根标签包裹了。

```html
<template>
	<img alt="Vue logo" src="./assets/logo.png">
	<HelloWorld msg="Welcome to Your Vue.js App"/>
</template>
```

## Composition API

Composition API相对于Options API来说，是将实现特定功能的所有相关代码(响应式数据、方法等)都放到一起。这样不管应用多大，都可以快速定位到某个功能的所有相关代码，维护方便。如果功能复杂，代码量大，我们还可以进行逻辑拆分处理。[官方文档](https://v3.cn.vuejs.org/guide/composition-api-introduction.html#%E4%BB%80%E4%B9%88%E6%98%AF%E7%BB%84%E5%90%88%E5%BC%8F-api)

![image-2022060664818397 PM](image-2022060664818397%20PM.png)

## setup配置项

setup是Vue3的一个新的配置项，值为一个函数。它是所有Composition API的“舞台”。

组件中要配置的数据、方法等，都需要配置在setup中。而setup函数返回的值可以为一个对象（常用），也可以为一个渲染函数。若返回一个对象，则对象中的属性、方法在模板中均可以直接使用，不需要加this。

```js
setup(){
  //数据
  let name = '张三'
  let age = 18

  //方法
  function sayHello(){
    alert(`我叫${name}，我${age}岁了，你好啊！`)
  }

  //返回一个对象（常用）
  return {
    name,
    age,
    sayHello,
  }

  //返回一个渲染函数
  // return ()=> h('h1','你好啊')
}
```

注意：虽然可以和Vue2的配置项，如data、methods等混用，但建议不要混用。setup函数不能是async函数。

## ref函数

setup中的数据默认不是响应式的，而ref可以定义一个响应式的数据（引用对象 / reference对象 / ref对象）。

```js
const foo = ref(initValue)
```

在js中读取操作数据`foo.value`，而在html模板中读取数据`foo`。

接收的initValue可以是基本类型也可以是对象类型。响应式的实现原理上，基本类型依靠`Object.defineProperty()`，对象类型依靠`reactive`函数。

## reactive函数

用于定义对象类型的响应式数据（代理对象 / Proxy实例对象 / Proxy对象）。

```js
const bar = reactive(initValue)
```

reactive定义的响应式数据是深层次的，即对象内部的所有层次都是响应式的。原理上基于ES6的Proxy实现，通过代理对象操作源对象内部数据。

## Vue3的响应式原理

通过[Proxy代理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)实现拦截对象中任意属性的读取、修改、添加、删除等操作。

通过[Reflect反射](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)实现对源对象中的属性进行操作。

```js
new Proxy(data, {
	// 拦截读取属性值
  get (target, prop) {
  	return Reflect.get(target, prop)
  },
  // 拦截设置属性值或添加新属性
  set (target, prop, value) {
  	return Reflect.set(target, prop, value)
  },
  // 拦截删除属性
  deleteProperty (target, prop) {
  	return Reflect.deleteProperty(target, prop)
  }
})
```

## ref与reactive对比

从定义的数据类型：

- ref用来定义基本数据类型。虽然也可以定义对象类型，但也会通过reactive转换为代理对象
- reactive用来定义对象类型

从实现原理：

- ref通过`Object.defineProperty()`来实现响应式（数据劫持）
- reactive通过`Proxy`来实现响应式（数据劫持），并通过`Reflect`操作源对象内部的数据

从使用方式：

- ref定义的数据，操作数据需要`.value`后缀，读取不需要
- reactive定义的数据，操作和读取数据均不需要后缀

## setup的两个注意点

setup函数会在`beforeCreate()`之前执行一次，其中的this是undefined。

setup的参数：

- props：值为对象，包含：组件外部传递过来，且组件内部声明接收了的属性
- context：上下文对象
  - attrs: 值为对象，包含：组件外部传递过来，但没有在props配置中声明的属性, 相当于`this.$attrs`
  - slots: 收到的插槽内容, 相当于`this.$slots`
  - emit: 分发自定义事件的函数, 相当于`this.$emit`

## computed函数

```js
// 先引入后使用
import {computed} from 'vue'

setup(){
  ...
  
	//计算属性——简写
  let fullName = computed(()=>{
      return person.firstName + '-' + person.lastName
  })
  
  //计算属性——完整
  let fullName = computed({
    get(){
      return person.firstName + '-' + person.lastName
    },
    set(value){
      const nameArr = value.split('-')
      person.firstName = nameArr[0]
      person.lastName = nameArr[1]
    }
  })
}
```

## watch函数

注意：当监视reactive数据时，oldValue无法正确获取，且强制开启了deep深度监视。当监视reactive数据中的某个对象属性值的时候，deep配置有效。

```js
//情况一：监视ref定义的响应式数据
watch(sum,(newValue,oldValue)=>{
	console.log('sum变化了',newValue,oldValue)
},{immediate:true})

//情况二：监视多个ref定义的响应式数据
watch([sum,msg],(newValue,oldValue)=>{
	console.log('sum或msg变化了',newValue,oldValue)
}) 

/* 情况三：监视reactive定义的响应式数据
			若watch监视的是reactive定义的响应式数据，则无法正确获得oldValue！！
			若watch监视的是reactive定义的响应式数据，则强制开启了深度监视 
*/
watch(person,(newValue,oldValue)=>{
	console.log('person变化了',newValue,oldValue)
},{immediate:true,deep:false}) //此处的deep配置不再奏效

//情况四：监视reactive定义的响应式数据中的某个属性
watch(()=>person.job,(newValue,oldValue)=>{
	console.log('person的job变化了',newValue,oldValue)
},{immediate:true,deep:true}) 

//情况五：监视reactive定义的响应式数据中的某些属性
watch([()=>person.job,()=>person.name],(newValue,oldValue)=>{
	console.log('person的job变化了',newValue,oldValue)
},{immediate:true,deep:true})

//特殊情况
watch(()=>person.job,(newValue,oldValue)=>{
    console.log('person的job变化了',newValue,oldValue)
},{deep:true}) //此处由于监视的是reactive素定义的对象中的某个属性，所以deep配置有效
```

## watchEffect函数

和watch函数很像，但不用指明要监视哪个属性，监视的回调中用到哪个属性，就自动监视哪个属性。

```js
//watchEffect所指定的回调中用到的数据只要发生变化，则直接重新执行回调。
watchEffect(()=>{
    const x1 = sum.value
    const x2 = person.age
    console.log('watchEffect配置的回调执行了')
})
```

## watchEffect与computed的区别

computed注重的计算出来的值（回调函数的返回值），所以必须要写返回值。而watchEffect更注重的是逻辑过程（回调函数的函数体），所以不用写返回值。

## 生命周期

![实例的生命周期](lifecycle.svg)

beforeDestroy和destroyed两个钩子变成了beforeUnmount和unmounted

同时Vue3也提供了Composition API形式的生命周期钩子，其对应关系如下：

![image-2022060632919611 PM](image-2022060632919611%20PM.png)

Composition API形式的生命周期钩子需要先引入后使用，写在setup中。而对应在 `beforeCreate` 和 `created` 中的代码都直接在 `setup` 函数中编写。

## 自定义hook函数

hook本质上是一个函数，把setup中的使用的一些Composition API进行了封装，然后组件由外部将hook引入。hook函数类似于Vue2的mixin，可以实现复用代码，让代码的逻辑更加清楚易懂。

## toRef

`toRef()`创建一个ref对象，其value值指向另一个对象中的某个属性。可以用在要将响应式对象中的某个属性单独提供给外部使用的情况。

```js
const name = toRef(person, 'name')
```

另外，`toRefs()`与`toRef()`功能相同，但用于批量创建多个ref对象。

```js
const p = toRefs(person)
```

