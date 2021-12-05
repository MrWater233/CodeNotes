# 一.Composition API

## 1.1 setup

### 组件

#### 定义组件名

```vue
<script lang="ts">
export default { name: 'CustomComponentsName' }
</script>
```

#### 普通组件的使用

直接引入组件就能在模版中使用，无需注册

```ts
<script setup lang='ts'>
import SaySomething from "./Components/SaySomething.vue";
</script>
```

#### 动态组件

动态组件仍然是使用 `is`

```vue
<script setup lang='ts'>
import { ref } from "vue";
import Bar from "./Components/Bar.vue";
import Foo from "./Components/Foo.vue";

const condition = ref(false);
setTimeout(() => condition.value = true, 2000);
</script>

<template>
  <component :is="condition ? Bar : Foo"/>
</template>
```

### props

#### 运行时声明和类型声明

- 运行时声明

  - 定义：运行时声明是一种在运行中才会生效的一种声明

    行时声明指对于 `props` 的类型的声明，这种声明方式 IED 是无法检测和给出提示的，只有在运行后才会给出提示

  - 使用：

    ```vue
    <script setup lang='ts'>
    const props = defineProps({
    foo: String,
    bar: {
      type: Number,
      required: true
    }
    })
    </script>
    ```

- 类型声明

  - 定义：在这里类型声明指基于 ts 的类型检查，对 `props` 进行类型的约束，因此，要使用类型声明，需要基于 ts，即 `<script setup lang="ts">`

  - 使用：

    `?`表示该参数可传可不传

    ```vue
    <script setup lang='ts'>
    const props = defineProps<{
      foo?: string
      bar: number
    }>()
    </script>
    ```

  - 引入interface接口

    接收父组件传入的一个对象数组并指定数组内部对象的属性类型

    ```vue
    <script setup lang="ts">
    // 暂不支持引入，因为 setup 语法糖会将 List 编译成一个变量，因此只能在文件内写
    // import { List } from "./type";
    interface List {
      id: number,
      content: string,
      isDone: boolean,
    };
    
    const props = defineProps<{
      title: string,
      list: List[],		// ts 接口
    }>();
    </script>
    ```

    引入外部接口：

    ```ts
    // src/types/list.ts
    
    declare namespace List {
      export interface Basic {
        id: number,
        content: string,
        isDone: boolean,
      }
    }
    ```

    ```vue
    import { List } from './types'
    
    const props = defineProps<{
      title: string,
      list: List.Basic[],
    }>();
    ```

比较：

| 类型       | 优势                                                         | 劣势                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 运行时声明 | 不使用 ts 的情况下能够对 props 进行一定的、运行时的类型校验  | 1. 运行时校验 2. 只能进行基本类型的校验 3. 编码时无任何提示  |
| 类型声明   | 完美的支持类型的校验，包括props 的完美类型约束、父组件在传 props 时的提示以及子组件在使用 props 的提示 | 目前 ts 的接口暂时只支持写在文件内，未来应该会实现可从外部导入的，但目前可通过ts自动扫描types来解决 |

#### props 的默认值 —— widthDefaults

基本用法：

```vue
<script setup lang="ts">
const props = withDefaults(defineProps<{
  title?: string,
  list?: List.Basic[],
}>(), {
  title: 'Hello withDefaults',
  list: () => [{ id: 3, content: '3', isDone: false }],
});
</script>
```

### context

#### emit

在 `<script setup>` 中声明 `emit` ，必须使用 `defineEmits` API。同样可采用运行时声明和类型声明式，在类型声明下 `emit` 将具备完美的类型推断。

- 运行时声明

  ```vue
  <script setup lang="ts">
  // 这样是没有任何的类型检查的
  const emit = defineEmits(['handleClick', 'handleChange']);
  
  const handleClick = () => emit('handleClick', Date.now()+'');
  const handleChange = () => emit('handleChange', Date.now());
  </script>
  ```

