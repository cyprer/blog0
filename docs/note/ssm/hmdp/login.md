# 基于session实现登录
## 基本逻辑
![session登录流程](https://raw.githubusercontent.com/cyprer/photo/main/obsidian/20250302191020986.png)
## 集群session共享问题:
  >多个tomcat服务器之间session不共享(注意不是浏览器)  

解决方法逻辑: 
![](https://raw.githubusercontent.com/cyprer/photo/main/obsidian/20250302201053011.png)
##用户信息存储
  > 由于session不共享,因此用户信息需要存储在redis中  

  解决方法:
  - 使用hash结构存储用户信息
    > 至于为什么使用hash结构,是因为hash结构可以对对象的每个属性进行操作,而不是对整个对象进行操作,并且内存占用也比较小

## 增加全局拦截器
用于解决登录状态的刷新问题
 >原因是一个login拦截器只拦截有关login的请求,但其他不需要登录的功能就不会拦截,以至于token会失效,因此添加一个全局拦截器以达到刷新token的功能