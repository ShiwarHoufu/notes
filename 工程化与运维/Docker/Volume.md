三种挂载方式

| 类型         | 存储位置  | 谁管理    | 适用场景                     |
| ---------- | ----- | ------ | ------------------------ |
| Volume     | 宿主机   | Docker | 生产环境，数据库持久化，跨容器共享数据      |
| Bind Mount | 宿主机   | 用户自己   | 开发：宿主机代码实时同步到容器、挂载单独配置文件 |
| tmpfs      | 宿主机内存 | 内存     | 临时敏感数据，不落地磁盘，高性能         |
# 一、Volume
## 特点
- 由 Docker 统一管理，不用关心宿主机真实路径，/var/lib/docker/volumes/
- 可以命名（命名卷），也可以匿名（匿名卷）
- 权限、迁移更方便；支持备份、导出、在多个容器之间共享
- 宿主机不能直接随便改底层文件（不建议直接进 `/var/lib/docker/volumes` 手动修改）

## 命令示例
1. **用`docker run`**
```bash
# mydata是Docker 卷名称；
# /var/lib/mysql 是容器内 mysql 数据目录。
docker run -d --name mysql -v mydata:/var/lib/mysql mysql:8.0
```
 2. **用compose**
```yml
services:
  rabbitmq:
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq   # 引用 —— 指向顶层 volumes 的 key
volumes:
  rabbitmq-data:                          # 声明 (必须写)
    name: rabbitmq-data                   # 真正的 Docker 卷名(可选)
```
>最终卷的位置：`/var/lib/docker/volumes/rabbitmq-data/_data`

## 删除卷
```bash
# 删除指定卷
docker volume rm mydata
# 删除所有未使用的卷
docker volume prune
```
# 二、Bind Mount 绑定挂载

把**宿主机的一个真实目录/文件**直接映射到容器内。宿主机路径必须提前存在。
>如果宿主机目录为空，容器内对应目录的原有文件会被覆盖为空
## 特点
- **双向映射**。宿主机修改文件，容器内立刻生效；容器内修改，宿主机同步修改，
- 容器内程序如果有漏洞，可能篡改宿主机文件；安全、权限问题多
- 典型场景：开发。本地代码挂载到容器，不用每次改代码都重新构建镜像

## 命令示例

```bash
# 宿主机 /home/dev/code 挂载到容器 /app 
docker run -d -v /home/dev/code:/app 镜像名
```

# 三、tmpfs 内存挂载

数据放在宿主机内存，容器停止数据直接消失，不写磁盘。
适用：临时缓存、密钥，不需要持久化，追求速度。
