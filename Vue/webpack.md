# 一.模块化

> 避免JavaScript文件变量重名并增加代码复用性

## 早期模块化

> ES6之前使用对象和匿名函数闭包进行模块化封装

- 模块导出

  ```javascript
  var ModuleA = (function(){
      var obj = {};
      obj.flag = true;
      obj.myFunc = function(info){
          console.log(info);
      }
      return obg;
  })()
  ```

- 模块使用

  ```javascript
  console.log(ModuleA.flag);
  ```

## CommonJS

- 模块导出

  ```javascript
  module.exports = {
      flag:true,
      test(a,b){
          return a + b;
      }
  }
  ```

- 模块导入

  ```javascript
  //对象解构写法
  let {flag,test} = require('模块路径.js')
  //等同于
  let _mA = require('模块路径.js');
  let flag = _mA.flag;
  let test = _mA.test;
  ```

## ES6

```html
<!--在html中导入脚本时设置类型为模块，这样各个js文件的变量就隔离了-->
<script src="模块路径.js" type="module"></script>
```

- 模块导出

  只有被导出的变量或者函数才能被其他文件使用

  - 方式一

    ```javascript
    let flag = true;
    function sum(a,b){
        return a + b;
    }
    export{
     flag,
     sum
    }
    ```

  - 方式二

    ```javascript
    export let flag = true;
    export function sum(a,b){
        return a + b;
    };
    //导出ES6的类，调用时直接new即可
    export class Person{
        run(){
        }
    }
    ```

  - 方式三

    ```javascript
    //在统一模块中export default只能有一个
    export default function(){
        console.log('默认导出');
    }
    //在模块导入时可以任意命名
    import myFunc from "模块路径.js"
    ```

- 模块导入

  - 方式一

    ```javascript
    import {flag,sum} from "模块路径.js"
    ```

  - 方式二

    ```javascript
    import * as test from "模块路径.js"
    //调用
    test.flag;
    ```


# 二.webpack基本使用

1. 初始化npm环境

   ```shell
   npm init
   ```

2. 本地安装webpack和webpack-cli的开发时依赖 

   ```shell
   npm install webpack webpack-cli --save-dev
   ```

3. 在`package.json`中添加使用`build`命令的映射

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200325151621.png)

4. 创建模块并导出

   `src/js/tools.js`

   ```javascript
   function sum(a, b) {
     return a + b;
   }
   //组件导出
   module.exports = {
     sum
   }
   ```

5. 创建入口文件并导入模块

   `src/main.js`

   ```javascript
   //导入组件，大括号不能省略
   const { sum } = require('./js/tools.js')
   
   console.log(sum(3, 4));
   ```

6. 配置`webpack.config.js`

   ```javascript
   //导入node内部模块，可以用来获取绝对路径
   const path = require('path');
   
   module.exports = {
     //指定入口文件
     entry: './src/main.js',
     //指定出口文件
     output: {
       path: path.resolve(__dirname, 'dist'),
       filename: 'bundle.js'
     }
   }
   ```

7. 使用打包命令打包js文件

   ```shell
   npm run build
   ```

8. 创建`index.html`页面并导入打包完成的js文件

   ```html
   <body>
     <script src="./dist/bundle.js"></script>
   </body>
   ```


# 三.loader加载器

