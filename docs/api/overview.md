# API 概览

BettaFish 提供了多层次的 API，支持不同场景的集成和扩展。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 BettaFish 的**三层 API 架构**
- [ ] 使用 **Web API** 进行 HTTP 调用
- [ ] 建立 **WebSocket 连接**获取实时推送
- [ ] 使用 **Python API** 进行二次开发
- [ ] 理解 **Core API** 的节点和状态机制
- [ ] 实施 **API 认证和限流**

## API 层次

```
┌─────────────────────────────────────────────────────────────┐
│                        BettaFish API 层次                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Web API (Flask + SocketIO)                         │    │
│  │  - HTTP 接口                                         │    │
│  │  - WebSocket 实时推送                                │    │
│  │  - 文件上传/下载                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ▲                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Python API (模块级)                                │    │
│  │  - Agent 类                                         │    │
│  │  - 工具函数                                         │    │
│  │  - 数据访问                                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                         ▲                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Core API (内部)                                    │    │
│  │  - Node 接口                                        │    │
│  │  - State 管理                                       │    │
│  │  - IR 操作                                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Web API

### 基础信息

- **Base URL**: `http://localhost:5000`
- **协议**: HTTP + WebSocket (SocketIO)
- **格式**: JSON

### 端点列表

#### 分析相关

| 方法 | 端点 | 说明 |
|------|------|------|
| POST | `/api/analyze` | 提交分析任务 |
| GET | `/api/status/:task_id` | 获取任务状态 |
| GET | `/api/result/:task_id` | 获取分析结果 |

#### 报告相关

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | `/api/report/templates` | 获取模板列表 |
| POST | `/api/report/generate` | 生成报告 |
| GET | `/api/report/:run_id` | 获取报告内容 |
| GET | `/api/report/:run_id/download` | 下载报告 |

#### 配置相关

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | `/api/config` | 获取配置 |
| PUT | `/api/config` | 更新配置 |

### API 示例

#### 提交分析任务

```bash
curl -X POST http://localhost:5000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "query": "分析武汉大学品牌声誉",
    "engines": ["query", "media", "insight"]
  }'
```

**响应**：
```json
{
  "task_id": "task_20240120_120000",
  "status": "pending",
  "message": "任务已提交"
}
```

#### 获取任务状态

```bash
curl http://localhost:5000/api/status/task_20240120_120000
```

**响应**：
```json
{
  "task_id": "task_20240120_120000",
  "status": "running",
  "progress": {
    "query": "completed",
    "media": "running",
    "insight": "pending"
  }
}
```

#### 生成报告

```bash
curl -X POST http://localhost:5000/api/report/generate \
  -H "Content-Type: application/json" \
  -d '{
    "query": "武汉大学品牌声誉",
    "template": "企业品牌声誉分析报告"
  }'
```

### WebSocket 连接

```javascript
// 连接 WebSocket
const socket = io('http://localhost:5000');

// 监听日志事件
socket.on('log', (data) => {
  console.log(`[${data.engine}] ${data.message}`);
});

// 监听进度事件
socket.on('progress', (data) => {
  console.log(`进度: ${data.percent}%`);
});

// 监听完成事件
socket.on('complete', (data) => {
  console.log('分析完成', data);
});
```

## Python API

### Agent API

#### QueryEngine

```python
from QueryEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索
report = agent.research(
    query="分析主题",
    save_report=True
)

# 获取进度
progress = agent.get_progress_summary()
```

#### MediaEngine

```python
from MediaEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行分析
report = agent.research(
    query="分析主题",
    save_report=True
)
```

#### InsightEngine

```python
from InsightEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索工具
response = agent.execute_search_tool(
    tool_name="search_topic_globally",
    query="搜索关键词"
)

# 独立情感分析
sentiment = agent.analyze_sentiment_only("分析文本")
```

### ReportEngine API

```python
from ReportEngine import create_report_agent

# 创建 Agent
agent = create_report_agent()

# 加载输入文件
agent.load_input_files(directories={
    'insight': 'insight_engine_streamlit_reports',
    'media': 'media_engine_streamlit_reports',
    'query': 'query_engine_streamlit_reports'
})

# 生成报告
document_ir = agent.generate_report()

# 渲染报告
html = agent.renderer.render(document_ir)
```

### 数据库 API

