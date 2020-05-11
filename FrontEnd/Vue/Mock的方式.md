1. 本地模拟JSON数据

   1. 创建模拟数据

      `/public/mock/user/login.json`

      ```json
      {
        "status": 0,
        "data": {
          "id": 12,
          "username": "admin",
          "email": "admin@51purse.com",
          "phone": null,
          "role": 0,
          "createTime": 1479048325000,
          "updateTime": 1479048325000
        }
      }
      ```

   2. 使用

      ```js
      //安装并配置了axios和vue-axios
      this.axios.get('/mock/user/login.json').then((res)=>{
        this.res = res;
      })
      ```

2. 使用easy-mock

   1. 在[easy-mock](https://www.easy-mock.com/)上创建接口

   2. 配置URL

      `/src/main.js`

      ```js
      axios.defaults.baseURL = 'easy-mock提供的url';
      ```

   3. 使用

      ```js
      //安装并配置了axios和vue-axios
      this.axios.get('xxxx').then((res)=>{
        this.res = res;
      })
      ```
   
3. 本机集成[mock.js](http://mockjs.com/)进行数据mock
   
   1. 安装
   
      ```shell
      npm install mockjs --save
      ```
   
   2. `/src/mock/index.js`
   
      ```js
      import Mock from 'mockjs'
      
      Mock.mock('api/user/login',{
        "status": 0,
        "data": {
          "id": 12,
          "username": "admin",
          "email": "admin@51purse.com",
          "phone": null,
          "role": 0,
          "createTime": 1479048325000,
          "updateTime": 1479048325000
        }
      });
      ```
   
   3. `/src/main.js`
   
      ```js
      import axios from 'axios';
      import VueAxios from 'vue-axios'
      
      //mock开关
      const mock = true;
      if(mock){
        //执行到这个语句才进行模块导入
        require('./mock/api');
      }
      axios.defaults.baseURL = '/api';
      
      Vue.use(VueAxios, axios);
      ```
   
   4. 使用
   
      ```js
      this.axios.get('/user/login').then((res)=>{
          this.res = res;
      })
      ```
   
      
   
   

