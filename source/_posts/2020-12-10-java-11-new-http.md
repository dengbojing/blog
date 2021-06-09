---
title: java_11_new_http
date: 2020-12-10 09:21:53
tags: [java,java11]
categories: java
---
`java.net.http`模块使用
<!--more-->


# 引言

写代码的时候难免会远程调用别人的api,之前用`httpclient`,在接着是`okhttp`,也都是跟着项目上用的,其实`java 9`就出现了一个http模块,只是当时是孵化版本,`java 11`正式推出了.

# 简介

## 主要类和接口
- 类
   + `java.net.http.HttpClient`
   + `java.net.http.HttpHeaders`
   + `java.net.http.HttpRequest`
   + `java.net.http.HttpRequest.BodyPublishers`
   + `java.net.http.HttpRequest.BodyHanler`
   + `java.net.http.HttpRequest.BodySubscribers`
- 接口
    + `java.net.http.HttpClient.Builder`
    + `java.net.http.HttpRequest.BodyPublisher`
    + `java.net.http.HttpRequest.Builder`
    + `java.net.http.HttpResponse<T>`
    + `java.net.http.HttpResponse.BodyHandler<T>`
    + `java.net.http.HttpResponse.BodySubscriber<T>`
    + `java.net.http.HttpResponse.PushPromiseHandler<T>`
    + `java.net.http.HttpResponse.ResponseInfo`
    + `java.net.http.WebSocket`
    + `java.net.http.WebSocket.Builder`
    + `java.net.http.WebSocket.Listener`




## 基本使用
1. `jdk 9`之后都是使用模块化组织代码,所以创建一个模块化的项目让后引入`java.net.http`模块. 
    ```java
    module com.dbj.httpClient{
        requires java.net.http
    }
    ```

2. 创建`httpClient`  
    > 使用`builder`模式创建对象, 基本上该包下面所有的对象都使用`builder`模式创建对象, 这么做的好处参见`effective java`一书
    ```java
    var httpClient = HttpClient.newBuilder()
                .authenticator(new BasicAuthenticator("user", "password"))
                //.authenticator(Authenticator.getDefault()
                .connectTimeout(Duration.ofSeconds(10))
                .cookieHandler(CookieHandler.getDefault())
                .executor(Executors.newFixedThreadPool(2))
                .followRedirects(HttpClient.Redirect.NEVER)
                .priority(1)
                .proxy(ProxySelector.getDefault())
                .sslContext(SSLContext.getDefault())
                .sslParameters(new SSLParameters())
                .version(HttpClient.Version.HTTP_2)
                .build();
    ```
    or  
    ```java
    var httpClient = HttpClient.newHttpClient();
    ```
    equivalent  
    ```java
    var httpCLient = HttpClient.newBuilder().build();
    ```
    - `httpClient` 类似`String` 设计模式是不变的,所以没有提供方法改变创建时候的参数.  
    - 如果使用`http2`创建链接,但是服务端不支持,那么会自动降级成为`http1.1`,如果没有指定,默认也是使用`http2`  
    - `excutor()` 在使用异步请求时候使用,默认是使用线程池技术
    - `connectionTimeout()` 默认没有超时时间  
    - `priority()` 优先级,范围[1-256],不在此范围会抛出异常  
    - `connectTimeout()` 链接超时设置,在设定的时间内没有连接上则抛出`HttpConnectTimeoutException`  
    - `executor()` 用于异步任务执行,如果未指定,则会为每个`HttpClient`实例创建一个.
    - `followRedirects()` 当服务器返回`30x`时,是否跳转,默认不跳转
    - `authenticator()` 验证参数,`Authenticator.getDefault()`获取当前验证规则,可以使用`BasicAuthenticator`来传递用户名密码,也可以继承`Authenticator`实现自己的验证规则.  
    - `proxy()` 是否使用代理. 

