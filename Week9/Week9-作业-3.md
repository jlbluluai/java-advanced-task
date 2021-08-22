# Table of Contents

* [Week9-作业-3](#week9-作业-3)
    * [题目](#题目)
    * [解题](#解题)
        * [尝试将服务端写死查找接口实现类变成泛型和反射](#尝试将服务端写死查找接口实现类变成泛型和反射)
        * [尝试将客户端动态代理改成 AOP，添加异常处理](#尝试将客户端动态代理改成-aop，添加异常处理)
        * [尝试使用 Netty+HTTP 作为 client 端传输方式](#尝试使用-nettyhttp-作为-client-端传输方式)


# Week9-作业-3

## 题目

> 改造自定义 RPC 的程序，提交到 GitHub：\
尝试将服务端写死查找接口实现类变成泛型和反射；\
尝试将客户端动态代理改成 AOP，添加异常处理；\
尝试使用 Netty+HTTP 作为 client 端传输方式。

## 解题

[项目地址](https://github.com/jlbluluai/xyz-study/tree/master/rpcfx)

### 尝试将服务端写死查找接口实现类变成泛型和反射

先取得request中的类信息通过Class.forName找寻类信息，若找不到直接返回异常

```java
public RpcfxResponse invoke(RpcfxRequest request) {
        RpcfxResponse response = new RpcfxResponse();
        String serviceClass = request.getServiceClass();

        // 作业1：改成泛型和反射
        // 先通过反射判断参数的全限定类名是否能找到类，若有问题，提早暴露，再根据类精确去查找对应的服务
        Class<?> clazz;
        try {
        clazz = Class.forName(serviceClass);
        } catch (ClassNotFoundException e) {
        e.printStackTrace();
        response.setException(e);
        response.setStatus(false);
        return response;
        }
        Object service = resolver.resolve(clazz);//this.applicationContext.getBean(serviceClass);

        try {
        Method method = resolveMethodFromClass(service.getClass(), request.getMethod());
        Object result = method.invoke(service, request.getParams()); // dubbo, fastjson,
        // 两次json序列化能否合并成一个
        response.setResult(JSON.toJSONString(result, SerializerFeature.WriteClassName));
        response.setStatus(true);
        return response;
        } catch ( IllegalAccessException | InvocationTargetException e) {

        // 3.Xstream

        // 2.封装一个统一的RpcfxException
        // 客户端也需要判断异常
        e.printStackTrace();
        response.setException(e);
        response.setStatus(false);
        return response;
        }
        }
```


### 尝试将客户端动态代理改成 AOP，添加异常处理

使用Cglib字节码生成动态代理代替JDK动态代理

```java
public static <T> T create(final Class<T> serviceClass, final String url, Filter... filters) {

        // 0. 替换动态代理 -> 字节码生成
//        return (T) Proxy.newProxyInstance(Rpcfx.class.getClassLoader(), new Class[]{serviceClass}, new RpcfxInvocationHandler(serviceClass, url, filters));

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(serviceClass);
        enhancer.setCallback(new RpcfxMethodInterceptor(serviceClass, url, filters));
        return (T) enhancer.create();
        }


public static class RpcfxMethodInterceptor implements MethodInterceptor {

    public static final MediaType JSONTYPE = MediaType.get("application/json; charset=utf-8");

    private Class<?> serviceClass;
    private String url;
    private Filter[] filters;
    private NettyHttpClient nettyHttpClient;

    public RpcfxMethodInterceptor(Class<?> serviceClass, String url, Filter[] filters) {
        this.serviceClass = serviceClass;
        this.url = url;
        this.filters = filters;
        this.nettyHttpClient = new NettyHttpClient();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        RpcfxRequest request = new RpcfxRequest();
        request.setServiceClass(this.serviceClass.getName());
        request.setMethod(method.getName());
        request.setParams(args);

        // 执行过滤器
        if (null != filters) {
            for (Filter filter : filters) {
                if (!filter.filter(request)) {
                    return null;
                }
            }
        }

        // 取得结果
        RpcfxResponse response = nettyHttpClient.post(url, request);
        if (response == null || !response.isStatus()) {
            if (response != null && response.getException() != null) {
                throw new RpcfxException(response.getException().getMessage(), response.getException());
            } else {
                throw new RpcfxException("Server error");
            }
        }

        return JSON.parse(response.getResult().toString());
    }
}
```


### 尝试使用 Netty+HTTP 作为 client 端传输方式

```java
public RpcfxResponse post(String url, RpcfxRequest rpcfxRequest) {
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
        // 取host和ip
        URL netUrl = new URL(url);
        String host = netUrl.getHost();
        int port = netUrl.getPort();

        NettyHttpClientHandler clientHandler = new NettyHttpClientHandler();

        Bootstrap b = new Bootstrap();
        b.group(workerGroup)
        .channel(NioSocketChannel.class)
        .remoteAddress(new InetSocketAddress(host, port))
        .option(ChannelOption.SO_KEEPALIVE, true)
        .handler(new ChannelInitializer<SocketChannel>() {
@Override
public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new HttpRequestEncoder());
        ch.pipeline().addLast(new HttpResponseDecoder());
        ch.pipeline().addLast(new HttpObjectAggregator(1024*1024));
        ch.pipeline().addLast(clientHandler);
        }
        });

        // Start the client.
        ChannelFuture cf = b.connect().sync();

        // 发送请求
        URI uri = new URI(url);
        FullHttpRequest request = new DefaultFullHttpRequest(HttpVersion.HTTP_1_0,
        HttpMethod.POST,
        uri.toASCIIString(),
        Unpooled.wrappedBuffer(JSON.toJSONBytes(rpcfxRequest)));
        request.headers().set(HttpHeaderNames.CONNECTION, HttpHeaderValues.KEEP_ALIVE);
        request.headers().set(HttpHeaderNames.CONTENT_LENGTH, request.content().readableBytes());
        request.headers().set(HttpHeaderNames.CONTENT_TYPE, HttpHeaderValues.APPLICATION_JSON);

        // 等待结果返回
        ChannelPromise channelPromise = clientHandler.sendMessage(request);
        channelPromise.await();
        RpcfxResponse rpcfxResponse = clientHandler.getRpcfxResponse();

        cf.channel().closeFuture().sync();
        return rpcfxResponse;
        } catch (Exception e) {
        log.error("netty post error", e);
        return null;
        } finally {
        workerGroup.shutdownGracefully();
        }
        }
```

```java
public class NettyHttpClientHandler extends ChannelInboundHandlerAdapter {

    private ChannelHandlerContext ctx;
    private ChannelPromise promise;
    private RpcfxResponse rpcfxResponse;

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
        this.ctx = ctx;
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg)
            throws Exception {
//        System.out.println("msg->" + msg);

        if (msg instanceof FullHttpResponse) {
            FullHttpResponse response = (FullHttpResponse) msg;
            ByteBuf buf = response.content();
            String body = buf.toString(CharsetUtil.UTF_8);

            rpcfxResponse = JSON.parseObject(body, RpcfxResponse.class);
            promise.setSuccess();
            buf.release();
        }
    }


    public ChannelPromise sendMessage(Object message) {
        while (ctx == null) {
            try {
                TimeUnit.MILLISECONDS.sleep(1);
            } catch (InterruptedException e) {
                log.error("等待ChannelHandlerContext实例化过程中出错", e);
            }
        }
        promise = ctx.newPromise();
        ctx.writeAndFlush(message);
        return promise;
    }


    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) {
        ctx.close();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }

    public RpcfxResponse getRpcfxResponse() {
        return rpcfxResponse;
    }
}
```