```python
from InsightEngine.utils.db import execute_query

# 执行查询
results = execute_query(
    "SELECT * FROM xhs_note WHERE title LIKE %s LIMIT 10",
    params=['%关键词%']
)

# 使用 ORM
from MindSpider.schema.models_bigdata import XhsNote
from sqlalchemy import create_engine

engine = create_engine('postgresql://...')
session = sessionmaker(bind=engine)()
notes = session.query(XhsNote).limit(10).all()
```

### 工具函数 API

#### Forum Reader

```python
from utils.forum_reader import (
    get_latest_host_speech,
    get_all_host_speeches,
    get_recent_agent_speeches
)

# 获取最新主持人发言
host_speech = get_latest_host_speech()

# 获取最近的 Agent 发言
recent_speeches = get_recent_agent_speeches(limit=10)
```

#### Retry Helper

```python
from utils.retry_helper import with_retry, LLM_RETRY_CONFIG

@with_retry(**LLM_RETRY_CONFIG)
def llm_function():
    # LLM 调用逻辑
    pass
```

## Core API

### Node API

所有节点都继承自 `BaseNode`：

```python
from nodes.base_node import BaseNode

class CustomNode(BaseNode):
    def run(self, input_data: Dict, state: State) -> Dict:
        """
        节点处理逻辑

        Args:
            input_data: 输入数据
            state: 当前状态

        Returns:
            处理结果字典
        """
        # 实现处理逻辑
        return result
```

### State API

```python
from state import State

# 创建状态
state = State(query="分析主题")

# 更新状态
state.add_paragraph(title="章节标题", content="内容要求")

# 获取进度
progress = state.get_progress_summary()
```

### IR API

```python
from ir.schema import DocumentIR, BlockTypes

# 创建 IR
document = {
    "title": "报告标题",
    "metadata": {...},
    "sections": [
        {"type": "heading", "level": 1, "content": "标题"},
        {"type": "paragraph", "content": "内容"}
    ]
}

# 验证 IR
from ir.validator import IRValidator
validator = IRValidator()
result = validator.validate_document(document)
```

## 扩展开发

### 添加新 Agent

```python
# 新建 NewEngine/
# 复制现有引擎结构
# 修改工具集
# 实现 Agent 类
```

### 添加新工具

```python
# Engine/tools/new_tool.py

class NewTool:
    def execute(self, query: str, **kwargs):
        """工具逻辑"""
        pass

# 在 __init__.py 中导出
```

### 添加新节点

```python
# Engine nodes/new_node.py

from nodes.base_node import BaseNode

class NewNode(BaseNode):
    def run(self, input_data: Dict) -> Dict:
        """节点逻辑"""
        pass
```

## API 认证

### 当前版本

当前版本未实现认证，API 完全开放。

### 生产环境

建议在生产环境添加认证：

```python
from flask_httpauth import HTTPBasicAuth

auth = HTTPBasicAuth()

@auth.verify_password
def verify_password(username, password):
    return check_auth(username, password)

@app.route('/api/analyze')
@auth.login_required
def analyze():
    pass
```

## 限流

### 实现限流

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/analyze')
@limiter.limit("10 per hour")
def analyze():
    pass
```

## 错误处理

### 标准错误格式

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "请求参数无效",
    "details": {...}
  }
}
```

### 错误代码

| 代码 | 说明 |
|------|------|
| `INVALID_REQUEST` | 请求参数无效 |
| `AUTH_FAILED` | 认证失败 |
| `RATE_LIMIT_EXCEEDED` | 超过限流 |
| `INTERNAL_ERROR` | 内部错误 |
| `ENGINE_UNAVAILABLE` | 引擎不可用 |

## 实践练习

### 练习 1：Web API 调用 ⭐

**任务**：使用 curl 测试 BettaFish Web API

```bash
# 1. 提交分析任务
TASK_ID=$(curl -s -X POST http://localhost:5000/api/analyze \
  -H "Content-Type: application/json" \
  -d '{"query":"测试分析","engines":["query","media","insight"]}' \
  | jq -r '.task_id')

echo "任务ID: $TASK_ID"

# 2. 查询任务状态
sleep 5
curl http://localhost:5000/api/status/$TASK_ID | jq

# 3. 获取模板列表
curl http://localhost:5000/api/report/templates | jq

# 4. 生成报告
curl -X POST http://localhost:5000/api/report/generate \
  -H "Content-Type: application/json" \
  -d '{"query":"测试分析"}' | jq
```

### 练习 2：WebSocket 连接 ⭐⭐

**任务**：创建一个 WebSocket 客户端监听实时进度

