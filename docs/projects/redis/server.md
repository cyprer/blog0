# redis服务端初始化流程

## 定义server接口并实现
> 接口  
```java
public interface RedisServer {
    public void start();
    public void stop();
}
```
> 使用netty实现
```java
public class RedisMiniServer implements RedisServer{

    private String host;
    private int port;
    private EventLoopGroup bossGroup;
    private EventLoopGroup workerGroup;
    private Channel serverChannel;

    public RedisMiniServer(String host, int port) {
        this.host = host;
        this.port = port;
        this.bossGroup = new NioEventLoopGroup(1);
        this.workerGroup = new  NioEventLoopGroup(4);
    }
    @Override
    public void start() {
        ServerBootstrap serverBootstrap = new ServerBootstrap();
        serverBootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    @Override
                    protected void initChannel(SocketChannel ch) throws Exception {
                        ChannelPipeline pipeline = ch.pipeline();
                        pipeline.addLast(new StringDecoder());
                        pipeline.addLast(new StringHandler());
                        pipeline.addLast(new StringEncoder());
                    }
                });
        try {
            serverChannel = serverBootstrap.bind(host, port).sync().channel();
            System.out.println("Redis Server start at " + host + ":" + port);
        } catch (InterruptedException e) {
            e.printStackTrace();
            stop();
            Thread.currentThread().interrupt();
        }
    }
    @Override
    public void stop() {
        try {
            if (serverChannel != null) {
                serverChannel.close().sync();
            }
            if (workerGroup != null) {
                workerGroup.shutdownGracefully().sync();
            }
            if (bossGroup != null) {
                bossGroup.shutdownGracefully().sync();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```
# Netty服务端工作流程笔记（RedisMiniServer）


## 一、整体概述
该类是基于Netty框架实现的简易Redis服务端，核心功能是：  
- 监听指定IP和端口  
- 接受客户端连接  
- 处理客户端发送的数据  
- 优雅关闭服务  


## 二、核心组件及作用
| 组件               | 作用                                                                 |
|--------------------|----------------------------------------------------------------------|
| `EventLoopGroup`   | 事件循环组（线程池），负责处理I/O操作和事件                           |
| `ServerBootstrap`  | 服务端启动辅助类，用于配置服务端参数                                 |
| `ChannelPipeline`  | 处理器链，负责按顺序处理网络数据（入站/出站）                         |
| `StringDecoder`    | 入站处理器，将网络字节数据解码为字符串                               |
| `StringHandler`    | 自定义业务处理器，处理解码后的字符串数据（如Redis命令逻辑）           |
| `StringEncoder`    | 出站处理器，将字符串数据编码为字节，用于向客户端发送响应               |
| `ServerChannel`    | 服务端监听通道，代表绑定的端口，负责接受新连接                       |


## 三、工作流程分步解析


### 1. 初始化阶段（构造方法）
```java
public RedisMiniServer(String host, int port) {
    this.host = host;
    this.port = port;
    this.bossGroup = new NioEventLoopGroup(1);  // 主事件循环组（1个线程）
    this.workerGroup = new NioEventLoopGroup(4);  // 工作事件循环组（4个线程）
}
```
- **核心操作**：创建2个`EventLoopGroup`  
  - `bossGroup`（主组）：仅负责**接受客户端新连接**（轻量工作，1个线程足够）  
  - `workerGroup`（工作组）：负责**处理已连接客户端的读写操作**（业务密集，4个线程提高效率）  


