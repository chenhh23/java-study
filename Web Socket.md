### Reference
- [廖雪峰WebSocket](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001472780997905c8f293615c5a42eab058b6dc29936a5c000)
- [阮一峰WebSocket](http://www.ruanyifeng.com/blog/2017/05/websocket.html)
- [各语言Demo代码库](https://github.com/joewalnes/websocketd/tree/master/examples)

#### WebSocket协议
WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工(full-duplex)通信——允许服务器主动发送信息给客户端。Http协议是一种Request-Response的请求，请求只能够由Client发起，服务器进行处理。WebSocket补充了Http协议的被动性，先通过http协议握手建立连接，随后请求升级成websocket协议，在浏览器和服务器之间建立一个不受限的双向通信的通道，服务器可以在任意时刻发送消息给浏览器。

#### 工作原理
![image](https://github.com/chenhh23/java-study/blob/master/picture/web-socket.png)

因为websocket先通过http协议握手建立连接，请求协议是一个标准的HTTP请求，WebSocket连接必须由浏览器发起。

```
GET ws://localhost:3000/ws/chat HTTP/1.1
Host: localhost
Upgrade: websocket
Connection: Upgrade
Origin: http://localhost:3000
Sec-WebSocket-Key: client-random-string
Sec-WebSocket-Version: 13
```

该请求和普通的HTTP请求有几点不同：

- GET请求的地址不是类似/path/，而是以ws://开头的地址；
- 请求头Upgrade: websocket和Connection: Upgrade表示这个连接将要被转换为WebSocket连接；
- Sec-WebSocket-Key是用于标识这个连接，并非用于加密数据；
- Sec-WebSocket-Version指定了WebSocket的协议版本。

服务器如果接受该请求，就会返回如下响应

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: server-random-string
```
该响应代码101表示本次连接的HTTP协议即将被更改，更改后的协议就是Upgrade: websocket指定的WebSocket协议。

#### Difference
Long Poll 和 Ajax轮询实现的类似于websocket的功能，这两种方式，都是在不断地建立HTTP连接，然后等待服务端处理，可以体现HTTP协议的另外一个特点，被动性，服务端不能主动联系客户端，只能有客户端发起。没有其他技术能够像WebSocket一样提供真正的双向通信，许多web开发者仍然是依赖于ajax的长轮询来实现。

##### Ajax
Ajax轮询让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息，来获取浏览器的信息从而获取到最新的信息。对于Ajax轮询，服务器需要不断用新的线程去接受请求，轮询的效率低，非常浪费资源(因为必须不停连接，或者 HTTP 连接始终打开)。

##### Long Poll
Long Poll 其实原理跟 ajax轮询 差不多，都是采用轮询的方式，不过采取的是阻塞模型，也就是说，客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有消息才返回，返回完之后，客户端再次建立连接，周而复始。
Long Poll会直接占用服务器的线程等待服务器回复，如果访问量过多而服务器又不能及时的回复，就会导致请求堆积起来最后超过服务并发量从而拒绝新的请求。

#### Spring WebSocket

##### Method
websocket允许通过JavaScript建立与远程服务器的连接，从而实现客户端与服务器间双向的通信。在websocket中有两个方法：
1. send() 向远程服务器发送数据
2. close() 关闭该websocket链接

websocket同时还定义了几个监听函数　　　　
1. onOpen 当网络连接建立时触发该事件
2. onError 当网络发生错误时触发该事件
3. onClose 当websocket被关闭时触发该事件
4. onMessage 当websocket接收到服务器发来的消息的时触发的事件，也是通信中最重要的一个监听事件。

websocket还定义了一个readyState属性，这个属性可以返回websocket所处的状态：
1. CONNECTING(0) websocket正尝试与服务器建立连接
2. OPEN(1) websocket与服务器已经建立连接
3. CLOSING(2) websocket正在关闭与服务器的连接
4. CLOSED(3) websocket已经关闭了与服务器的连接

websocket的url开头是ws，如果需要ssl加密可以使用wss，当我们调用websocket的构造方法构建一个websocket对象（new WebSocket(url)）的之后，就可以进行即时通信了。

##### 服务端

```
public interface WebSocketHandler {
	void afterConnectionEstablished(WebSocketSession session) throws Exception;
	void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception;
	void handleTransportError(WebSocketSession session, Throwable exception) throws Exception; 
	void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception; 
	boolean supportsPartialMessages();
}
```
---

握手拦截器
```
public class WebSocketHandshakeInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
        if (request instanceof ServletServerHttpRequest) {
            attributes.put("username",userName);
        }
        return true;
    }
    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
    }
}

```

---

handler和拦截器注册
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer{
	@Autowired
	private ChatHandler chatHandler;
	@Autowired
	private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(chatHandler,"/chat").addInterceptors(webSocketHandshakeInterceptor);
	}
}
```

##### SockJs
SockJS 是在 WebSocket 之上的 API，应对许多浏览器不支持 WebSocket 协议
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer{
	@Autowired
	private ChatHandler chatHandler;
	@Autowired
	private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;
	@Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        //允许连接的域,只能以http或https开头
        String[] allowsOrigins = {"http://www.xxx.com"};
        
        //WebSocket通道,withSockJS()表示开启sockjs
        registry.addHandler(chatHandler,"/chat").setAllowedOrigins(allowsOrigins).addInterceptors(webSocketHandshakeInterceptor).withSockJS();
	}
}
```
setAllowedOrigins(String[] domains),允许指定的域名或 IP (含端口号)建立长连接，如果只允许自家域名访问，这里轻松设置。如果不限时使用”*”号，如果指定了域名，则必须要以 http 或 https 开头。
