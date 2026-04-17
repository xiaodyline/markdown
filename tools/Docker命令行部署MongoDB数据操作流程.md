# Docker 命令行部署 MongoDB 数据操作流程（Linux）

## 1. 适用范围
- 目标：在 Linux 服务器上通过 Docker 命令行部署 MongoDB
- 方式：`docker run` 单容器（Demo/中小规模最常用）
- 数据持久化：宿主机目录挂载 + 可选备份目录

---

## 2. 前置检查

```bash
docker --version
docker ps
```

如未安装 Docker（Ubuntu）：

```bash
curl -fsSL https://get.docker.com | bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 3. 创建数据目录（宿主机）

```bash
sudo mkdir -p /data/mongodb/data
sudo mkdir -p /data/mongodb/backup
sudo chown -R 999:999 /data/mongodb/data
```

说明：
- Mongo 官方镜像默认用户 UID/GID 常见为 `999`
- 该目录用于容器重建后的数据持久化

---

## 4. 使用 `docker run` 启动 MongoDB（带账号）

```bash
docker run -d \
  --name dg-mongo \
  -p 27017:27017 \
  -v /data/mongodb/data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=rootAdmin \
  -e MONGO_INITDB_ROOT_PASSWORD='StrongPass!2026' \
  --restart unless-stopped \
  mongo:7.0
```

查看状态：

```bash
docker ps --filter name=dg-mongo
docker logs --tail 80 dg-mongo
```

---

## 5. 初始化业务库与业务账号

进入容器并执行 `mongosh`：

```bash
docker exec -it dg-mongo mongosh -u rootAdmin -p 'StrongPass!2026' --authenticationDatabase admin
```

在 shell 中执行：

```javascript
use tender_demo

db.createUser({
  user: "tender_app",
  pwd: "TenderApp!2026",
  roles: [ { role: "readWrite", db: "tender_demo" } ]
})
```

验证业务账号：

```bash
docker exec -it dg-mongo mongosh "mongodb://tender_app:TenderApp!2026@127.0.0.1:27017/tender_demo"
```

---

## 6. 应用连接串（Koa/Mongoose）

容器与应用同机（宿主机访问）：

```env
MONGO_URI=mongodb://tender_app:TenderApp!2026@127.0.0.1:27017/tender_demo
```

如果你的应用也在 Docker 网络中，建议使用容器名：

```env
MONGO_URI=mongodb://tender_app:TenderApp!2026@dg-mongo:27017/tender_demo
```

---

## 7. 导入初始化数据

### 7.1 从宿主机导入 JSON

```bash
docker cp ./seed/documents.json dg-mongo:/tmp/documents.json

docker exec -it dg-mongo mongoimport \
  --uri="mongodb://tender_app:TenderApp!2026@127.0.0.1:27017/tender_demo" \
  --collection=documents \
  --file=/tmp/documents.json \
  --jsonArray
```

### 7.2 从宿主机导入 CSV

```bash
docker cp ./seed/documents.csv dg-mongo:/tmp/documents.csv

docker exec -it dg-mongo mongoimport \
  --uri="mongodb://tender_app:TenderApp!2026@127.0.0.1:27017/tender_demo" \
  --collection=documents \
  --type=csv \
  --headerline \
  --file=/tmp/documents.csv
```

---

## 8. 备份与恢复

### 8.1 备份到容器内再拷出

```bash
docker exec -it dg-mongo sh -c 'rm -rf /tmp/backup && mkdir -p /tmp/backup && mongodump --uri="mongodb://rootAdmin:StrongPass!2026@127.0.0.1:27017/admin" --db=tender_demo --out=/tmp/backup'

docker cp dg-mongo:/tmp/backup /data/mongodb/backup/backup_$(date +%F_%H%M%S)
```

### 8.2 恢复

```bash
docker cp /data/mongodb/backup/backup_2026-04-10_120000 dg-mongo:/tmp/restore

docker exec -it dg-mongo mongorestore \
  --uri="mongodb://rootAdmin:StrongPass!2026@127.0.0.1:27017/admin" \
  --drop \
  --db=tender_demo \
  /tmp/restore/backup/tender_demo
```

---

## 9. 常用运维命令

查看日志：

```bash
docker logs -f dg-mongo
```

重启：

```bash
docker restart dg-mongo
```

停止/启动：

```bash
docker stop dg-mongo
docker start dg-mongo
```

进入容器：

```bash
docker exec -it dg-mongo bash
```

---

## 10. 安全建议（最小要求）

1. 生产环境不要使用示例密码，必须改成强密码
2. 不要直接对公网暴露 27017
3. 仅开放内网访问或指定白名单 IP
4. 定时备份并验证恢复可用性
5. 保留 `--restart unless-stopped`，避免异常后服务不自动恢复

