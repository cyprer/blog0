# Spring Cloud Gateway 学习笔记
## 1. 什么是 Spring Cloud Gateway
Spring Cloud Gateway 是 Spring 官方推出的新一代 API 网关服务，基于 Spring 5、Spring Boot 2 和 Project Reactor 开发，用于替代 Netflix Zuul。
核心功能：路由转发、负载均衡、熔断降级、权限控制、限流、监控等。
相比 Zuul 的优势：基于非阻塞响应式编程（性能更高）、支持 WebSocket、强大的路由匹配、易于扩展的过滤器机制。
## 2. 核心概念
Route（路由）：网关基本组成，包含 ID、目标 URI、断言（Predicate）和过滤器（Filter），断言为真则路由匹配。
Predicate（断言）：基于 Java 8 Function Predicate，匹配 HTTP 请求的路径、方法、请求头等。
Filter（过滤器）：修改请求 / 响应，分 GatewayFilter（局部）和 GlobalFilter（全局）。
## 3. 环境搭建
### 3.1 引入依赖（Maven）
xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!-- 结合服务发现（如Eureka） -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- 熔断支持 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency> 

### 3.2 基本配置（application.yml）
yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service  # lb表示负载均衡，user-service为服务名
          predicates:
            - Path=/users/**
          filters:
            - StripPrefix=1  # 去除路径第一个前缀（如/users/1 → /1）
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/orders/**
            - Method=GET  # 仅匹配GET请求
          filters:
            - StripPrefix=1
  application:
    name: api-gateway

# 服务注册配置（Eureka为例）
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
  instance:
    preferIpAddress: true
## 4. 路由配置详解
### 4.1 常用断言（Predicate）
断言	说明	示例
Path	路径匹配	Path=/users/**
Method	HTTP 方法匹配	Method=GET,POST
Header	请求头匹配	Header=X-Request-Id, \d+
Query	请求参数匹配	Query=name, abc.
Host	主机名匹配	Host=**.example.com
Cookie	Cookie 匹配	Cookie=sessionId, test
After	指定时间后匹配	After=2023-10-01T00:00:00+08:00[Asia/Shanghai]
### 4.2 组合断言示例
yaml
spring:
  cloud:
    gateway:
      routes:
        - id: combined-route
          uri: http://example.com
          predicates:
            - Path=/api/**
            - Method=POST
            - Header=Content-Type, application/json
            - Query=version, v1
## 5.过滤器顺序
在 Spring Cloud Gateway 中，过滤器的执行顺序直接影响请求处理流程，其核心原则是 **“按优先级排序，先执行前置逻辑，后执行后置逻辑”**。以下是关于过滤器执行顺序的详细说明：
一、过滤器的分类与执行阶段
所有过滤器（全局过滤器、默认过滤器、局部过滤器）都会经历两个阶段：

“前置” 阶段：请求被路由到目标服务之前执行（如认证、日志记录开始）。
“后置” 阶段：目标服务返回响应之后执行（如日志记录结束、响应头修改）。

无论哪种过滤器，都会先执行自身的前置逻辑，再等待路由完成，最后执行后置逻辑。
二、核心排序规则
过滤器的执行顺序由优先级（order 值） 决定，遵循以下规则：

order 值越小，优先级越高，越先执行前置逻辑。
所有过滤器的前置逻辑执行完毕后，才会执行目标服务调用。
目标服务返回后，按 order 值从大到小执行后置逻辑（与前置顺序相反）。

例如：

过滤器 A（order=1）、过滤器 B（order=2）
前置阶段顺序：A → B
后置阶段顺序：B → A