> webpack对js之外的文件进行借在打包时需要安装对应的loader
>
> [loader](https://www.webpackjs.com/loaders/)

## CSS

1. 安装`css-loader`和`style-loader`

   <font color=blue>`css-loader`负责加载CSS文件，`style-loader`负责将样式添加到dom中</font>

   ```shell
   npm install css-loader style-loader --save-dev
   ```

2. 配置`webpack.config.js`

   增加一个`module`的`rule`规定匹配`.css`文件并设置该种文件的loader

   <font color=red>`use`中配置的`loader`从右向左加载</font>

   ```javascript
   module.exports = {
     entry: './src/main.js',
     output: {
       path: path.resolve(__dirname, 'dist'),
       filename: 'bundle.js'
     },
     module: {
       rules: [
         {
           test: /\.css$/,
           use: ['style-loader', 'css-loader']
         }
       ]
     }
   }
   ```

3. 编写CSS文件

   `src/css/normal.css`

   ```css
   body{
     background-color: red;
   }
   ```

4. 在入口文件中导入CSS模块

   `src/main.js`

   ```javascript
   require('./css/normal.css')
   ```

5. 打包

   ```shell
   npm run build
   ```

## less

1. 安装`less-loader`和`less`

   `less`：解析less

   `less-loader`：通过`less`加载less文件

   ```shell
   npm install --save-dev less-loader less
   ```

2. 配置`webpack.config.js`

   增加`module`

   ```javascript
   module.exports = {
       ...
       module: {
           rules: [{
               test: /\.less$/,
               use: [{
                   loader: "style-loader" // creates style nodes from JS strings
               }, {
                   loader: "css-loader" // translates CSS into CommonJS
               }, {
                   loader: "less-loader" // compiles Less to CSS
               }]
           }]
       }
   };
   ```

3. 编写less文件

   `src/css/special.less`

   ```less
   @fontSize: 50px;
   @fontColor: white;
   
   body{
       font-size: @fontSize;
       color: @fontColor;
   }
   ```

4. 入口导入less模块

   `src/main.js`

   ```javascript
   require('./css/special.less')
   ```

5. 打包

   ```shell
   npm run build
   ```

## 图片

1. 安装`url-loader`

   ```shell
   npm install --save-dev url-loader
   ```

2. 配置`webpack.config.js`

   `options`中的`limit`限制base64图片大小（单位：字节）

   ```javascript
   module.exports = {
     ...
     module: {
       rules: [
         {
           test: /\.(png|jpg|gif|jpeg)$/,
           use: [
             {
               loader: 'url-loader',
               options: {
                 //当图片大小小于limit时编译成base64字符串形式
                 limit: 8192,
                 //自定义打包的图片文件名：图片原名.8位哈希.扩展名
                 name: 'img/[name].[hash:8].[ext]'
               }
             }
           ]
         }
       ]
     }
   }
   ```

   当图片大于`limit`直接将图片进行打包，需要安装`file-loader`

   接着配置`webpack.config.js`

   ```javascript
   output: {
     path: path.resolve(__dirname, 'dist'),
     filename: 'bundle.js',
     //配置打包完成后图片路径，如果main.js在dist中就无需配置
     publicPath: 'dist/'
   },
   ```

3. 导入图片

   `src/img/demo2.jpg`

4. 配置CSS文件

   `src/css/normal.css`

   ```css
   body{
     background: url("../img/demo2.jpg");
   }
   ```

5. 导入CSS模块

   ```javascript
   require('./css/normal.css')
   ```

6. 打包

   ```shell
   npm run build
   ```

# 四.babel

> 将原来代码中的ES6语法在打包的时候转化成ES5，使js代码更加通用
>
> [安装方式](https://www.webpackjs.com/loaders/babel-loader/)

1. 安装

   ```shell
   npm install babel-loader@7 babel-core babel-preset-es2015 --save-dev
   ```

2. 配置`webpack.config.js`

   ```javascript
   module.exports = {
     ...
     module: {
       rules: [
         {
           test: /\.js$/,
           exclude: /(node_modules|bower_components)/,
           use: {
             loader: 'babel-loader',
             options: {
               presets: ['es2015']
             }
           }
         }
       ]
     }
   }
   ```

# 五.配置Vue

## 基础配置

1. 安装Vue

   ```shell
   npm install vue --save
   ```

2. 配置`webpack.config.js`

   默认使用`runtime-only`：代码中不能有`template`

   配置以后使用`runtime-compiler`：代码中可以有`template`，因为compiler可以用于编译`template`

   ```javascript
   module.exports = {  
     ...
     resolve: {
       alias: {
         'vue$': 'vue/dist/vue.esm.js'
       }
     }
   }
   ```

3. 编写`main.js`

   ```javascript
   //导入Vue
   import Vue from 'vue';
   
   new Vue({
     el: '#app',
     data: {
       message: 'Hello webpack'
     }
   });
   ```

4. 编写`index.html`

   ```html
   <div id="app">{{message}}</div>
   ```

5. 打包

   ```shell
   npm run build
   ```

## 组件化

> 在实际的开发项目中`index.html`一般不做修改，需要在组件中进行编写
>
> 如果一个Vue实例有template属性，Vue将template的内容替换el在`index.html`指定的div中

- `main.js`

  - 原型：

    ```javascript
    import Vue from 'vue';
    
    const app = new Vue({
      el: '#app',
      template: `
      <div>
        {{message}}
      </div>
      `,
      data: {
        message: 'Hello webpack'
      }
    });
    ```

  - 组件化：

    - 抽取组件

      `src/vue/cpn.js`

      ```javascript
      //编写并导出组件
      export default {
        template: `
          <div>
            {{message}}
          </div>
          `,
        data() {
          return {
            message: 'Hello webpack'
          }
        }
      }
      ```

    - 将组件的引用放在了Vue的template中

      `main.js`

      ```javascript
      import Vue from 'vue';
      import cpn from './vue/cpn.js'
      
      new Vue({
        el: '#app',
        //导入组件
        template: '<cpn/>',
        components: {
          cpn
        }
      });
      ```

  - 实现Vue组件化：

    - 安装loader，实现对`.vue`文件的加载和编译

      ```shell
      npm install vue-loader vue-template-compiler --save-dev
      ```

    - 配置`webpack.config.js`

      [官网配置](https://vue-loader.vuejs.org/zh/guide/#vue-cli)

      ```javascript
      const VueLoaderPlugin = require('vue-loader/lib/plugin')
      
      module.exports = {
        ...
        module: {
          rules: [
            {
              test: /\.vue$/,
              use: ['vue-loader']
            }
          ]
        },
        resolve:{
          //import时可以省略.vue
          extensions:['.vue'],
          //别名，使用runtime-compiler
          alias:{
            'vue$': 'vue/dist/vue.esm.js'
          }
        },
        plugins: [
          // 请确保引入这个插件！
          new VueLoaderPlugin()
        ]
      }
      ```

    - 编写Vue风格的组件

      `src/vue/cpn.vue`

      ```vue
      <template>
        <div>{{message}}</div>
      </template>
      
      <script>
      export default {
        data() {
          return {
            message: "Hello webpack"
          };
        }
      };
      </script>
      
      <style scoped>
      </style>
      ```

    - `main.js`

      ```javascript
      import Vue from 'vue';
      import cpn from './vue/cpn.vue'
      
      new Vue({
        el: '#app',
        template: '<cpn/>',
        components: {
          cpn
        }
      });
      ```

- `index.html`

  ```html
  <div id="app"></div>
  ```

# 六.plugin扩展器

> plugin可以对webpack进行扩充，如打包优化或者增加版权声明等

- 版权申明插件：

  `webpack.config.js`

  ```javascript
  const webpack = require('webpack')
  
  module.exports = {
    ...
    plugins: [
      new webpack.BannerPlugin('最终版权归小明所有')
    ]
  }
  ```

- 打包HTML插件

  将`index.html`打包到dist文件夹中，并且自动加入打包完成的js文件

  1. 安装插件

     ```shell
     npm install html-webpack-plugin --save-dev
     ```

  2. 修改`index.html`，去掉js引入，将他作为打包生成模板

     ```html
     <!DOCTYPE html>
     <html lang="en">
     
     <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Document</title>
     </head>
     
     <body>
       <div id="app"></div>
       <!--<script src="./dist/bundle.js"></script>-->
     </body>
     
     </html>
     ```

  3. 配置插件并指定模板

     `webpack.config.js`

     ```javascript
     const HtmlWebpackPlugin = require('html-webpack-plugin');
     
     module.exports = {
       ...
       //切记注释掉之前output中的publicPath: 'dist/'
       plugins: [
         new HtmlWebpackPlugin({
           template: 'index.html'
         })
       ]
     }
     ```

- js压缩插件

  1. 安装插件

     ```shell
     npm install uglifyjs-webpack-plugin --save-dev
     ```

  2. `webpack.config.js`

     ```javascript
     const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
     
     module.exports = {
       ...
       plugins: [
         new UglifyJsPlugin()
       ]
     }
     ```

# 七.搭建本地服务器

1. 安装

   ```shell
   npm install webpack-dev-server --save-dev
   ```

2. `webpack.config.js`

   ```javascript
   module.exports = {
     ...
     devServer: {
       contentBase: './dist',
       inline: true
     }
   }
   ```

   `devServer`的属性：

   - contentBase：为哪一个文件夹提供本地服务

   - port：端口号
   - inline：页面实时刷新
   - historyApiFallback：在SPA页面中，依赖HTML5的history模式

3. 编写`package.json`中的scripts

   ```json
   "dev": "webpack-dev-server"
   ```

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200326203410.png)

4. 启动服务器

   ```shell
   npm run dev
   ```

5. 服务器中的项目编译全部在内存中

   当项目编写完成后需要进行打包发布

   ```shell
   npm run build
   ```

# 八.webpack配置文件分离

> 分离开发和发布的配置

1. 安装webpack-merge

   ```shell
   npm install webpack-merge --save-dev
   ```

2. 创建公共配置文件，抽取公共配置（build和dist同级目录）

   `build/base.config.js`

   并修改导出路径

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200326210046.png)

3. 创建开发配置文件，抽取除了公共配置外开发需要的配置

   `build/dev.config.js`

   ```javascript
   const WebpackMerge = require('webpack-merge')
   const baseConfig = require('./base.config')
   
   module.exports = WebpackMerge(baseConfig, {
     devServer: {
       contentBase: './dist',
       inline: true
     }
   })
   ```

4. 配置发布配置文件，抽取除了公共配置外生产需要的配置

   `build/dev.config.js`

   ```javascript
   const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
   const WebpackMerge = require('webpack-merge')
   const baseConfig = require('./base.config')
   
   module.exports = WebpackMerge(baseConfig, {
     plugins: [
       new UglifyJsPlugin()
     ]
   })
   ```

5. 指定不同环境配置文件位置

   `package.json`

   ![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200326205516.png)

