# 1.需求

使用netty开发的服务器，使用cordova和ionic混合开发方案，网络访问使用的angular的HttpClient,将访问netty开发的服务端，在浏览器里做测试结果如图

![](https://github.com/liukai90/liukai90.github.io/blob/master/netty/img/error.png)

这一看就是跨域错误，我在服务端解决。

```java
response.headers().set(ACCESS_CONTROL_ALLOW_ORIGIN,"*");
```

结果发现还是报错。

# 2.最终解决

后面我发现我只是设置了可以远程访问，但我使用angular发请求时自定义headers，而netty服务端没有设置，所以添加了以下代码解决

```java
   FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, OK, Unpooled.wrappedBuffer(content));
//        response.headers().set(CONTENT_TYPE, "text/plain; charset=UTF-8");
        response.headers().set(CONTENT_TYPE, "application/json;charset=UTF-8");
        String host = msg.headers().get("Host");
        logger.info("host:"+host);
        //允许跨域访问
        response.headers().set(ACCESS_CONTROL_ALLOW_ORIGIN,"*");
        response.headers().set(ACCESS_CONTROL_ALLOW_HEADERS,"*");//允许headers自定义
        response.headers().set(ACCESS_CONTROL_ALLOW_METHODS,"GET, POST, PUT,DELETE");
        response.headers().set(ACCESS_CONTROL_ALLOW_CREDENTIALS,"true");
```

