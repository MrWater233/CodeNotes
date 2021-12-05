# 一.后端

## 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

## WebSocket配置类

`com.learn.chat.config.WebSocketConfig`

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * @author WanJingmiao
 * @Description 启用由消息代理支持的WebSocket消息处理
 * @date 2020/5/1 11:05
 * @Version
 */
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    /**
     *  配置一个sockjs连接的端口，并配置跨域
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOrigins("*").withSockJS();
    }

    /**
     * 消息代理
     * enableSimpleBroker 服务端推送给客户端的路径前缀
     * setApplicationDestinationPrefixes  客户端发送数据给服务器端的一个前缀
     * @param registry
     */
    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        //开启基于内存的消息代理，并指定消息代理回调的地址
        registry.enableSimpleBroker("/topic");
    }
}
```

## 控制层

在接收用户的消息的基础上加个`hello`返回

`com.learn.chat.controller.ChatController`

```java
package com.learn.chat.controller;

import com.learn.chat.model.ChatMessage;
import com.learn.chat.model.Greeting;
import com.learn.chat.model.HelloMessage;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.messaging.simp.SimpMessageHeaderAccessor;
import org.springframework.stereotype.Controller;
import org.springframework.web.util.HtmlUtils;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/1 11:12
 * @Version
 */
@Controller
public class ChatController {
    /**
     * /app/hello请求被这个方法执行
     * 结果将返回到/topic/greetings这个broker中，前端订阅这个地址就可以接收消息
     * @param message
     * @return
     * @throws Exception
     */
    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
        Thread.sleep(1000); // simulated delay
        return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
    }

}
```

## 接收的实体类

`com.learn.chat.model.HelloMessage`

```java
package com.learn.chat.model;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/11 17:19
 * @Version
 */
@Data
public class HelloMessage {
    private String name;
}

```

`com.learn.chat.model.Greeting`

```java
package com.learn.chat.model;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/11 17:19
 * @Version
 */
@Data
public class HelloMessage {
    private String name;
}

```

# 二.前端

## 依赖

```shell
npm install sockjs-client stompjs --save
```

## 封装的stomp组件

`src/widget/stomp.js`

```js
import SockJs from "sockjs-client"
import Stomp from "stompjs"

let bConnected = false
let bFail = false

/**
 * 断开连接
 */
const Disconnect = function () {
  if (this.$stompClient && bConnected) {
    this.$stompClient.disconnect(() => {
      console.log("stomp disconnect from server");
    })
  }
}

let subscriberQueue = []
const Subscribe = function (strTopic, oCallback) {
  if (bFail) {
    throw "stomp connect to server failed"
  }
  if (bConnected) {
    this.$stompClient.subscribe(strTopic, oCallback)
  }
  else {
    subscriberQueue.push({
      topic: strTopic,
      callback: oCallback.bind(this)
    })
  }
}

const Send = function (...args) {
  if (this.$stompClient && bConnected) {
    if (args.length == 2) {
      this.$stompClient.send(args[0], {}, args[1])
    }
    else if (args.length == 3) {
      this.$stompClient.send(args[0], args[1], args[2])
    }
  }
}

const Init = function (Vue, options) {
  //init
  let m_Options = options || {}

  //初始化socket和stomp
  let socket = new SockJs(m_Options.endPoint)
  let stompClient = Stomp.over(socket)
  Vue.prototype.$stompClient = stompClient

  stompClient.connect({}, () => {
    bConnected = true
    console.log("success")
    if (subscriberQueue.length) {
      subscriberQueue.forEach((oSubscriber) => {
        stompClient.subscribe(oSubscriber.topic, oSubscriber.callback)
      })
      subscriberQueue = []
    }
  }, (error) => {
    bFail = true
    console.log("fail")
    console.error(error)
  })

  //disconnect
  Vue.prototype.$disconnect = Disconnect

  //subscribe
  Vue.prototype.$subscribe = Subscribe

  //send
  Vue.prototype.$send = Send
}

const install = function (Vue, options) {
  Init(Vue, options);
}

export default {install}
```

## 安装组件

`src/main.js`

```js
//stomp目录下是上面封装好的模块
import Stomp from "@/widget/stomp"

//stomp endPoint是后台开放的连接点 /*globalConfig.endPoint*/
Vue.use(Stomp, {endPoint: 'http://localhost:8888/ws/'})
```

## 页面

```vue
<template>
  <div id="app">
    <button @click="send">发送</button><br>
    {{message}}
  </div>
</template>

<script>
export default {
  data(){
    return{
      message: ''
    }
  },
  methods:{
    //向SpringBoot控制层配置的地址发送信息
    send(){
      this.$send('/app/hello',JSON.stringify({'name': '小明'}))
    }
  },
  //初始化的时候订阅一个broker，每次这个broker有消息message就会更新
  created() {
    this.$subscribe("/topic/greetings", message => {
      this.message = JSON.parse(message.body).content
    });
  }
};
</script>
```

