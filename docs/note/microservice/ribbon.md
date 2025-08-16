# ribbon
> 负载均衡
## 修改负载均衡规则
1. 代码方式:在微服务项目的application类中定义一个新的IRule:
    ```java
    @Bean
    public IRule randomRule(){
        return new RandomRule();
    }
    ```
2. 配置文件方式:在微服务项目的配置文件中添加如下配置:
    ```yaml
    usersevice:
      ribbon:
        NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
## 加载规则
>ribbon默认的加载规则是懒加载,只有第一次访问的时候才会去加载,所以第一次访问时间较长.可以通过修改配置文件的形式来修改默认的加载规则为饥饿加载

    ribbon:
      eager-load:
        enabled: true
        clients: usersevice



