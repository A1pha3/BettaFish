# 部署指南

本指南介绍 BettaFish 在不同环境下的部署方案。

## 学习目标

完成本章学习后，你将能够：

- [ ] 选择合适的**部署方式**（Docker/源码/K8s/云服务）
- [ ] 完成 **Docker Compose 部署**
- [ ] 配置 **Nginx 反向代理**
- [ ] 配置 **Systemd 服务**实现开机自启
- [ ] 进行**生产环境优化**和安全配置
- [ ] 实施**备份和恢复策略**

## 部署方式概览

| 方式 | 难度 | 适用场景 | 推荐度 |
|------|------|----------|--------|
| **Docker Compose** | ⭐ | 生产环境、快速部署 | ⭐⭐⭐⭐⭐ |
| **源码部署** | ⭐⭐ | 开发环境、定制需求 | ⭐⭐⭐ |
| **Kubernetes** | ⭐⭐⭐ | 大规模部署 | ⭐⭐⭐⭐ |
| **云服务** | ⭐ | 无服务器需求 | ⭐⭐⭐⭐ |

## Docker Compose 部署（推荐）

### 前置要求

- Docker 20.10+
- Docker Compose 2.0+

### 步骤

#### 1. 准备配置文件

```bash
# 克隆项目
git clone https://github.com/666ghj/BettaFish.git
cd BettaFish

# 复制配置文件
cp .env.example .env

# 编辑配置
nano .env
```

#### 2. 配置 docker-compose.yml

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: bettafish
      POSTGRES_PASSWORD: bettafish
      POSTGRES_DB: bettafish
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  app:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    volumes:
      - ./logs:/app/logs
      - ./final_reports:/app/final_reports

volumes:
  postgres_data:
```

#### 3. 启动服务

```bash
# 构建并启动
docker compose up -d

# 查看日志
docker compose logs -f app

# 停止服务
docker compose down
```

### 配置说明

### 镜像加速

如果镜像拉取慢，可以使用国内镜像源：

```yaml
services:
  app:
    build:
      context: .
      args:
        - MIRROR_URL=https://mirrors.aliyun.com
```

## 源码部署

### 系统要求

- Python 3.9+
- PostgreSQL 12+ 或 MySQL 8+
- 2GB+ 内存
- 10GB+ 硬盘

### 步骤

#### 1. 安装系统依赖

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.11 python3.11-venv postgresql nginx

# CentOS/RHEL
sudo yum install python311 postgresql-server nginx
```

#### 2. 安装 Playwright

```bash
pip install playwright
playwright install chromium
```

#### 3. 配置数据库

```bash
# PostgreSQL
sudo -u postgres psql
CREATE USER bettafish WITH PASSWORD 'your_password';
CREATE DATABASE bettafish OWNER bettafish;
GRANT ALL PRIVILEGES ON DATABASE bettafish TO bettafish;
```

#### 4. 安装 Python 依赖

```bash
# 创建虚拟环境
python3.11 -m venv venv
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

#### 5. 配置环境变量

```bash
cp .env.example .env
nano .env
```

#### 6. 启动应用

```bash
# 开发模式
python app.py

# 生产模式（使用 Gunicorn）
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

## Nginx 配置

### 反向代理配置

```nginx
# /etc/nginx/sites-available/bettafish

server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static {
        alias /path/to/BettaFish/static;
    }

    location /socket.io {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 启用配置

```bash
sudo ln -s /etc/nginx/sites-available/bettafish /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## Systemd 服务配置

### 创建服务文件

```ini
# /etc/systemd/system/bettafish.service

[Unit]
Description=BettaFish Application
After=network.target postgresql.service

[Service]
User=bettafish
Group=bettafish
WorkingDirectory=/path/to/BettaFish
Environment="PATH=/path/to/BettaFish/venv/bin"
ExecStart=/path/to/BettaFish/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

### 启动服务

```bash
sudo systemctl daemon-reload
sudo systemctl enable bettafish
sudo systemctl start bettafish
sudo systemctl status bettafish
```

## Kubernetes 部署

### Deployment 配置

```yaml
# deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: bettafish
spec:
  replicas: 3
  selector:
    matchLabels:
      app: bettafish
  template:
    metadata:
      labels:
        app: bettafish
    spec:
      containers:
      - name: bettafish
        image: bettafish:latest
        ports:
        - containerPort: 5000
        env:
        - name: DB_HOST
          value: postgres-service
        - name: INSIGHT_ENGINE_API_KEY
          valueFrom:
            secretKeyRef:
              name: bettafish-secrets
              key: insight-api-key
---
apiVersion: v1
kind: Service
metadata:
  name: bettafish-service
spec:
  selector:
    app: bettafish
  ports:
  - port: 80
    targetPort: 5000
  type: LoadBalancer
```

### 部署

```bash
kubectl apply -f deployment.yaml
kubectl apply -f secret.yaml
kubectl get pods
```

## 云服务部署

### AWS 部署

#### 使用 ECS

1. 推送 Docker 镜像到 ECR
2. 创建 ECS 任务定义
3. 配置负载均衡器
4. 启动服务

#### 使用 Lambda

需要改造为 Serverless 架构：

```python
# lambda_handler.py
from mangum import Mangum
from app import app

