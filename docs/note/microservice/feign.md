# Feign
    Feign是一个声明http客户端，使用Feign，帮助我们更加方便的调用http接口
## Feign使用
1. 添加依赖
2. 在启动类中添加@EnableFeignClients注解
3. 创建接口，并添加@FeignClient注解，指定服务名,添加方法