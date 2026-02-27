# 快速开始

本指南将帮助你在 5 分钟内部署并运行 BettaFish 系统。

## 学习目标

完成本章后，你将能够：

- [ ] 完成系统的环境准备和依赖安装
- [ ] 配置必要的环境变量和 API 密钥
- [ ] 启动 BettaFish 系统并访问 Web 界面
- [ ] 提交第一个分析任务
- [ ] 理解系统各个组件的启动方式

## 前置要求

### 必需条件

| 项目 | 要求 | 检查命令 |
|------|------|----------|
| 操作系统 | Windows / Linux / macOS | `uname -a` 或 `ver` |
| Python | 3.9 或更高版本 | `python --version` |
| 内存 | 建议 2GB 以上 | `free -h` 或 `systeminfo` |
| 硬盘 | 至少 1GB 可用空间 | `df -h` 或 `wmic logicaldisk get freespace` |

### 推荐环境管理工具

```bash
# Conda（推荐新手）
conda create -n bettafish python=3.11

# uv（推荐高级用户，速度更快）
pip install uv
uv venv --python 3.11
```

## 安装步骤

### 方法一：Docker 部署（推荐 ⭐⭐⭐⭐⭐）

Docker 是最简单的部署方式，无需手动配置依赖。

#### 步骤 1：安装 Docker

**Linux (Ubuntu/Debian)**：
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

**macOS**：
```bash
brew install --cask docker
```

