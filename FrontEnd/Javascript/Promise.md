# 一.介绍

> Promise是异步编程的解决方案，将异步请求和异步处理分离
>
> 一般是有异步操作时对异步操作进行封装

# 二.回调地狱

> 多层的嵌套回调，导致逻辑复杂，代码不好维护

```js
setTimeout(() => {
  console.log('hello world');
  setTimeout(() => {
    console.log('hello world');
      setTimeout(() => {
        console.log('hello world');
      }, 1000)
  }, 1000)
}, 1000)
```

# 三.Promise基本使用

```js
//参数->函数(resolve,reject)
//resolve和reject本身也是函数
new Promise((resolve, reject) => {
  //第一次网络请求的代码
  setTimeout(() => {
    //调用resolve()函数以后可以使用then来接收函数执行接下来的操作
    resolve('Hello Word');
  }, 1000)
}).then((data) => {
  //第一次拿到结果的处理代码
  console.log(data);

  return new Promise((resolve, reject) => {
    //第二次网络请求的代码
      setTimeout(() => {
        resolve(data)
          
        if(/*发生错误*/...){
          reject('error')   
        }
      }, 1000)
  })
}).then((data) => {
  //第二次拿到结果处理的代码
  console.log(data);
}).catch(error=>{
  //处理reject()函数
  console.lof(error);
})
```

- `resovle(data)`：传递参数并继续接下去的处理操作，通过`.then(data=>{})`接收参数并处理
- `reject(error)` ：传递异常信息并进行接下去处理，通过`.catch(error=>{})`接收信息并处理异常

- `catch和then`可以一起进行处理

  ```js
  .then(()=>{
    //成功操作
  },()=>{
    //失败操作
  })
  ```

# 四.Promise的链式处理

## 传递resolve的简写

- 按照基本使用的处理方法：

  ```js
  new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('hello');
    }, 1000)
  }).then(res => {
    //对结果第一次处理
    res = res + ' world';
  
    return new Promise((resolve) = {
        resolve(res);
    })
  }).then(res => {
    //对结果进行第二次处理
    console.log(res)
  })
  ```

- 简写

  ```js
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('hello');
    }, 1000)
  }).then(res => {
    //对结果第一次处理
    res = res + ' world';
  
    return Promise.resolve(res);
  }).then(res => {
    //对结果进行第二次处理
    console.log(res)
  })
  ```

- 精简

  ```js
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('hello');
    }, 1000)
  }).then(res => {
    //对结果第一次处理
    res = res + ' world';
  
    return res;
  }).then(res => {
    //对结果进行第二次处理
    console.log(res)
  })
  ```

## 传递reject的简写

- ```js
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('hello');
    }, 1000)
  }).then(res => {
    //对结果第一次处理
    res = res + ' world';
    if(/*发生错误*/)
      return Promise.reject('error');
    return res
  }).then(res => {
    //对结果进行第二次处理
    console.log(res)
  }).catch(err=>{
    console.log(err)
  })
  ```

- 通过throw

  ```js
  new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('hello');
    }, 1000)
  }).then(res => {
    //对结果第一次处理
    res = res + ' world';
    if(/*发生错误*/)
      throw 'error'
    return res
  }).then(res => {
    //对结果进行第二次处理
    console.log(res)
  }).catch(err=>{
    console.log(err)
  })
  ```

# 五.Promise.all

> 能够在多个网络请求都成功后再执行相关操作

```js
Promise.all([
  new Promise((resolve, reject) => {
      $ajax({
          url: 'url1',
          success: (data) => {
              resolve(data)
          }
      })
  }),
  new Promise((resolve, reject) => {
      $ajax({
          url: 'url2',
          success: (data) => {
              resolve(data)
          }
      })
  })
]).then(results => {
  //第一次请求结果
  console.log(results[0]);
  //第二次请求结果
  console.log(results[1]);
})
```