```python
# websocket_client.py
import socketio
import time

# 创建客户端
sio = socketio.Client()

# 连接到服务器
sio.connect('http://localhost:5000')

# 监听日志
@sio.on('log')
def on_log(data):
    print(f"[{data['engine']}] {data['message']}")

# 监听进度
@sio.on('progress')
def on_progress(data):
    print(f"进度: {data.get('percent', 0)}%")

# 监听完成
@sio.on('complete')
def on_complete(data):
    print("分析完成!")
    print(f"任务ID: {data.get('task_id')}")
    sio.disconnect()

# 保持连接
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("\n断开连接")
    sio.disconnect()
```

### 练习 3：Python API 集成 ⭐⭐

**任务**：创建一个简单的分析脚本

```python
# simple_analysis.py
from QueryEngine import create_agent
from MediaEngine import create_agent
from InsightEngine import create_agent

def parallel_analysis(query: str):
    """并行分析

    Args:
        query: 分析查询
    """
    print(f"开始分析: {query}\n")

    # 创建三个 Agent
    query_agent = create_agent()
    media_agent = create_agent()
    insight_agent = create_agent()

    # 并行执行（简化版，实际可用线程）
    results = {}

    # QueryEngine 分析
    print("[1/3] QueryEngine 分析中...")
    results['query'] = query_agent.research(
        query=query,
        save_report=False
    )

    # MediaEngine 分析
    print("[2/3] MediaEngine 分析中...")
    results['media'] = media_agent.research(
        query=query,
        save_report=False
    )

    # InsightEngine 分析
    print("[3/3] InsightEngine 分析中...")
    results['insight'] = insight_agent.research(
        query=query,
        save_report=False
    )

    print("\n=== 分析完成 ===")
    for engine, result in results.items():
        # 提取前500字符作为预览
        preview = result[:500] if result else "无结果"
        print(f"\n{engine.upper()} 结果预览:")
        print(f"{preview}...")

if __name__ == "__main__":
    import sys
    query = sys.argv[1] if len(sys.argv) > 1 else "Python 编程"
    parallel_analysis(query)
```

## 常见问题

### Q1: API 返回 404 错误？

**原因**：端点路径不正确或服务未启动

**解决方案**：

```bash
# 1. 确认服务已启动
curl http://localhost:5000/

# 2. 检查正确的端点路径
# 正确: /api/analyze
# 错误: /api/analyzes /analyze

# 3. 查看可用端点
curl http://localhost:5000/api/
```

### Q2: WebSocket 连接断开？

**原因**：网络波动或服务重启

**解决方案**：

```python
import socketio

# 添加重连机制
sio = socketio.Client(reconnection=True, reconnection_attempts=5)

@sio.on('disconnect')
def on_disconnect():
    print("连接断开，尝试重连...")

@sio.on('connect')
def on_connect():
    print("连接成功")
```

### Q3: API 调用超时？

**原因**：分析任务执行时间过长

**解决方案**：

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# 配置重试和超时
session = requests.Session()
retry = Retry(total=3, backoff_factor=1)
adapter = HTTPAdapter(max_retries=retry)
session.mount('http://', adapter)

# 设置超时
response = session.post(
    'http://localhost:5000/api/analyze',
    json={'query': '测试'},
    timeout=(3, 300)  # 连接超时3秒，读取超时300秒
)
```

### Q4: 返回结果为空？

**原因**：数据库无数据或查询失败

**解决方案**：

```bash
# 1. 检查数据库是否有数据
python -c "
from InsightEngine.utils.db import execute_query
result = execute_query('SELECT COUNT(*) FROM xhs_note')
print(f'数据量: {result[0][0]}')
"

# 2. 检查引擎日志
tail -20 logs/insight.log
tail -20 logs/query.log

# 3. 尝试简单的测试查询
python -c "
from QueryEngine import create_agent
agent = create_agent()
result = agent.research('测试', save_report=False)
print(result[:200])
"
```

### Q5: 跨域问题 (CORS)？

**原因**：前端与后端域名不同

**解决方案**：

```python
# app.py
from flask_cors import CORS

# 启用 CORS
CORS(app, resources={
    r"/api/*": {
        "origins": "*",  # 生产环境应指定具体域名
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

## 相关文档

- **[配置指南](../development/config.md)** - API 配置
- **[部署指南](../development/deployment.md)** - 生产环境部署
- **[核心概念](../getting-started/concepts.md)** - 架构概念

---

> 💡 **学习提示**：BettaFish API 设计遵循 RESTful 原则，易于理解和集成。建议先通过 Web API 熟悉系统交互，再深入使用 Python API。
