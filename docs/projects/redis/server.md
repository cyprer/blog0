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