- 类型声明式

  ```vue
  <script setup lang="ts">
  interface Click {
    id: string,
    val: number,
  }
  // 完美的类型检查
  // List.Basic 是基于 ts 自动扫描 types 文件夹以及 delcare namespace 自动导入的
  const emit = defineEmits<{
    (e: 'handleClickWithTypeDeclaration', data: Click): void,
    (e: 'handleChangeWithTypeDeclaration', data: List.Basic): void,
  }>();
  
  const handleClickWithTypeDeclaration = () => emit('handleClickWithTypeDeclaration', { id: '1', val: Date.now() });
  const handleChangeWithTypeDeclaration = () => emit('handleChangeWithTypeDeclaration', {
    id: 1,
    content: 'change',
    isDone: false,
  });
  </script>
  ```

#### expose

能够打破组件之间的封装，将组件内部的元素共享出去。默认是关闭的

```vue
// 子组件

<script setup>
import { ref } from 'vue'

const msg = 'Hello World'
const a = ref(2)

defineExpose({
  msg,
  a
})
</script>
```

```vue
// 父组件

#父组件代码片段
<Index ref="I"></Index>

<script setup>
    import Index from "./index.vue";

    const I = ref(null);
    
    function test() {
        console.log(I.value.msg) // Hello World
    }

</script>
```

#### useSlots和useAttrs

在 `<script setup>` 使用 `slots` 和 `attrs` 的情况应该是很罕见的，因为可以在模板中直接可通过 `$slots` 和 `$attrs` 来访问它们。

在那些罕见的需要使用它们的场景中，可以分别用 `useSlots` 和 `useAttrs` 两个函数来获取到对应的信息：

```vue
<script setup lang="ts">
import { useSlots, useAttrs } from "vue";

const slot = useSlots();
console.log('TestUseSlots', slot.header && slot.header());		// 获取到使用插槽的具体信息
  
const attrs = useAttrs();
console.log('TestUseAttrs', attrs);		// 获取到使用组件时传递的 attributes
</script>

<template>
  <h1> Here is slots test!!</h1>
  <slot name="header"></slot>
</template>
```

## 1.2 响应式

- `reactive`

  可以将一个Object类型的数据转换成响应式数据

  特点：

  1. 直接取属性值，属性值不是响应式的
  2. 只能接受Object类型的参数

  ```js
  import { reactive } from 'vue'
  
  // 响应式状态
  const state = reactive({
    count: 0
  })
  
  // 打印count的值
  console.log(state.count)
  ```

