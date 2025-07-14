# 基础层持久化数据
## 架构
    - persistent
        - dao接口(用来对接app层resources包下mybatis查询xml文件,需要加上mapper注解)
        - po(数据库实体类)
        -  redis(缓存)
        -  repository(仓储类用于操作数据的查询和缓存,实现的domain层的IRepository接口)