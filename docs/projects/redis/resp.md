# resp协议的解析

## 定义Resp抽象类以及实现类(实现类实现抽象类的encode抽象方法)

```java
public abstract class Resp {
    public static final byte[] CRLF = "\r\n".getBytes();

    //SimpleString "+OK\r\n"
    //Errors "-Error message\r\n"
    //RedisInteger :0\r\n
    //BulkString "$6\r\nfoobar\r\n"
    //RespArray "*2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n"
    public static Resp decode(ByteBuf buffer){
        //判断是不是一个完整的命令
        if(buffer.readableBytes() <=0){
            throw new RuntimeException("没有一个完整的命令");
        }

        //拿到符号
        char c = (char)buffer.readByte();
        switch (c){
            case '+':
                return new SimpleString(getString(buffer));
            case '-':
                return new Errors(getString(buffer));
            case ':':
                return new RespInteger(getNumber(buffer));
            case '$':
                int length = getNumber(buffer);
                if(buffer.readableBytes() < length+2){
                    throw new IllegalStateException("没有找到换行符");
                }

                byte[] content;
                if(length == -1){
                    content = null;
                }else{
                    content = new byte[length];
                    buffer.readBytes(content);
                }
                if(buffer.readByte() != '\r' || buffer.readByte() != '\n'){
                    throw new IllegalStateException("没有找到换行符");
                }

                return new BulkString(content);
            case '*':
                int number = getNumber(buffer);
                Resp[] array = new Resp[number];
                for(int i=0;i<number;i++){
                    array[i] = decode(buffer);
                }
                return new RespArray(array);

            default:
                throw new IllegalStateException("不是Resp类型");
        }
    }

    public abstract void encode(Resp resp, ByteBuf byteBuf);

    //Errors "-Error message\r\n"
    static String getString(ByteBuf buffer){
        char c;
        StringBuilder result = new StringBuilder();
        while((c = (char)buffer.readByte()) != '\r' && buffer.readableBytes()>0){
            result.append(c);
        }
        if(buffer.readableBytes()<=0 || buffer.readByte() != '\n'){
            throw new IllegalStateException("没有找到换行符");
        }
        return result.toString();
    }

    static int getNumber(ByteBuf buffer){
        char c;
        c = (char)buffer.readByte();
        boolean positive = true;
        int value = 0;
        if(c == '-'){
            positive = false;
        }
        else{
            value = c - '0';
        }
        while((c = (char)buffer.readByte()) != '\r' && buffer.readableBytes()>0){
            value = value*10 + (c - '0');
        }
        if(buffer.readableBytes()<=0 || buffer.readByte() != '\n'){
            throw new IllegalStateException("没有找到换行符");
        }
        if(!positive){
            value = -value;
        }
        return value;

    }
}
```
## RespDecoder,RespCommandHandler,RespEncoder的实现
> RespDecoder
```java
public class RespDecoder extends ByteToMessageDecoder {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        try{
            if(in.readableBytes() >0){
                in.markReaderIndex();
            }

            if(in.readableBytes() < 4){
                return;
            }

            try{
                Resp resp = Resp.decode(in);
                if(resp != null){
                    log.debug("decode resp:{}", resp);
                    out.add(resp);
                }
            }catch(Exception e){
                log.error("decode error");
                in.resetReaderIndex();
                return;
            }
        }catch(Exception e){
            log.error("decode error", e);
        }
    }
}
```
- RespCommandHandler的构造需要redisCore,命令的处理构造过程需要redisCore,redisCore是核心储存区域
> RespCommandHandler
```java
public class RespCommandHandler extends SimpleChannelInboundHandler<Resp> {

    private final RedisCore redisCore;
    public RespCommandHandler(RedisCore redisCore) {
        this.redisCore = redisCore;
    }
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Resp msg) throws Exception {
        if(msg instanceof RespArray){
            RespArray respArray = (RespArray) msg;
            Resp response = processCommand(respArray);

            if(response!=null){
                ctx.channel().writeAndFlush(response);
            }
        }else{
            ctx.channel().writeAndFlush(new Errors("不支持的命令"));
        }
    }

    private Resp processCommand(RespArray respArray) {
        if(respArray.getContent().length==0){
            return new Errors("命令不能为空");
        }

        try{
            Resp[] array = respArray.getContent();
            String commandName = new String(((BulkString)array[0]).getContent());
            commandName = commandName.toUpperCase();
            CommandType commandType;

            try{
                commandType = CommandType.valueOf(commandName);
            }catch (IllegalArgumentException e){
                return new Errors("命令不存在");
            }

            Command command = commandType.getSupplier().apply(redisCore);
            command.setContext(array);
            Resp result = command.handle();

            return result;
            }catch (Exception e){
                log.error("命令执行失败",e);
                return new Errors("命令执行失败");
            }
        }
}
```
> RespEncoder
```java
public class RespEncoder extends MessageToByteEncoder<Resp> {

    @Override
    protected void encode(ChannelHandlerContext ctx, Resp msg, ByteBuf out) throws Exception {
        try{
            msg.encode(msg, out);
        }catch(Exception e){
            log.error("encode error");
            ctx.channel().close();
        }
    }
}
```
## 创建Command接口,实现类以及枚举类CommandType
> Command接口
```java
public interface Command {
    CommandType getType();
    void setContext(Resp[] array);
    Resp handle();
}
```
> CommandType枚举类
```java
public enum CommandType {
    PING(core ->new Ping());

    private final Function<RedisCore, Command> supplier;

    CommandType(Function<RedisCore, Command> supplier) {
        this.supplier = supplier;
    }
}
```
> Ping命令的实现
```java
public class Ping implements Command {

    @Override
    public CommandType getType() {
        return CommandType.PING;
    }

    @Override
    public void setContext(Resp[] array) {
        //不需要内容
    }

    @Override
    public Resp handle() {
        return new SimpleString("PONGPONGPONGSAHUR");
    }
}
```