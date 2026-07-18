# HttpClient
HttpClient的范围/特性
1）HttpClient是一个基于HttpCore的客户端Http传输类库

2）Http基于传统的IO(阻塞)

HttpClient不是浏览器，它是一个客户端http协议传输类库。HttpClient被用来发送和接受Http消息。HttpClient不会处理http消息的内容，不会进行javascript解析，不会关心contenttype，如果没有明确设置，httpclient也不会对请求进行格式化、重定向url，或者其他任何和http消息传输相关的功能。


### 请求执行
HttpClient最基本的功能就是执行Http方法。一个Http方法的执行涉及到一个或者多个Http请求/Http响应的交互，通常这个过程都会自动被HttpClient处理，对用户透明。用户只需要提供Http请求对象，HttpClient就会将http请求发送给目标服务器，并且接收服务器的响应，如果http请求执行不成功，httpclient就会抛出异样。

```java
public class RequestExecution001 {
    public static void main(String[] args) throws ClientProtocolException, IOException {
        // ① 创建一个 HttpClient 对象
        // 相当于打开了一个没有界面的浏览器，后面所有 HTTP 请求都由它发送
        CloseableHttpClient httpClient = HttpClients.createDefault();
        // ② 创建一个 GET 请求对象
        // 相当于在浏览器地址栏输入：https://www.baidu.com/
        // 注意：这里只是创建请求，还没有真正发送
        HttpGet httpGet = new HttpGet("https://www.baidu.com/");
        // ③ 发送 GET 请求
        // execute() 会向百度服务器发送 HTTP 请求，并等待服务器返回响应(Response)
        // 返回的 response 中包含：
        //   1. 状态码（200、404、500...）
        //   2. 响应头(Header)
        //   3. 响应体(网页HTML、JSON等)
        CloseableHttpResponse response = httpClient.execute(httpGet);
        try {
            // ④ 获取所有响应头(Header)
            // getAllHeaders() 返回的是 Header[] 数组
            System.out.println(response.getAllHeaders());
        } finally {
            // ⑤ 关闭响应对象
            // response 内部持有网络连接，不关闭会浪费资源
            response.close();
        }
        // 关闭 HttpClient，释放资源
        httpClient.close();
    }
}
```

HttpClient提供URIBuilder工具类来简化URIs的创建和修改过程。
```java
URI uri = new URIBuilder()
        .setScheme("http")
        .setHost("www.baidu.com")
        .setPath("s")
        .setParameter("param1", "value1")
        .setParameter("param2", "value2")
        .build();

HttpGet httpGet = new HttpGet(uri);
System.out.println(httpGet.getURI());
// 结果是
// http://www.baidu.com/s?param1=value1&param2=value2
```

服务器收到客户端的http请求后，就会对其进行解析，然后把响应发给客户端，这个响应就是HTTPresponse

```java
        HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,HttpStatus.SC_OK,"OK");

        System.out.println(response.getProtocolVersion());

        System.out.println(response.getStatusLine().getStatusCode());

        System.out.println(response.getStatusLine().getReasonPhrase());

        System.out.println(response.getStatusLine().toString());


```