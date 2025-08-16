# nacos

## 安装nacos
  去github安装即可
## nacos启动
1. 在bin目录下执行startup.cmd -m standalone
2. 访问http://localhost:8848/nacos
3. 登录账号密码都是nacos
4. 在微服务项目中导入依赖即可并配置
## nacos服务分级存储模型
   - 服务
     - 集群
       - 实例
> 可以在配置文件中修改实例的集群属性
## nacos负载均衡
默认是本地集群的负载均衡
可以配置为权重的负载均衡,设置为0至1之间的数字，数字越大，权重越高
## nacos环境隔离
> 使用namespace来实现环境隔离
> 不同的namespace下的服务不能互相访问
## nacos配置管理
1. 在nacos中创建一个配置管理服务
3. 导入依赖
2. 创建bootstrap.yaml文件,添加nacos的相关配置
## nacos配置热更新
- 在@value注入变量所在的类加上@refreshScope注解
- 新建一个配置类使用@configurationProperties注解实现配置热更新
## nacos配置优先级
1. 服务名-profile.yaml
2. 服务名.yaml
3. 本地配置