3. 创建`HttpRequest` 
    ```java
    var httpRequset = HttpRequest.newBuilder(URI.create(""))
                .header("Content-Type","application/json")
                .header("token","faeaafwefeawgaer")
                .timeout(Duration.of(10, ChronoUnit.SECONDS))
                .expectContinue(true)
                .POST(HttpRequest.BodyPublishers.ofString(""))
                .version(HttpClient.Version.HTTP_2)
                .build();
    ```
    - `uri()` 可以在`newBuidler()` 中指定请求地址,也可以调用`uri()`方法指定请求地址.两者效果是一样的  
    - `header()` 效果与`setHeader()`相同,另有`headers()`批量设置请求头, 请求头键值对必须严格按照[RFC7230-section-3.2](https://tools.ietf.org/html/rfc7230#section-3.2)约定,否则抛出异常.
    - `timeout()` 请求超时时间设置,超过设定时间未收到响应则抛出异常,如不限制会永远阻塞(等待)
    - `POST()` `GET()` `DELETE()` `PUT()` 请求方法,或者使用`mehtod()`设置请求方法  
       使用前后端分离时候往往前端会发送一次`options`请求来判断后端是否支持跨域,此时就可以使用`method("OPTIONS",BodyPublishers.noBody())`   
       `BodyPublishers` 用于构建`BodyPublisher`的工具类,包含了一系列实用的构建请求体的方法,其中`BodyPublishers` 主要是调用`RequestPublishers` 来完成创建,`RequestPublishers` 中包含了很多`BodyPublisher`接口的实现
4. `HttpResponse`  
    同步请求
    ```java
    var httpResponse = httpClient.send(requset,BodyHandlers.ofString());
    ```
    异步请求  
    ```java
    var httpResponse = httpClient.sendAsync(httpRequest, HttpResponse.BodyHandlers.ofString())
                .thenApply(stringHttpResponse -> {
                    System.out.println(stringHttpResponse.statusCode());
                    return stringHttpResponse;
                })
                .thenApply(HttpResponse::body)
                .thenAccept(System.out::println);
    ```
    or 批量请求接口
    ```java
     var client = HttpClient.newHttpClient();

    List<HttpRequest> requests = paths.stream()
        .map(path -> "https://localhost:8443" + path)
        .map(URI::create)
        .map(uri -> HttpRequest.newBuilder(uri).build())
        .collect(Collectors.toList());
        
    
    CompletableFuture<?>[] responses = requests.stream()
        .map(request -> client.sendAsync(request, BodyHandlers.ofString())
            .thenApply(HttpResponse::body)
            .exceptionally(e -> "Error: " + e.getMessage())
            .thenAccept(System.out::println))
        .toArray(CompletableFuture<?>[]::new);
    ```
    - 异步请求返回一个`CompletableFuture<HttpResponse<T>>`,当有响应返回时,该对象后续回调将会被调用.异步请求使用创建`httpClient`时指定的`executor`来执行异步请求.
    - `HttpResponse`为一个接口, 不能直接创建, 所有实例都是`httpClient`请求返回, 接口提供方法如下:

    |返回值|方法|描述|
    |--|--|--|
    |T|`body()`|返回响应体|
    |HttpHeaders|`headers()`|返回响应投|
    |int|`statusCode()`|返回的状态码|
    |HttpRequset|`request()`|返回对应的请求体|
    |URI|`uri()`|返回请求地址|
    |HttpClient.Version|`version()`|返回http请求协议版本|

    -  `BodyHandlers` 用于构建`BodyHandler`的工厂类.

## 进阶使用

1. JSON请求  

    发送请求时秩序指定`Content-Type`为`application/json`, 然后将对象转换为`json`字符串
    ```java

    var httpRequest = HttpRequest.newBuilder(URI.create(""))
                    .header("content-type","application/json")
                    .GET()
                    .build();
    ```

    接受响应时,自定义`BodyHandler`将返回的`json`字符串转换为对象
    ```java
    public class JsonHandler<T> implements HttpResponse.BodyHandler<T> {

        private final Class<T> type;
        private final Gson gson;

        public JsonHandler(Class<T> type, Gson gson) {
            this.type = type;
            this.gson = gson;
        }

        @Override
        public HttpResponse.BodySubscriber<T> apply(HttpResponse.ResponseInfo responseInfo) {
            return  HttpResponse.BodySubscribers.mapping(HttpResponse.BodySubscribers.ofByteArray(),bytes -> gson.fromJson(new String(bytes),this.type));
        }

        public static class JsonHandlers {
            private JsonHandlers(){}
            public static <T> JsonHandler<T> ofType(Class<T> type){
                return of(new Gson(), type);

            }

            public static <T> JsonHandler<T> of(Gson gson, Class<T> type){
                return new JsonHandler<T>( type,gson);
            }

        }
    }
    ```
    使用`client`发送请求,并接收响应.  
    ```java
    var client = HttpClient.newHttpClient(URI.create("Http://localhost:8080"));
    var response = client.send(request, JsonHandler.JsonHandlers.ofType(UserBody.class));
    var userBody = response.body();
    ```
    // or  使用异步响应
    ```java 
    var task = client.sendAsync(request, JsonHandler.JsonHandlers.ofType(UserBody.class))
                .thenApply(HttpResponse::body)
                .thenApply(UserBody::getName)
                .thenAccept(System.out::println);
    task.get();//测试方便输出结果.
    ```
2. `x-www-form-urlencoded` 请求  

    这种请求类型是`form`表单的默认请求类型,另一种就是可以上传文件的`form-data`了,但是没有现成的类或者方法支持`x-www-form-urlencoded`请求,不过该请求投类型很好分析  

    - 将`form`表单里面的`name`和`value`用`=`链接,在把他们用`&`符号链接起来,如果包含空格替换为`+`,如果有特殊符号,则转换为`ASCII HEX`值;如果包含中文字符,则转成`ASCII HEX`后在百分号编码.  
    - 百分号编码: 汉字在`utf-8`字符集里面是占3个字节的,所以转换成16进制字符串就是占6个字节,每两个字节前面加一个百分号,就变成9个字节传递.  
    - 如果是`GET`请求,那直接在`url`后`?`拼接.
    - 如果是`POST`请求, 那就把拼接好的字符串放在`body`里面.
    - 简单点就是用现成的库`urlencoded`, 这种库应该是大部分语言都自带的. 

    ```java
    public static HttpRequest.BodyPublisher ofXForm(Map<Object,Object> map){
        var builder = new StringBuilder();
        map.forEach((key, value) -> {
            if (builder.length() > 0) {
                builder.append("&");
            }
            builder.append(URLEncoder.encode(key.toString(), StandardCharsets.UTF_8));
            builder.append("=");
            builder.append(URLEncoder.encode(value.toString(), StandardCharsets.UTF_8));
        });
        return HttpRequest.BodyPublishers.ofString(builder.toString());
    }
    ```


3. 文件上传下载  

    - 下载  

    下载很简单直接,有现成的方法可以使用.  
    ```java
    var client = HttpClient.newHttpClient();
    var request = HttpRequest.newBuilder(URI.create(url)).build();
    var file = Paths.get("1.png");
    var response = client.send(request,BodyHandlers.ofFile(file));
    ```
    该方法适合知道文件名称时使用.  
    or 
    ```java
    var client = HttpClient.newHttpClient();
    var request = HttpRequset.newBuilder(URI.create(url)).build();
    var file = Paths.get("/usr/local/file");
    var response = client.send(requset,BodyHandlers.ofFileDownload(file));
    ```
    `ofFileDownload`属于比较常见的下载方式.  

    - 上传

    上传没有现成的方法,所以需要我们自定义一个`BodyPublishers.ofFile()`方法,然后请求头为`mutipart/form-data`发送请求  
    ```java
    public static HttpRequest.BodyPublisher ofFile(Map<Object,Object> data,String boundary) throws IOException {
        var byteArrays = new ArrayList<byte[]>();
        byte[] separator = ("--" + boundary + "\r\nContent-Disposition: form-data; name=")
                .getBytes(StandardCharsets.UTF_8);
        for (Map.Entry<Object, Object> entry : data.entrySet()) {
            byteArrays.add(separator);

            if (entry.getValue() instanceof Path) {
                var path = (Path) entry.getValue();
                String mimeType = Files.probeContentType(path);
                byteArrays.add(("\"" + entry.getKey() + "\"; filename=\"" + path.getFileName()
                        + "\"\r\nContent-Type: " + mimeType + "\r\n\r\n").getBytes(StandardCharsets.UTF_8));
                byteArrays.add(Files.readAllBytes(path));
                byteArrays.add("\r\n".getBytes(StandardCharsets.UTF_8));
            }
            else {
                byteArrays.add(("\"" + entry.getKey() + "\"\r\n\r\n" + entry.getValue() + "\r\n")
                        .getBytes(StandardCharsets.UTF_8));
            }
        }
        byteArrays.add(("--" + boundary + "--").getBytes(StandardCharsets.UTF_8));
        return HttpRequest.BodyPublishers.ofByteArrays(byteArrays);
    }
    ```
    ```java
    Map<Object,Object> data = new HashMap<>();
    data.put("apikey", virusTotalApiKey);
    data.put("file", localFile);
    String boundary = new BigInteger(256, new Random()).toString();

    request = HttpRequest.newBuilder()
              .header("Content-Type", "multipart/form-data;boundary=" + boundary)
              .POST(ofMimeMultipartData(data, boundary))
              .uri(URI.create(url))
              .build();
    HttpResponse<String> response = client.send(request, BodyHandlers.ofString());
    ```

## 高阶使用  

   1. HTTP2 server push 

   2. WebSocket  

   ```java
   WebSocket webSocket = HttpClient.newHttpClient().newWebSocketBuilder().buildAsync(new URI("ws://localhost:8081/platform/device/gps"), new WebSocket.Listener() {
            @Override
            public CompletionStage<?> onText(WebSocket webSocket,
                                             CharSequence data, boolean last) {


                System.out.println("onText: " + data);

                return WebSocket.Listener.super.onText(webSocket, data, last);
            }

            @Override
            public void onOpen(WebSocket webSocket) {
                System.out.println("onOpen");
                WebSocket.Listener.super.onOpen(webSocket);
            }

            @Override
            public CompletionStage<?> onClose(WebSocket webSocket, int statusCode,
                                              String reason) {
                System.out.println("onClose: " + statusCode + " " + reason);
                return WebSocket.Listener.super.onClose(webSocket, statusCode, reason);
            }
        }).join();
        Gson gson = new Gson();
        Message message = new Message();
        message.setFrom("dbj");
        message.setContent("client data send");
        message.setTo("some one");
        webSocket.sendText(gson.toJson(message),true);
   ```
   其中`super.OnXxxx()`为固定句式, 其实就是调用`websocket.requset(1)`.固定调用.   
