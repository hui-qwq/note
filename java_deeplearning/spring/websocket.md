# ServerEndpoint
是 Java WebSocket 中的一个注解，用来标识一个类是 WebSocket 服务端端点（Endpoint）。
它属于 Java EE / Jakarta WebSocket API，常用于实现实时通信，比如：

- 聊天系统
- 在线客服
- 实时通知
- 股票价格推送
- 游戏状态同步
- 弹幕系统

### 基本用法
```java
@ServerEndpoint("/chat")
public class ChatEndpoint {

    @OnOpen
    public void onOpen(Session session) {
        System.out.println("有客户端连接");
    }


    @OnMessage
    public void onMessage(String message, Session session) {
        System.out.println("收到消息：" + message);
    }


    @OnClose
    public void onClose() {
        System.out.println("连接关闭");
    }


    @OnError
    public void onError(Throwable error) {
        error.printStackTrace();
    }
}
```

### 四个核心注解

#### @OnOpen
连接建立时执行：
```java
@OnOpen
public void open(Session session){
}
```
```
客户端 ----建立WebSocket连接----> 服务端
                                  |
                                  ↓
                              @OnOpen
```

#### @OnMessage

收到消息时执行：
```java
@OnMessage
public void message(String msg){
}
```

#### @OnClose
连接断开：
```java
@OnClose
public void close(){
}
```
比如用户关闭网页。

#### @OnError
发生异常:
```java
@OnError
public void error(Throwable e){
}
```

maven坐标   
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```