### 2. 启动配置阶段（start()方法）
```java
ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap.group(bossGroup, workerGroup)  // 关联事件循环组
    .channel(NioServerSocketChannel.class)  // 指定服务端通道类型（NIO非阻塞）
    .childHandler(new ChannelInitializer<SocketChannel>() {  // 配置新连接的处理器链
        @Override
        protected void initChannel(SocketChannel ch) throws Exception {
            ChannelPipeline pipeline = ch.pipeline();  // 获取通道的处理器链
            pipeline.addLast(new StringDecoder());     // 1. 先解码（字节→字符串）
            pipeline.addLast(new StringHandler());     // 2. 再处理业务（自定义逻辑）
            pipeline.addLast(new StringEncoder());     // 3. 最后编码（字符串→字节，用于响应）
        }
    });
```
- **核心操作**：通过`ServerBootstrap`配置服务端  
  - 绑定`bossGroup`和`workerGroup`  
  - 指定通道类型为`NioServerSocketChannel`（非阻塞I/O，高效处理多连接）  
  - 定义`childHandler`：新客户端连接建立时，自动初始化该连接的`ChannelPipeline`（处理器链）  


### 3. 绑定监听阶段（start()方法续）
```java
serverChannel = serverBootstrap.bind(host, port).sync().channel();
```
- **核心操作**：绑定端口并开始监听  
  - `bind(host, port)`：异步绑定指定IP和端口（非阻塞操作）  
  - `sync()`：等待异步绑定完成（阻塞当前线程，确保绑定成功后再继续）  
  - `channel()`：获取绑定成功后的`ServerChannel`（代表服务端监听通道）  


### 4. 处理客户端连接流程
当客户端发起连接并发送数据时，流程如下：  
1. **接受连接**：  
   - `bossGroup`的线程接受新连接，将连接交给`workerGroup`的某个线程（`EventLoop`）管理  

2. **数据流转**：  
   ```
   客户端发送字节数据 → 
   经过StringDecoder解码为字符串 → 
   交给StringHandler处理业务（如解析Redis命令） → 
   处理结果经StringEncoder编码为字节 → 
   发送回客户端
   ```

3. **注意**：  
   - `ChannelPipeline`中的处理器按添加顺序执行（入站处理器：Decoder→Handler；出站处理器：Encoder反向执行）  


### 5. 服务停止流程（stop()方法）
```java
public void stop() {
    if(bossGroup != null) bossGroup.shutdownGracefully();  // 关闭主事件循环组
    if(workerGroup != null) workerGroup.shutdownGracefully();  // 关闭工作事件循环组
    if(serverChannel != null) serverChannel.close();  // 关闭服务端监听通道
}
```
- **核心操作**：优雅关闭资源  
  - `shutdownGracefully()`：让线程池逐渐停止（处理完当前任务再关闭，避免数据丢失）  
  - 关闭顺序：先停事件循环组，再关监听通道  


## 四、Netty关键概念补充
1. **异步非阻塞**：  
   Netty的I/O操作都是异步的（不会阻塞当前线程），通过`Future`/`Promise`获取结果，效率远高于传统BIO（同步阻塞）。  

2. **EventLoop**：  
   每个`EventLoopGroup`包含多个`EventLoop`（线程），一个`EventLoop`负责处理多个通道的I/O事件，且线程与通道绑定（避免线程切换开销）。  

3. **入站/出站处理器**：  
   - 入站（Inbound）：数据从网络到应用（如Decoder）  
   - 出站（Outbound）：数据从应用到网络（如Encoder）  
   - `ChannelPipeline`中，入站处理器按添加顺序执行，出站处理器按相反顺序执行  


## 五、总结
该服务端的核心是通过Netty的组件协作，实现高效的网络通信：  
- 用`EventLoopGroup`管理线程，处理I/O事件  
- 用`ChannelPipeline`串联数据处理流程（解码→业务→编码）  
- 用`ServerBootstrap`简化服务端配置  
- 最终实现“监听端口→接受连接→处理数据→优雅关闭”的完整流程
## 定义一个StringHandler

> 代码
```java
public class StringHandler extends ChannelInboundHandlerAdapter {
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        System.out.println("收到信息:" + msg);
        ctx.channel().writeAndFlush("+OjbK\r\n");
    }
}
```

## 使用redis-cli测试
如图所示
![](https://raw.githubusercontent.com/cyprer/photo/main/obsidian/20250514164853980.png)