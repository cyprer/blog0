# Sds和redisString以及相关命令的实现

## Sds的实现
> Sds是redis中用来表示字符串的一种数据结构,它是一个动态字符串,可以根据需要自动调整大小.
```java
public class Sds {
    private byte[] bytes;
    private int len;
    private int alloc;

    private static final int SDS_MAX_PREALLOC = 1024 * 1024; // 1M

    public Sds(byte[] bytes) {
        this.len = bytes.length;
        this.alloc = calculateAlloc(bytes.length);
        this.bytes = new byte[this.alloc];
        System.arraycopy(bytes, 0, this.bytes, 0, bytes.length);
    }

    private int calculateAlloc(int length) {
        if(length <= SDS_MAX_PREALLOC){
            return Math.max(length*2, 8);
        }
        return length+SDS_MAX_PREALLOC;
    }


    public String toString(){
        return new String(this.bytes,  RedisBytes.CHARSET);
    }

    public int length(){
        return len;
    }

    public void setBytes(byte[] bytes){
        this.bytes = bytes;
    }

    public void clear(){
        this.len = 0;
    }

    public Sds append(byte[] extra){
        int newLen = len + extra.length;
        if(newLen > this.alloc){
            int newAlloc = calculateAlloc(newLen);
            byte[] newBytes = new byte[newAlloc];
            System.arraycopy(bytes, 0, newBytes, 0, len);
            bytes = newBytes;
            this.alloc = newAlloc;
            this.len = newLen;
        }
        return this;
    }

    public Sds append(String str){
        return append(str.getBytes());
    }

    public byte[] getBytes(){
        byte[] result = new byte[len];
        System.arraycopy(bytes, 0, result, 0, len);
        return result;
    }
}
```
## redisBytes的实现
> 封装Bytes,修改其编码以及equals方法
```java
public class RedisBytes {
    private byte[] bytes;
    public static final Charset CHARSET = StandardCharsets.UTF_8;

    public RedisBytes(byte[] bytes) {
        this.bytes = bytes;
    }

    @Override
    public boolean equals(Object o){
        if(o == this){
            return true;
        }
        if(o == null || o.getClass() != getClass()){
            return false;
        }
        RedisBytes other = (RedisBytes) o;
        return Arrays.equals(bytes, other.bytes);
    }

    @Override
    public int hashCode(){
        return Arrays.hashCode(bytes);
    }

    public byte[] getBytes(){
        return bytes;
    }
    public String getString(){
        return new String(bytes, StandardCharsets.UTF_8);
    }
}
```
## redisString的实现
> 使用Sds实现redisString
```java
public class RedisString implements RedisData{
    private volatile long timeout;
    private Sds value;

    public RedisString(Sds value) {
        this.value = value;
        this.timeout = -1;
    }
    @Override
    public long timeout() {
        return timeout;
    }

    @Override
    public void setTimeout(long timeout) {
        this.timeout = timeout;
    }

    public RedisBytes getValue() {
        return new RedisBytes(value.getBytes());
    }

    public void setSds(Sds sds) {
        this.value = sds;
    }

    public long incr(){
        try{
            long cur = Long.parseLong(value.toString());
            long newValue = cur + 1;
            value = new Sds(String.valueOf(newValue).getBytes());
            return newValue;
        }catch(NumberFormatException e){
            throw new IllegalStateException("value is not a number");
        }
    }
}
```
## Get和Set的命令实现
> Get命令的实现
```java
public class Get implements Command {
    private RedisCore redisCore;
    private RedisBytes key;

    public Get(RedisCore redisCore) {
        this.redisCore = redisCore;
    }
    @Override
    public CommandType getType() {
        return CommandType.GET;
    }

    @Override
    public void setContext(Resp[] array) {
        key = ((BulkString)array[1]).getContent();
    }

    @Override
    public Resp handle() {
        try{
            RedisData data = redisCore.get(key);
            if(data == null){
                return new BulkString(new RedisBytes(NULL_BYTES));
            }
            if(data instanceof RedisString){
                RedisString redisString = (RedisString) data;
                return new BulkString(redisString.getValue());
            }
        }catch(Exception e){
            log.error("handle error", e);
            return new Errors("ERR internal server error");
        }
        return new Errors("ERR unknown error");
    }
}
```
> Set命令的实现
```java
public class Set implements Command {
    private RedisBytes key;
    private RedisBytes value;
    private RedisCore redisCore;

    public Set(RedisCore redisCore) {
        this.redisCore = redisCore;
    }
    @Override
    public CommandType getType() {
        return CommandType.SET;
    }

    @Override
    public void setContext(Resp[] array) {
        if(array.length < 3){
            throw new IllegalStateException("参数不足");
        }
        key = ((BulkString)array[1]).getContent();
        value = ((BulkString)array[2]).getContent();
    }

    @Override
    public Resp handle() {
        if(redisCore.get(key) != null){
            RedisData data = redisCore.get(key);
            if(data instanceof RedisString){
                RedisString redisString = (RedisString) data;
                redisString.setSds(new Sds(value.getBytes()));
                return new SimpleString("OK");
            }
        }
        redisCore.put(key, new RedisString(new Sds(value.getBytes())));
        log.info("set key:{} value:{}", key, value);

        return new SimpleString("OK");
    }
}
```
