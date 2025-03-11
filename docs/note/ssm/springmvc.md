# Spring MVC 介绍

## 1. 什么是 Spring MVC
Spring MVC 是 Spring 框架的一个模块，专门用于 Web 开发。它是 Spring 的一个后续产品，通常被称为 Spring Web MVC。Spring MVC 是一个基于 Java 的轻量级 Web 框架，用于简化 Web 开发过程。它通过 MVC（模型-视图-控制器）设计模式，将 Web 应用分为模型、视图和控制器三个部分，从而提高代码的可维护性和可扩展性。

## 2. Spring MVC 的优点
1. 清晰的工程结构：遵循 MVC 设计模式，代码结构清晰，便于维护和扩展。
2. 强大的配置功能：支持多种配置方式，包括 XML 配置和注解配置。
3. 注解式开发：通过注解（如 `@Controller`、`@RequestMapping` 等）简化开发流程，减少配置文件的使用。
4. 便捷的异常处理：提供了全局异常处理器，能够统一处理 Web 应用中的异常。
5. 无缝集成 Spring：与 Spring 框架的其他模块（如 IoC 容器、AOP、事务管理等）无缝集成。
6. 灵活的视图解析：支持多种视图技术，如 JSP、Thymeleaf、Freemarker 等。
7. 数据绑定功能：自动将请求参数绑定到控制器的参数对象中，简化数据处理。
8. 国际化支持：支持国际化（i18n），可以轻松实现多语言支持。

## 3. Spring MVC 的工作原理
1. 用户发送请求至前端控制器（DispatcherServlet）。
2. DispatcherServlet 调用处理器映射器（HandlerMapping）。
3. 处理器映射器根据请求 URL 找到具体的处理器（Controller），并生成处理器对象及处理器拦截器（如果有则生成），一并返回给 DispatcherServlet。
4. DispatcherServlet 通过处理器适配器（HandlerAdapter）调用处理器（Controller）。
5. 执行处理器（Controller）。
6. Controller 执行完成返回 ModelAndView。
7. HandlerAdapter 将 Controller 执行结果 ModelAndView 返回给 DispatcherServlet。
8. DispatcherServlet 将 ModelAndView 传给视图解析器（ViewResolver）。
9. ViewResolver 解析后返回具体 View。
10. DispatcherServlet 对 View 进行渲染视图（即将模型数据填充至视图中）。
11. DispatcherServlet 响应用户。

## 4. Spring MVC 的关键组件
1. **DispatcherServlet**：前端控制器，是 Spring MVC 的核心，负责接收用户请求并调度到具体的处理器。
2. **HandlerMapping**：处理器映射器，用于将请求映射到具体的处理器（Controller）。
3. **HandlerAdapter**：处理器适配器，用于调用处理器（Controller），并处理返回值。
4. **Controller**：控制器，是具体的业务逻辑处理类，通常通过注解（如 `@Controller` 和 `@RequestMapping`）定义。
5. **ModelAndView**：包含模型数据和视图信息的对象，用于将处理结果传递给视图。
6. **ViewResolver**：视图解析器，用于解析视图名称并返回具体的视图实现。

## 5. Spring MVC 的注解
1. **@Controller**：标记一个类为控制器，处理用户请求。
2. **@RequestMapping**：用于映射请求路径到具体的处理器方法。
3. **@RequestParam**：用于绑定请求参数到方法参数。
4. **@ModelAttribute**：用于将模型数据添加到视图中。
5. **@ResponseBody**：用于将方法返回值直接写入 HTTP 响应体。
6. **@RestController**：用于定义 RESTful 风格的控制器，自动应用 `@ResponseBody`。

## 6. Spring MVC 的应用场景
Spring MVC 适用于各种 Web 应用开发，尤其是需要高性能、低耦合和易于维护的场景。它广泛应用于企业级 Web 应用、RESTful API 开发以及前后端分离的项目中。

## 7. 参考资料
- [Spring 官方文档](https://spring.io/projects/spring-framework)
- [Spring MVC 官方文档](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)