handler = Mangum(app, lifespan="off")
```

### 阿里云部署

#### 使用 ECS

1. 推送镜像到阿里云容器镜像服务
2. 创建 ECS 实例
3. 配置负载均衡
4. 启动服务

## 生产环境优化

### 性能优化

```bash
# Gunicorn 配置
workers = (2 * cpu_count) + 1
threads = 2

# 启动命令
gunicorn -w $workers --threads $threads -k gthread app:app
```

### 日志管理

```python
# config.py
LOG_LEVEL = "INFO"
LOG_ROTATION = "100 MB"
LOG_RETENTION = "30 days"
```

### 监控配置

```bash
# 安装监控工具
pip install prometheus-fastapi-instrumentator

# 添加监控端点
from prometheus_fastapi_instrumentator import Instrumentator
Instrumentator().instrument(app).expose(app)
```

## 安全配置

### 环境变量保护

```bash
# 使用环境变量文件
chmod 600 .env

# 生产环境使用 Secrets 管理工具
# - HashiCorp Vault
# - AWS Secrets Manager
# - 阿里云 KMS
```

### 数据库安全

```bash
# 使用强密码
# 限制远程访问
# 定期备份

# PostgreSQL
pg_dump -U bettafish bettafish > backup.sql

# MySQL
mysqldump -u bettafish -p bettafish > backup.sql
```

## 备份策略

### 数据库备份

```bash
# 每日备份脚本
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_DIR="/backups/bettafish"

pg_dump -U bettafish bettafish | gzip > $BACKUP_DIR/bettafish_$DATE.sql.gz

# 保留最近 30 天
find $BACKUP_DIR -name "bettafish_*.sql.gz" -mtime +30 -delete
```

### 配置文件备份

```bash
# 备份配置
cp .env .env.backup

# 版本控制（排除敏感信息）
echo ".env" >> .gitignore
```

## 故障恢复

### 数据库恢复

```bash
# PostgreSQL
gunzip < bettafish_20240120.sql.gz | psql -U bettafish bettafish

# MySQL
gunzip < bettafish_20240120.sql.gz | mysql -u bettafish -p bettafish
```

### 应用恢复

```bash
# 回滚到之前版本
git checkout <commit-hash>
docker compose down
docker compose up -d --build
```

## 常见问题

### Q1: Docker 构建失败？

**解决方案**：

```bash
# 1. 检查网络连接
ping -c 3 registry-1.docker.io

# 2. 使用镜像加速
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
EOF
sudo systemctl restart docker

# 3. 清理 Docker 缓存
docker system prune -a

# 4. 重新构建
docker compose build --no-cache
```

### Q2: 数据库连接失败？

**排查步骤**：

```bash
# 1. 检查数据库服务
docker compose ps db

# 2. 检查数据库日志
docker compose logs db

# 3. 测试连接
docker compose exec db psql -U bettafish -d bettafish -c "SELECT 1"

# 4. 检查环境变量
docker compose exec app env | grep DB_
```

### Q3: 内存不足？

**优化方案**：

```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

## 实践练习

### 练习 1：Docker 部署验证 ⭐

**任务**：完成 Docker 部署并验证所有服务正常运行

```bash
# 1. 启动服务
docker compose up -d

# 2. 检查服务状态
docker compose ps

# 3. 验证数据库
docker compose exec db psql -U bettafish -d bettafish -c "SELECT COUNT(*) FROM xhs_note LIMIT 1"

# 4. 验证 Web 服务
curl http://localhost:5000

# 5. 查看日志
docker compose logs --tail=50 app
```

### 练习 2：Nginx 反向代理配置 ⭐⭐

**任务**：配置 Nginx 反向代理

```bash
# 1. 安装 Nginx
sudo apt install nginx

# 2. 创建配置文件
sudo tee /etc/nginx/sites-available/bettafish <<EOF
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }

    location /static {
        alias /path/to/BettaFish/static;
    }

    location /socket.io {
        proxy_pass http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
EOF

# 3. 启用配置
sudo ln -s /etc/nginx/sites-available/bettafish /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# 4. 验证
curl http://localhost/
```

### 练习 3：Systemd 服务配置 ⭐⭐⭐

**任务**：创建 Systemd 服务实现开机自启

```bash
# 1. 创建服务文件
sudo tee /etc/systemd/system/bettafish.service <<EOF
[Unit]
Description=BettaFish Application
After=network.target postgresql.service

[Service]
User=$USER
Group=$USER
WorkingDirectory=$(pwd)
Environment="PATH=$(pwd)/venv/bin"
ExecStart=$(pwd)/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 app:app
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# 2. 重载并启动
sudo systemctl daemon-reload
sudo systemctl enable bettafish
sudo systemctl start bettafish

# 3. 验证状态
sudo systemctl status bettafish
```

## 相关文档

- **[配置指南](config.md)** - 详细配置说明
- **[故障排查](troubleshooting.md)** - 问题排查
- **[快速开始](../getting-started/quickstart.md)** - 快速部署

---

> 💡 **学习提示**：生产环境部署需要考虑安全性、可靠性和可维护性。建议从 Docker Compose 开始，熟悉后再考虑 Kubernetes 等更复杂的部署方式。
