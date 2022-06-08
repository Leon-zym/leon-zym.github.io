---
layout: mypost
title: Vue3学习-其他CompositionAPI与其他改动
categories: [Vue3]
---

## shallowReactive与shallowRef

shallowReactive()相对于Reactive()来说，只处理最外层属性的响应式（浅响应式）。shallowRef()相对于Ref()来说，只处理基本数据类型的响应式，不处理对象类型。

作用在于节约资源，提高性能。

## readonly与shallowReadonly

readonly()让一个响应式的数据变为只读。shallowReadonly()只让一个响应式对象数据的最外层属性变为只读（浅只读）。

作用在于不希望数据被修改时。

## toRaw与markRaw

toRaw()让一个由reactive生成的响应式对象变为普通对象。用于读取一个响应式对象对应的普通对象，对这个普通对象的操作不会引起页面的更新。

markRaw()标记一个对象，使其永远不会再成为响应式对象。用于标记有些第三方类库不被设置为响应式，或当渲染有些不可变数据源的大列表时，跳过响应式转换，以提高性能。

## customRef

customRef()可以创建一个自定义的ref，并使用track()和trigger()对其依赖项的跟踪和更新触发进行显式控制。

案例：实现防抖效果

```js
 <template>
  <input type="text" v-model="keyword" />
  <h3>{{ keyword }}</h3>
</template>

<script>
import { ref, customRef } from 'vue'
export default {
  name: 'Demo',
  setup() {
    // let keyword = ref('hello') //使用Vue准备好的内置ref
    
    //自定义一个myRef
    function myRef(value, delay) {
      let timer
      //通过customRef去实现自定义
      return customRef((track, trigger) => {
        return {
          get() {
            track() //告诉Vue这个value值是需要被“追踪”的
            return value
          },
          set(newValue) {
            clearTimeout(timer)
            timer = setTimeout(() => {
              value = newValue
              trigger() //触发，告诉Vue去更新界面
            }, delay)
          },
        }
      })
    }
    
    let keyword = myRef('hello', 500) //使用自定义的ref
    
    return {
      keyword,
    }
  },
}
</script>

```

## provide与inject

配合使用可以实现祖代与后代之间的数据通信。祖代组件使用provide()来提供数据，后代组件使用inject()来接收数据。

![img](components_provide.png)

在祖代组件中：

```js
setup(){
  let car = reactive({name:'奔驰',price:'40万'})
  provide('car',car) //提供数据
}
```

在后代组件中：

```js
setup(){
  const car = inject('car')
  return {car}
}
```

## 响应式数据的判断

isRef()：检查一个值是否为ref对象。

isReactive()：检查一个值是否为由reactive()创建的响应式代理。

isReadonly()：检查一个对象是否为由readonly()创建的只读代理。

isProxy()：检查一个对象是否为由reactive()或readonly()创建的对象。

---



>以下是Vue3中的部分其他改动的地方，更完整的改动应当参考[Vue3官方文档](https://v3.cn.vuejs.org/guide/migration/introduction.html)

## teleport组件

teleport可以将html结构移动到指定位置。位置可以是DOM元素名，也可以是css选择器名。

```html
<teleport to="移动位置">
	<div v-if="isShow" class="mask">
		<div class="dialog">
			<h3>我是一个弹窗</h3>
			<button @click="isShow = false">关闭弹窗</button>
		</div>
	</div>
</teleport>
```

## suspense组件

suspense可以在等待异步组件的时候渲染一些额外的内容，让用户体验更好。

异步引入组件：

```js
import {defineAsyncComponent} from 'vue'
const Child = defineAsyncComponent(()=>import('./components/Child.vue'))
```

使用suspense包裹组件的html结构，并配置好default和fallback：

```html
<template>
	<div class="app">
		<h3>我是App组件</h3>
    
		<Suspense>
      <!-- 默认要被显示的组件结构 -->
			<template v-slot:default>
				<Child/>
			</template>
      <!-- 等待时额外渲染出来的结构 -->
			<template v-slot:fallback>
				<h3>加载中.....</h3>
			</template>
		</Suspense>
    
	</div>
</template>
```

## 全局API的转移

在Vue2中的全局API和配置等，Vue3都调整到应用实例app身上了。

| 2.x 全局API（Vue）       | 3.x 实例API (app)           |
| :----------------------- | --------------------------- |
| Vue.config.xxxx          | app.config.xxxx             |
| Vue.config.productionTip | 移除                        |
| Vue.component            | app.component               |
| Vue.directive            | app.directive               |
| Vue.mixin                | app.mixin                   |
| Vue.use                  | app.use                     |
| Vue.prototype            | app.config.globalProperties |

## data选项

Vue3中data选项应当始终被声明为一个函数了。

## 动画中过渡的类名

Vue3中将类名v-enter改为了v-enter-from，更加语义化。

```css
.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.v-leave-from,
.v-enter-to {
  opacity: 1;
}
```

## 移除keycode及native修饰符

Vue3不再支持keyCode作为v-on事件修饰符了，也不再支持config.keyCodes自定义按键别名了。

同时也不再支持native作为v-on事件修饰符了。在父组件中给子组件绑定事件，若为自定义事件则需在子组件中用emits接收，若为原生事件则不需要。

在父组件中给子组件绑定原生与自定义事件：

```html
<my-component
  :close="handleComponentEvent"
  :click="handleNativeClickEvent"
/>
```

在子组件中仅接收自定义事件：

```js
export default {
  emits: ['close']
}
```

## 移除filter过滤器

Vue3官网说：

>过滤器虽然这看起来很方便，但它需要一个自定义语法，打破大括号内表达式是 “只是 JavaScript” 的假设，这不仅有学习成本，而且有实现成本！建议用方法调用或计算属性去替换过滤器。