# 基于session实现登录
## 基本逻辑
![session登录流程](https://raw.githubusercontent.com/cyprer/photo/main/obsidian/20250302191020986.png)
## 集群session共享问题:
  >多个tomcat服务器之间session不共享
  解决方法逻辑: 
![](https://raw.githubusercontent.com/cyprer/photo/main/obsidian/20250302201053011.png)
## 增加全局拦截器
 >原因是一个login拦截器只拦截有关login的请求,但其他不需要登录的功能就不会拦截,以至于token会失效,因此添加一个全局拦截器以达到刷新token的功能