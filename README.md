# JUSI 基础设施服务

这是 JUSI 项目的共享基础设施服务，包含 MySQL 和 Redis，可被多个业务服务共享使用。

## 服务列表

- **MySQL 8.0**: 数据库服务，容器名 `jusi_mysql`
- **Redis 7**: 缓存服务，容器名 `jusi_redis`

## 快速开始

### 1. 配置环境变量

复制 `.env.example` 到 `.env` 并修改配置：

```bash
cp .env.example .env
```

修改 `.env` 文件中的密码等敏感信息。

### 2. 启动服务

```bash
# 启动所有基础设施服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 3. 停止服务

```bash
# 停止服务（保留数据）
docker-compose stop

# 停止并删除容器（保留数据卷）
docker-compose down

# 停止并删除容器和数据卷（慎用！）
docker-compose down -v
```

## 网络配置

服务使用共享网络 `jusi_shared_network`，其他业务服务需要连接到此网络才能访问这些基础设施服务。

业务服务的 docker-compose.yml 配置示例：

```yaml
version: '3.8'

services:
  your_app:
    # ... 其他配置
    environment:
      DB_HOST: jusi_mysql
      DB_PORT: 3306
      REDIS_HOST: jusi_redis
      REDIS_PORT: 6379
    networks:
      - jusi_shared_network

networks:
  jusi_shared_network:
    external: true
    name: jusi_shared_network
```

## 数据持久化

数据存储在 Docker 卷中：
- `mysql_data`: MySQL 数据
- `redis_data`: Redis 数据

## 数据库初始化

MySQL 容器启动时会自动执行 `init-scripts` 目录下的 SQL 脚本，用于初始化数据库结构。

## 端口映射

- MySQL: 3306 (可通过 `.env` 的 `DB_PORT` 修改)
- Redis: 6379 (可通过 `.env` 的 `REDIS_PORT` 修改)

## 健康检查

所有服务都配置了健康检查，确保服务正常运行后才标记为健康状态。

## 访问服务

### MySQL

```bash
# 使用 docker exec 访问
docker exec -it jusi_mysql mysql -u jusi -p

# 使用主机访问
mysql -h 127.0.0.1 -P 3306 -u jusi -p
```

### Redis

```bash
# 使用 docker exec 访问
docker exec -it jusi_redis redis-cli

# 如果设置了密码
docker exec -it jusi_redis redis-cli -a your_password

# 使用主机访问
redis-cli -h 127.0.0.1 -p 6379
```

## 维护命令

```bash
# 重启服务
docker-compose restart

# 查看资源使用情况
docker stats jusi_mysql jusi_redis

# 备份 MySQL 数据
docker exec jusi_mysql mysqldump -u root -proot123456 jusi_db > backup.sql

# 查看 MySQL 日志
docker logs jusi_mysql

# 查看 Redis 日志
docker logs jusi_redis
```

## 注意事项

1. 首次启动时请务必修改默认密码
2. 生产环境建议为 Redis 设置密码
3. 定期备份数据库
4. 监控容器资源使用情况
