# 一.环境搭建

- 创建脚手架项目

  ```shell
  vue create name
  ```

- 接着自定义配置

- 进入项目文件夹以后，启动项目

  ```shell
  npm run serve
  ```

--------

当从`git`上下载的项目搭建的时候，不会有`node_modules`文件夹，需要通过以下命令下载模块

```shell
npm install
```

# 二.Vue组件基本结构

```vue
<template>
  <!--HTML结构；只能有一个根标签-->
  <div class="hello">
    <h1>{{ msg }}</h1>
  </div>
</template>

<script>
//js逻辑部分
export default {
  name: "HelloWorld",
  props: {
    msg: String
  }
};
</script>

<style scoped>
/* CSS 样式 */
</style>
```

# 三.组件引入

## 全局组件

1. <a name='component'>编写组件</a>

   `vuecli-demo\src\components\Users.vue`

   ```vue
   <template>
     <div id="user">User</div>
   </template>
   
   <script>
   export default {};
   </script>
   
   <style></style>
   ```

2. 注册全局组件

   `vuecli-demo\src\main.js`

   ```javascript
   import Vue from 'vue'
   import App from './App.vue'
   //1.引入组件
   import User from './components/Users.vue'
   
   //2.注册全局组件
   Vue.component("user", User);
   Vue.config.productionTip = false
   
   new Vue({
     render: h => h(App),
   }).$mount('#app')
   ```

3. 引入组件

   `vuecli-demo\src\App.vue`

   ```vue
   <template>
     <div id="app">
       <user />
     </div>
   </template>
   
   <script>
   export default {
     name: "app",
     components: {}
   };
   </script>
   
   <style>
   </style>
   ```

## 局部组件

1. <a href="component">编写组件</a>

2. 注册并引入局部组件

   `vuecli-demo\src\App.vue`

   ```vue
   <template>
     <div id="app">
       <!--调用组件-->
       <User />
     </div>
   </template>
   
   <script>
   //引入组件
   import User from "./components/Users";
   export default {
     name: "app",
     components: {
       //注册组件，也可以使用user:User的写法
       User
     }
   };
   </script>
   
   <style>
   </style>
   ```

# 四.组件CSS样式

组件CSS如果不设置隔离，样式之间会相互影响

隔离方法：

```html
<style scoped></style>
```

# 五. 属性传值

> 父组件将值传给子组件

## 使用方法

父组件：

```vue
<template>
  <div id="app">
    <!--通过属性向子组件传入值-->
    <User :users="users" />
  </div>
</template>

<script>
import User from "./components/Users";
export default {
  name: "app",
  data() {
    return {
      users: [
        { name: "小明", age: 18, show: false },
        { name: "小明", age: 18, show: false },
        { name: "小明", age: 18, show: false },
        { name: "小明", age: 18, show: false },
        { name: "小明", age: 18, show: false },
        { name: "小明", age: 18, show: false }
      ]
    };
  },
  components: {
    User
  }
};
</script>
```

子组件：

```html
<template>
  <div id="user">
    <ul>
      <li @click="user.show = !user.show" v-for="(user,index) in users" :key="index">
        {{user.name}}-
        <span v-show="user.show">{{user.age}}</span>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  //通过属性值接收父组件传入的值
  //如果不子当以可以写为props: ["users","",...]
  props: {
      //传入值的属性名
      users: {
          //传入值的类型
          type: Array,
          //是否调用一定要传值
          required: true
      }
  }
};
</script>
```

## 传值和传引用

- 传引用：
  - 引用：对象，字符串等
  - 效果：子组件和父组件共享引用，当子组件改变传入的值时父组件的值也会发生变化
- 传值：
  - 值：String，Boolean等类型
  - 效果：子组件独立拥有值，不会对父组件的值造成影响

## 注册事件

> 子组件去修改父组件值时使用

子组件：

```javascript
methods:{
	change(){
		//注册事件 参数1：事件名称 参数2...：值
        this.$emit("titileChange","hello");
	}
}
```

父组件：

```vue
<template>
  <div id="app">
    <Header />
    <!--向子组件传入值-->
    <User @titleChange="updateTitle" :title="title" />
    <Footer />
  </div>
</template>

<script>
    data(){
        return{
            title:"hi"
        }
    }
	methods:{
        updateTitle(updatedTitle){
            this.title = updatedTitle
        }
    }
</script>
```

# 六.内置组件

## Slot插槽

> 占位符，父组件可以向子组件传入标签并在占位符处显示
>
> 注意：CSS样式在使用Slot时既可以定义在父组件上，也可以定义在子组件上

父组件：

```vue
<template>
  <div id="app">
    <FormHelper>
      //通过slot属性指定slot名字
      <h2 slot="title">{{title}}</h2>
      <p slot="text">这是一个文本</p>
      <p>公共slot</p>
    </FormHelper>
  </div>
</template>

<script>
import FormHelper from "./components/FormHelper";
export default {
  name: "app",
  data() {
    return {
      title: "这是一个title"
    };
  },
  components: {
    FormHelper
  }
};
</script>
```

子组件

```vue
<template>
  <div>
    //通过传入slot名字来调用
    <slot name="title"></slot>
    <h1>hello world</h1>
    <slot name="text"></slot>
    //调用所有没有名字的slot
    <slot></slot>
  </div>
</template>
```

## Component

> 动态引入组件
>
> 做到和直接调用组件（<组件名/>）不同的动态引入

父组件：

```vue
<template>
  <div id="app">
    <component :is="component"></component>
    <button @click="compoent = 'form-one'">
        show form one
    </button>
    <button @click="compoent = 'form-two'">
        show form two
    </button>
  </div>
</template>

<script>
import FormOne from "./components/FormOne";
import FormTwo from "./components/FormTwo";
export default {
  name: "app",
  data() {
      return{
          compoent = "form-one"
      }
  },
  components: {
      "form-one": FormOne,
      "form-two": FormTwo
  }
};
</script>
```

## keep-alive

> 将组件进行缓存
>
> 能够将上次组件的内容保存，也减少了调用，避免了性能消耗

```vue
<template>
  <div id="app">
    <keep-alive>
      <component :is="component"></component>
    </keep-alive>
    <button @click="compoent = 'form-one'">
        show form one
    </button>
    <button @click="compoent = 'form-two'">
        show form two
    </button>
  </div>
</template>

<script>
import FormOne from "./components/FormOne";
import FormTwo from "./components/FormTwo";
export default {
  name: "app",
  data() {
      return{
          compoent = "form-one"
      }
  },
  components: {
      "form-one": FormOne,
      "form-two": FormTwo
  }
};
</script>
```

