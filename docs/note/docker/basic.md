# Docker 基础命令使用教程
## 一、Docker 核心概念（快速理解）
| 概念       | 说明                                                                 |
|------------|----------------------------------------------------------------------|
| 镜像（Image） | 容器的“只读模板”，包含运行应用所需的代码、依赖、配置等，不可直接修改   |
| 容器（Container） | 镜像的运行实例，可启动、停止、删除，是独立的运行环境                 |
| 仓库（Repository） | 存储镜像的仓库（如 Docker Hub），用于拉取/推送镜像                   |

## 二、镜像（Image）相关命令
### 1. 查看本地镜像
```bash
# 列出所有本地镜像（含名称、标签、ID、大小等）
docker images

# 仅输出镜像ID（用于批量操作）
docker images -q

# 列出所有镜像（包括中间层镜像）
docker images -a

# 按条件过滤镜像（如过滤名称包含nginx的镜像）
docker images --filter=reference="*nginx*"
```

### 2. 拉取镜像（从 Docker Hub）
```bash
# 基本格式：docker pull [镜像名]:[标签]，标签默认latest（最新版）
docker pull nginx                # 拉取nginx最新版
docker pull mysql:8.0            # 拉取mysql8.0版本
docker pull alpine:3.18          # 拉取轻量级alpine镜像
```

### 3. 搜索镜像
```bash
# 搜索Docker Hub上的镜像
docker search nginx

# 过滤官方镜像
docker search --filter "is-official=true" nginx
```

### 4. 删除镜像
```bash
# 删除单个镜像（镜像ID/名称均可）
docker rmi nginx:latest
docker rmi 9f8936812905

# 强制删除（镜像被容器引用时）
docker rmi -f nginx:latest

# 批量删除所有未使用的镜像
docker rmi $(docker images -q -f "dangling=true")

# 批量删除所有本地镜像（慎用）
docker rmi $(docker images -q)
```

### 5. 构建自定义镜像（Dockerfile）
```bash
# 基本格式：docker build -t [镜像名]:[标签] [Dockerfile目录]
# -t：指定镜像标签；. 表示Dockerfile在当前目录
docker build -t my-app:v1.0 .

# 构建时指定自定义Dockerfile名称
docker build -t my-app:v1.0 -f Dockerfile.dev .
```

## 三、容器（Container）相关命令
### 1. 查看容器
```bash
# 列出运行中的容器
docker ps

# 列出所有容器（运行中+已停止）
docker ps -a

# 仅列出最后创建的容器
docker ps -l

# 列出最后创建的N个容器（如3个）
docker ps -n 3

# 仅输出容器ID（批量操作常用）
docker ps -q

# 查看容器详细信息（JSON格式）
docker inspect [容器ID/容器名]
```

### 2. 创建并启动容器
```bash
# 核心格式：docker run [参数] [镜像名] [容器内命令]
# 关键参数说明：
# -d：后台运行容器
# -p：端口映射（宿主机端口:容器端口）
# -v：目录挂载（宿主机目录:容器目录）
# --name：指定容器名称（唯一）
# -e：设置环境变量
# -it：交互式运行（进入容器终端）
# --restart=always：容器退出时自动重启

# 示例1：后台运行nginx，映射80端口，命名为my-nginx
docker run -d -p 80:80 --name my-nginx nginx

# 示例2：交互式运行ubuntu，进入终端
docker run -it --name my-ubuntu ubuntu /bin/bash

# 示例3：运行mysql，设置密码+挂载数据目录+自动重启
docker run -d -p 3306:3306 --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v /host/mysql/data:/var/lib/mysql \
  --restart=always \
  mysql:8.0

# 示例4：运行容器后立即执行命令（查看nginx版本）
docker run --rm nginx nginx -v  # --rm：容器退出后自动删除
```

### 3. 容器启停/重启
```bash
# 启动已停止的容器
docker start my-nginx
docker start [容器ID]

# 停止运行中的容器
docker stop my-nginx
docker stop [容器ID]

# 强制停止容器（类似kill）
docker kill my-nginx

# 重启容器
docker restart my-nginx
```

### 4. 进入容器终端
```bash
# 推荐方式（exec，不影响容器主进程）
docker exec -it my-nginx /bin/bash

# 适用于无bash的轻量镜像（如alpine）
docker exec -it my-alpine sh

# 旧方式（attach，退出会导致容器停止，不推荐）
docker attach [容器ID]
```

### 5. 查看容器日志
```bash
# 查看容器全部日志
docker logs my-nginx

# 实时跟踪日志（类似tail -f）
docker logs -f my-nginx

# 查看最后100行日志
docker logs --tail 100 my-nginx

# 查看指定时间范围内的日志
docker logs --since "2025-01-01" --until "2025-01-02" my-nginx
```

### 6. 删除容器
```bash
# 删除已停止的容器
docker rm my-nginx
docker rm [容器ID]

# 强制删除运行中的容器
docker rm -f my-nginx

# 批量删除所有已停止的容器
docker rm $(docker ps -aq -f "status=exited")

# 批量删除所有容器（慎用）
docker rm -f $(docker ps -aq)
```

### 7. 容器数据拷贝
```bash
# 从容器拷贝文件到宿主机
docker cp my-nginx:/etc/nginx/nginx.conf /host/conf/

# 从宿主机拷贝文件到容器
docker cp /host/conf/my.conf my-nginx:/etc/nginx/conf.d/
```

## 四、其他常用命令
### 1. 查看 Docker 状态/信息
```bash
# 查看Docker服务状态（Linux）
systemctl status docker

# 查看Docker系统信息（镜像、容器数量等）
docker info

# 查看Docker版本
docker --version
docker version
```

### 2. 容器资源监控
```bash
# 实时查看容器CPU/内存/网络等资源使用情况
docker stats

# 查看指定容器的资源使用
docker stats my-nginx
```

### 3. 清理无用资源
```bash
# 清理停止的容器、未使用的镜像、网络、缓存（交互式）
docker system prune

# 清理所有无用资源（包括未使用的镜像，慎用）
docker system prune -a

# 仅清理构建缓存
docker builder prune
```

### 4. 镜像/容器重命名
```bash
# 镜像重命名
docker tag nginx:latest my-nginx:v1.0

# 容器重命名
docker rename my-nginx nginx-prod
```

## 五、常用命令速查表
| 操作目标 | 核心命令                                  | 补充说明                     |
|----------|-------------------------------------------|------------------------------|
| 镜像     | `docker images`/`pull`/`rmi`/`build`      | `-f` 强制删除，`-t` 指定标签 |
| 容器     | `docker ps`/`run`/`start`/`stop`/`rm`     | `-d` 后台运行，`-p` 端口映射 |
| 日志     | `docker logs -f [容器名]`                 | `-tail N` 查看最后N行        |
| 进入容器 | `docker exec -it [容器名] bash/sh`        | 推荐exec方式，避免attach     |
| 清理     | `docker system prune`                     | `-a` 清理所有无用镜像        |