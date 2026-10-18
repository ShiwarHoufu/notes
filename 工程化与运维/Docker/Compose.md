当一个应用由多个容器组成时，单独依次使用`docker run`很麻烦，于是就需要用**多容器编排**。
就是对这些容器进行统一管理、调度、启动、网络、存储、扩容、自愈。

# 配置文件

用来**一次性定义和管理多个 Docker 容器**。（`docker-compose.yml` ）

核心作用：
1. **定义多个服务 (service)**：每个服务对应一个容器，例如 `mysql`、`nginx`、`java-backend`
2. **端口映射、**配置数据卷、**配置网络**、**重启策略**** 等`docker run`里面的策略全部写在这里

## 常用命令

>（在配置文件目录执行）
```bash
# 启动所有服务（后台运行）
docker-compose up -d
# 查看运行状态
docker-compose ps
# 停止服务，容器保留
docker-compose stop
# 停止并删除容器、网络（数据卷不会删）
docker-compose down
# 查看日志
docker-compose logs -f
```