- `ref`

  可以在任何地方将变量变成响应式

  特点：

  1. 在JS中获取和设置时都要通过变量的`value`属性，而在`template`中则直接使用变量

     ```js
     import { ref } from 'vue'
     
     const counter = ref(0)
     
     console.log(counter) // { value: 0 }
     console.log(counter.value) // 0
     
     counter.value++
     console.log(counter.value) // 1
     ```

  2. 可以接受任何类型的参数。

     如果原始变量target为普通变量，value = target

     如果target为对象，value为Proxy(target)

     ```js
     const ref1 = ref({name: "小明", tel: 123456})
     const ref2 = ref(0)
     ```

  原理：

  `ref`不管普通变量还是对象都将它封装到一个对象中，目的是保证不同数据行为的行为统一，因为在 JavaScript 中，`Number` 或 `String` 等基本类型是通过值而非引用传递的：

  ![](https://gitee.com/mrwater233/PictureHost/raw/master/2021/202111192239558.gif)

- `toRef`

  `toRef(target, property)`将响应式对象的某个属性变为响应式

  1. target为一个响应式对象
  2. 返回一个包含value的`ObjectRefImpl`对象，value为`target[property]`的引用

  ```js
  const state = reactive({
    foo: 1,
    bar: 2
  })
  
  // state.foo 类型为 number，不具有响应式的功能
  // 通过toRef可以将其变为响应式
  const fooRef = toRef(state, 'foo')
  
  fooRef.value++
  console.log(state.foo) // 2
  
  state.foo++
  console.log(fooRef.value) // 3
  ```

## 1.3 watch

`watch`能够监听响应式对象的更改

```js
interface Data {
  counter: number
  doubleCounter: number
}

const data: Data = reactive({
  counter: 1,
  doubleCounter: computed(() => data.counter * 2)
})

watch(toRef(data, "counter"), (newValue, oldValue) => {
  console.log(newValue)
})


const count = ref(0)

watch(count, (newValue, oldValue) => {
  console.log(newValue)
})
```

## 1.4 计算属性

```js
import { ref, computed } from 'vue'

const counter = ref(0)
const twiceTheCounter = computed(() => counter.value * 2)

counter.value++
console.log(counter.value) // 1
console.log(twiceTheCounter.value) // 2
```

这里 `computed` 函数传递了第一个参数，它是一个类似 getter 的回调函数，输出的是一个*只读*的**响应式引用**。为了访问新创建的计算变量的 **value**，我们需要像 `ref` 一样使用 `.value`

## 1.5 拆分

可以将不同的composition进行拆分

```ts
// src/composables/useCount.ts

import {ref} from "@vue/reactivity";
import {watch} from "vue";

export default function useCount() {
    const count = ref(8)

    function clickButton() {
        count.value++
    }

    watch(count, (newValue, oldValue) => {
        console.log(newValue)
    })

    return {
        count,
        clickButton
    }
}
```

并在需要的地方进行引入并使用

```vue
<script setup lang="ts">
	const {count, clickButton} = useCount()
</script>
```

## 1.6 style module

使用 `<script setup>` 时，可以使用 `useCssModule` API 获取到 css module 对象

```vue
<script setup lang="ts">
import { useCssModule } from "vue";
const css = useCssModule();
console.log(css);		// { blue: "_blue_13cse_5", red: "_red_13cse_2"}
</script>

<style module>
.red {
  color: red;
}
.blue {
  color: blue;
}
</style>
```

## 1.7 状态驱动的动态 CSS

单文件组件的 `<style>` 标签可以通过 `v-bind` 这一 CSS 函数将 CSS 的值关联到动态的组件状态上，**有了这一特性，可以将大量的动态样式通过状态来驱动了，而不是写动态的 calss 类名或者获取 dom 来动态设置了**。

```vue
<script setup lang="ts">
import { ref } from "vue";
const color = ref('red');

setTimeout(() => color.value = 'blue' , 2000);
</script>

<template>
  <p>hello</p>
</template>

<style scoped>
p {
  color: v-bind(color);
}
</style>
```

# 二.Web Components

https://arrow.blog.csdn.net/article/details/120371634

vue3.2提供了一种特性，让开发者可以完全使用 vue 单文件组件的写法来创建自定义元素

## defineCustomElement

通过`defineCustomElement`来自定义元素对象

```tsx
import { defineCustomElement } from 'vue'

const MyVueElement = defineCustomElement({
  // 普通的 vue 组件
  props: {
    msg: String,
    selected: Boolean,
    index: Number,
  },
  emits: {},
  template: `<div class="msg">{{ msg }}</div>`,

  styles: [`.msg { color: red; }`]
});

// 注册自定义元素.
// 注册后，页面上所有的 `<my-vue-element>` 标签都会更新
customElements.define('my-vue-element', MyVueElement);
```

注册完成后， `<my-vue-element />` 就可以在组件或者 `html` 文件中使用了

```html
<my-vue-element
  msg="This is a message"
  :selected="true"
  :index="1"
/>
```

# 参考资料

- [Vue3.2 新特性详解——＜script setup＞ 和 ＜style＞ v-bind](https://arrow.blog.csdn.net/article/details/120091329)