**Windows**：
下载并安装 [Docker Desktop](https://www.docker.com/products/docker-desktop)

#### 步骤 2：验证安装

```bash
docker --version
docker compose version
```

#### 步骤 3：克隆项目

```bash
git clone https://github.com/666ghj/BettaFish.git
cd BettaFish
```

#### 步骤 4：配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件（至少配置以下内容）
cat > .env << 'EOF'
# 数据库配置（Docker 会自动创建）
DB_HOST=db
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=bettafish
DB_NAME=bettafish
DB_DIALECT=postgresql

# Flask 配置
HOST=0.0.0.0
PORT=5000

# 至少配置一个 LLM（这里使用 DeepSeek 作为示例）
QUERY_ENGINE_API_KEY=your_deepseek_api_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat
EOF
```

#### 步骤 5：启动服务

```bash
# 启动所有服务
docker compose up -d

# 查看日志
docker compose logs -f app
```

#### 步骤 6：验证部署

打开浏览器访问 http://localhost:5000

### 方法二：源码部署

#### 步骤 1：创建虚拟环境

```bash
# 使用 Conda
conda create -n bettafish python=3.11
conda activate bettafish

# 或使用 uv
pip install uv
uv venv --python 3.11
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate   # Windows
```

#### 步骤 2：安装系统依赖

**Ubuntu/Debian**：
```bash
sudo apt update
sudo apt install -y python3-dev python3-pip postgresql postgresql-contrib
```

**macOS**：
```bash
brew install postgresql@14
```

#### 步骤 3：安装 Python 依赖

```bash
# 基础依赖安装
pip install -r requirements.txt

# uv 版本（更快）
pip install uv
uv pip install -r requirements.txt

# 如果不想使用情感分析模型，可以跳过机器学习部分：
# 编辑 requirements.txt，注释掉 torch、transformers 等包
```

#### 步骤 4：安装 Playwright

```bash
playwright install chromium
```

如果安装失败，尝试：

```bash
# 使用国内镜像
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright/
playwright install chromium
```

#### 步骤 5：配置数据库

**方式 A：使用 Docker 运行 PostgreSQL**

```bash
docker run -d \
  --name bettafish-db \
  -e POSTGRES_USER=bettafish \
  -e POSTGRES_PASSWORD=bettafish \
  -e POSTGRES_DB=bettafish \
  -p 5432:5432 \
  postgres:15
```

**方式 B：使用本地 PostgreSQL**

```bash
# 创建数据库和用户
sudo -u postgres psql
CREATE USER bettafish WITH PASSWORD 'bettafish';
CREATE DATABASE bettafish OWNER bettafish;
GRANT ALL PRIVILEGES ON DATABASE bettafish TO bettafish;
\q
```

#### 步骤 6：配置环境变量

```bash
# 复制配置模板
cp .env.example .env

# 编辑配置文件
nano .env  # 或使用你喜欢的编辑器
```

**最小配置示例**：

```bash
# 数据库配置
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=bettafish
DB_NAME=bettafish
DB_DIALECT=postgresql

# 至少配置一个 LLM
QUERY_ENGINE_API_KEY=sk-xxx
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

#### 步骤 7：启动系统

```bash
# 激活虚拟环境
conda activate bettafish  # 或 source .venv/bin/activate

# 启动系统
python app.py
```

## 配置说明

### 最小配置（快速测试）

```bash
# .env 文件
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=bettafish
DB_NAME=bettafish
DB_DIALECT=postgresql

# 只配置 Query Engine（用于快速测试）
QUERY_ENGINE_API_KEY=your_api_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

### 推荐的 LLM 配置

| Agent | 推荐模型 | 提供商 | 官方申请地址 | 特点 |
|-------|---------|--------|-------------|------|
| Insight | kimi-k2-0711-preview | Moonshot | https://platform.moonshot.cn/ | 长上下文，适合深度分析 |
| Media | gemini-2.5-pro | AIHubMix | https://aihubmix.com/?aff=8Ds9 | 多模态能力强 |
| Query | deepseek-chat | DeepSeek | https://www.deepseek.com/ | 响应快，性价比高 |
| Report | gemini-2.5-pro | AIHubMix | https://aihubmix.com/?aff=8Ds9 | 输出质量高 |
| Forum Host | qwen-plus | 阿里云 | https://www.aliyun.com/product/bailian | 对话能力强 |

### 获取 API Key

1. **DeepSeek**（推荐新手）
   - 访问：https://platform.deepseek.com/
   - 注册并获取 API Key
   - 新用户通常有免费额度

2. **Moonshot**（Kimi）
   - 访问：https://platform.moonshot.cn/
   - 注册并获取 API Key

3. **AIHubMix**（推荐中转服务）
   - 访问：https://aihubmix.com/?aff=8Ds9
   - 支持多个模型的统一接口

## 首次运行

### 1. 验证安装

打开浏览器访问 http://localhost:5000

你应该看到 BettaFish 的主界面，包含：
- 搜索输入框
- 分析按钮
- 状态显示区域

**预期界面**：
```
┌─────────────────────────────────────────────┐
│              BettaFish 微舆系统              │
├─────────────────────────────────────────────┤
│                                             │
│  ┌───────────────────────────────────────┐  │
│  │ 请输入要分析的主题...                   │  │
│  │                                    [搜索]│  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Agent 状态:                               │
│  □ Query   □ Media   □ Insight            │
│                                             │
└─────────────────────────────────────────────┘
```

### 2. 提交第一个分析任务

**步骤**：

1. 在搜索框中输入分析主题，例如：
   ```
   武汉大学品牌声誉分析
   ```

2. 点击"开始分析"按钮

3. 观察实时日志输出

**预期流程**：
```
[10:00:00] 启动分析任务...
[10:00:01] QueryAgent 开始工作...
[10:00:01] MediaAgent 开始工作...
[10:00:01] InsightAgent 开始工作...
[10:00:05] ForumEngine 论坛启动...
[10:00:30] 分析完成，开始生成报告...
[10:01:00] 报告生成完成！
```

### 3. 查看报告

分析完成后，你可以：

- **在线查看**：直接在浏览器中查看 HTML 报告
- **下载 PDF**：点击下载按钮获取 PDF 文件（需配置 PDF 导出依赖）
- **导出 Markdown**：获取 .md 格式的原始内容

## 常用命令

### 完整系统

```bash
# 启动完整系统
python app.py

# 后台启动
nohup python app.py > app.log 2>&1 &

# 停止：Ctrl + C 或 kill <pid>
```

### 单独启动某个 Agent（调试用）

```bash
# QueryEngine
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503

# MediaEngine
streamlit run SingleEngineApp/media_engine_streamlit_app.py --server.port 8502

# InsightEngine
streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501
```

### 爬虫系统

```bash
cd MindSpider

# 项目初始化
python main.py --setup

# 运行话题提取
python main.py --broad-topic

# 运行完整爬虫流程
python main.py --complete --date 2024-01-20

# 仅运行深度爬取
python main.py --deep-sentiment --platforms xhs dy wb
```

### 报告生成工具

```bash
# 基本使用
python report_engine_only.py

# 指定主题
python report_engine_only.py --query "土木工程行业分析"

# 跳过 PDF 生成
python report_engine_only.py --skip-pdf

# 跳过 Markdown 生成
python report_engine_only.py --skip-markdown

# 显示详细日志
python report_engine_only.py --verbose

# 启用 GraphRAG
python report_engine_only.py --graphrag-enabled true --graphrag-max-queries 3
```

### 重渲染工具

```bash
# 重渲染最新 HTML
python regenerate_latest_html.py

# 重渲染最新 Markdown
python regenerate_latest_md.py

# 重渲染最新 PDF
python regenerate_latest_pdf.py
```

## 验证安装

### 检查清单

- [ ] Python 版本 ≥ 3.9
- [ ] 虚拟环境已激活
- [ ] 依赖包已安装
- [ ] Playwright 已安装
- [ ] 数据库服务运行正常
- [ ] .env 文件已配置
- [ ] 至少一个 LLM API Key 已配置

### 健康检查脚本

```bash
# 一键检查
python -c "
import sys
print(f'Python: {sys.version}')

try:
    import flask
    print('✓ Flask 已安装')
except ImportError:
    print('✗ Flask 未安装')

try:
    import loguru
    print('✓ Loguru 已安装')
except ImportError:
    print('✗ Loguru 未安装')

try:
    from config import settings
    print(f'✓ 配置加载成功')
except Exception as e:
    print(f'✗ 配置加载失败: {e}')
"
```

### 检查服务状态

```bash
# 检查 Web 服务
curl http://localhost:5000

# 检查日志
tail -f logs/report.log

# 检查数据库连接
python -c "
from config import settings
from InsightEngine.utils.db import test_connection
test_connection()
"
```

## 故障排查

### 问题 1：端口被占用

**症状**：
```
Address already in use: 5000
```

**解决方案**：

```bash
# 查找占用进程
lsof -i :5000  # Linux/macOS
netstat -ano | findstr :5000  # Windows

# 结束进程
kill -9 <PID>  # Linux/macOS
taskkill /PID <PID> /F  # Windows

# 或修改端口
# .env 文件中设置 PORT=5001
```

### 问题 2：LLM 连接失败

**症状**：
```
openai.APIError: Connection error
```

**解决方案**：

1. 检查 API Key 是否正确
2. 检查网络连接
3. 尝试更换 Base URL
4. 验证 API 额度是否用完

```bash
# 测试 API 连接（以 DeepSeek 为例）
curl https://api.deepseek.com/v1/models \
  -H "Authorization: Bearer $QUERY_ENGINE_API_KEY"
```

### 问题 3：数据库连接失败

**症状**：
```
psycopg2.OperationalError: could not connect to server
```

**解决方案**：

1. 确认数据库服务运行正常
2. 检查连接参数是否正确
3. 查看防火墙设置

```bash
# PostgreSQL
sudo systemctl status postgresql  # Linux
brew services list | grep postgresql  # macOS

# 启动数据库
sudo systemctl start postgresql
```

### 问题 4：依赖安装失败

**症状**：
```
ERROR: Could not build wheels for xxx
```

**解决方案**：

```bash
# 更新 pip
pip install --upgrade pip setuptools wheel

# 使用国内镜像源
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 或分步安装，跳过有问题的包
# 编辑 requirements.txt，注释掉报错的包
```

### 问题 5：Playwright 安装失败

**症状**：
```
playwright install 失败
```

**解决方案**：

```bash
# 手动安装浏览器
playwright install chromium --with-deps

# 使用国内镜像
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright/
playwright install chromium
```

## 下一步

- **[核心概念](concepts.md)** ⭐ - 理解 Agent、论坛机制等核心概念
- **[配置指南](../development/config.md)** ⭐⭐ - 深入了解系统配置
- **[各引擎文档](../engines/)** - 学习各分析引擎的详细用法

## 实践练习

### 练习 1：验证部署 ⭐

**任务**：完成系统部署并验证所有组件正常

**检查项**：
- [ ] 访问 http://localhost:5000 可以看到主界面
- [ ] 在搜索框输入测试文本，界面正常响应
- [ ] 查看 logs/ 目录，确认日志文件已创建

### 练习 2：测试单个 Agent ⭐⭐

**任务**：单独启动 QueryEngine 并测试

```bash
# 启动 QueryEngine
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503

# 在浏览器中访问 http://localhost:8503
# 输入测试查询并观察结果
```

### 练习 3：完整分析流程 ⭐⭐⭐

**任务**：完成一次完整的分析流程

1. 提交一个你感兴趣的话题
2. 观察 Agent 的工作过程
3. 查看生成的报告
4. 记录分析时长和结果质量

---

> 💡 **学习提示**：如果遇到问题，请先查看[故障排查指南](../development/troubleshooting.md)，大部分常见问题都有解决方案。
