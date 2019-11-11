---
title: 在线聊天室实现-使用WebSocket
date: 2019-11-09 14:44:10
tags: [java,WebSocket]
---

# 在线聊天室实现-使用WebSocket

[demo地址](https://github.com/AsummerCat/im-websocket-demo)

# 导入pom

```java
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        
             <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>
```

<!--more-->

# 开启socket支持

```
/**
 * 开启socket支持
 */
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```



# 开始创建服务端

```java
package com.linjingc.top.imwebsocketdemo.webService;

import com.alibaba.fastjson.JSON;
import netscape.javascript.JSObject;
import org.springframework.stereotype.Component;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;

@ServerEndpoint("/websocket/{username}")
@Component
public class WebScoketServer {

    private static Integer onlineNum = 0; //当前在线人数，线程必须设计成安全的
    private static CopyOnWriteArraySet<WebScoketServer> arraySet = new CopyOnWriteArraySet<WebScoketServer>(); //存放每一个客户的的WebScoketServer对象，线程安全
    private Session session;

    /**
     * 连接成功
     *
     * @param session 会话信息
     */
    @OnOpen
    public void onOpen(@PathParam("username") String username, Session session) {
        System.out.println(username);
        this.session = session;
        arraySet.add(this);
        this.addOnlineNum();
        System.out.println("有一个新连接加入，当前在线 " + this.getOnLineNum() + " 人");
    }

    /**
     * 连接关闭
     */
    @OnClose
    public void onClose() {
        arraySet.remove(this);
        this.subOnlineNum();
        System.out.println("有一个连接断开，当前在线 " + this.getOnLineNum() + " 人");
    }

    /**
     * 连接错误
     *
     * @param session
     * @param error
     */
    @OnError
    public void onError(Session session, Throwable error) {
        System.err.println("发生错误！");
        error.printStackTrace();
    }

    /**
     * 发送消息,不加注解，自己选择实现
     *
     * @param msg
     * @throws IOException
     */
    public void onSend(String msg) throws IOException {
        this.session.getBasicRemote().sendText(msg);
    }

    /**
     * 收到客户端消息回调方法
     *
     * @param session
     * @param msg
     */
    @OnMessage
    public void onMessage(Session session, String msg) {
        System.out.println("消息监控：" + msg);
        MessageDto messageDto = JSON.parseObject(msg, MessageDto.class);
        for (WebScoketServer webScoketServer : arraySet) {
            try {
                webScoketServer.onSend(msg);
            } catch (IOException e) {
                e.printStackTrace();
                continue;
            }
        }
    }

    /**
     * 增加一个在线人数
     */
    private synchronized void addOnlineNum() {
        onlineNum++;
    }

    /**
     * 减少一个在线人数
     */
    private synchronized void subOnlineNum() {
        onlineNum--;
    }

    private Integer getOnLineNum() {
        return onlineNum;
    }
}
```

# 消息层

```java
package com.linjingc.top.imwebsocketdemo.webService;

/**
 * 消息
 * @author cxc
 * @date 2019/11/10 20:48
 */
public class MessageDto {
    private String id;
    private String message;
    private String orgCode;
    private String departId;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getOrgCode() {
        return orgCode;
    }

    public void setOrgCode(String orgCode) {
        this.orgCode = orgCode;
    }

    public String getDepartId() {
        return departId;
    }

    public void setDepartId(String departId) {
        this.departId = departId;
    }
}

```



# html页面

```java
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="utf-8">
    <title>菜鸟教程(runoob.com)</title>

    <script type="text/javascript">
        var ws;
        function WebSocketTest()
        {
            if ("WebSocket" in window)
            {
                console.log("您的浏览器支持 WebSocket!");

                // 打开一个 web socket
                 ws = new WebSocket("ws://192.168.31.106:8080/websocket/123");

                ws.onopen = function()
                {
                    // Web Socket 已连接上，使用 send() 方法发送数据
                    var data={
                        "id": "小明",
                        "message": "1111",
                        "orgCode": "1111"
                    }
                    ws.send(JSON.stringify(data));
                    console.log("数据发送中..."+data);
                    // alert("数据发送中...");
                };

                ws.onmessage = function (evt)
                {
                    var received_msg = evt.data;
                    console.log("数据已接收..."+received_msg);
                    // alert("数据已接收...");
                };

                ws.onclose = function()
                {
                    // 关闭 websocket
                    console.log("连接已关闭...");
                };
            }

            else
            {
                // 浏览器不支持 WebSocket
                alert("您的浏览器不支持 WebSocket!");
            }
        }


        function sendMessage(){
            var data={
                "id": "小丑",
                "message": "222",
                "orgCode": "222"
            }
            ws.send(JSON.stringify(data));
            console.log("数据发送中..."+data);
        }
    </script>

</head>
<body>

<div id="sse">
    <a href="javascript:WebSocketTest()">运行 WebSocket</a>
    <a href="javascript:sendMessage()">发送消息</a>
</div>

</body>
</html>
```

