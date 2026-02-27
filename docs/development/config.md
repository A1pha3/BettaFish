# 配置指南

BettaFish 使用统一的配置管理系统，所有配置通过环境变量和 `.env` 文件管理。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 BettaFish 的**配置架构和优先级**
- [ ] 正确配置**数据库连接**
- [ ] 配置多个 **LLM 引擎**（Query、Media、Insight、Report）
- [ ] 配置**搜索 API**（Tavily、Bocha、Anspire）
- [ ] 根据**不同场景**优化配置参数
- [ ] 验证配置的正确性

## 概述

### 配置架构

```
.env 文件
    ↓
config.py (Pydantic Settings)
    ↓
各子模块
    ↓
运行时配置
```

### 配置优先级

1. 环境变量（最高优先级）
2. `.env` 文件
3. 默认值（代码中定义）

## 配置文件

### .env 文件

主配置文件位于项目根目录：

```bash
# 复制模板
cp .env.example .env

# 编辑配置
nano .env
```

### 环境变量分组

```bash
# ================== Flask 服务器配置 ====================
HOST=0.0.0.0
PORT=5000

# ================== 数据库配置 ======================
DB_DIALECT=postgresql
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=your_password
DB_NAME=bettafish

# ================== LLM 配置 ======================
# Insight Agent
INSIGHT_ENGINE_API_KEY=your_key
INSIGHT_ENGINE_BASE_URL=https://api.moonshot.cn/v1
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview

# Media Agent
MEDIA_ENGINE_API_KEY=your_key
MEDIA_ENGINE_BASE_URL=https://aihubmix.com/v1
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro

# Query Agent
QUERY_ENGINE_API_KEY=your_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat

# Report Agent
REPORT_ENGINE_API_KEY=your_key
REPORT_ENGINE_BASE_URL=https://aihubmix.com/v1
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro

# Forum Host
FORUM_HOST_API_KEY=your_key
FORUM_HOST_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
FORUM_HOST_MODEL_NAME=qwen-plus

# Keyword Optimizer
KEYWORD_OPTIMIZER_API_KEY=your_key
KEYWORD_OPTIMIZER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
KEYWORD_OPTIMIZER_MODEL_NAME=qwen-plus

# ================== 搜索 API 配置 ==================
TAVILY_API_KEY=your_key
SEARCH_TOOL_TYPE=AnspireAPI
BOCHA_WEB_SEARCH_API_KEY=your_key
ANSPIRE_API_KEY=your_key

# ================== GraphRAG 配置 ==================
GRAPHRAG_ENABLED=false
GRAPHRAG_MAX_QUERIES=3

# ================== 搜索限制配置 ==================
DEFAULT_SEARCH_HOT_CONTENT_LIMIT=100
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=50
DEFAULT_SEARCH_TOPIC_BY_DATE_LIMIT_PER_TABLE=100
DEFAULT_GET_COMMENTS_FOR_TOPIC_LIMIT=500
DEFAULT_SEARCH_TOPIC_ON_PLATFORM_LIMIT=200
MAX_SEARCH_RESULTS_FOR_LLM=0
MAX_REFLECTIONS=3
MAX_PARAGRAPHS=6
```

## 详细配置说明

### 数据库配置

#### PostgreSQL

```bash
DB_DIALECT=postgresql
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=your_password
DB_NAME=bettafish
DB_CHARSET=utf8mb4
```

#### MySQL

```bash
DB_DIALECT=mysql
DB_HOST=localhost
DB_PORT=3306
DB_USER=bettafish
DB_PASSWORD=your_password
DB_NAME=bettafish
DB_CHARSET=utf8mb4
```

### LLM 配置

#### Insight Agent

负责数据库挖掘和情感分析，推荐使用长上下文模型。

```bash
# 推荐：Moonshot Kimi
INSIGHT_ENGINE_API_KEY=your_api_key
INSIGHT_ENGINE_BASE_URL=https://api.moonshot.cn/v1
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview
```

#### Media Agent

负责多模态分析，推荐使用视觉能力强的模型。

```bash
# 推荐：Gemini
MEDIA_ENGINE_API_KEY=your_api_key
MEDIA_ENGINE_BASE_URL=https://aihubmix.com/v1
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro
```

#### Query Agent

负责网络搜索，推荐使用响应快的模型。

```bash
# 推荐：DeepSeek
QUERY_ENGINE_API_KEY=your_api_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

#### Report Agent

负责报告生成，推荐使用多轮对话能力强的模型。

```bash
# 推荐：Gemini
REPORT_ENGINE_API_KEY=your_api_key
REPORT_ENGINE_BASE_URL=https://aihubmix.com/v1
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro
```

#### Forum Host

负责论坛主持，推荐使用对话能力强的模型。

```bash
# 推荐：Qwen
FORUM_HOST_API_KEY=your_api_key
FORUM_HOST_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
FORUM_HOST_MODEL_NAME=qwen-plus
```

#### Keyword Optimizer

负责优化搜索关键词。

```bash
# 推荐：Qwen
KEYWORD_OPTIMIZER_API_KEY=your_api_key
KEYWORD_OPTIMIZER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
KEYWORD_OPTIMIZER_MODEL_NAME=qwen-plus
```

### 搜索 API 配置

#### Tavily API

用于 QueryEngine 的网络搜索。

```bash
# 申请地址：https://www.tavily.com/
TAVILY_API_KEY=your_api_key
```

#### Bocha API

用于 MediaEngine 的多模态搜索。

```bash
# 申请地址：https://open.bochaai.com/
BOCHA_BASE_URL=https://api.bocha.cn/v1/ai-search
BOCHA_WEB_SEARCH_API_KEY=your_api_key
```

#### Anspire API

通用搜索 API。

```bash
# 申请地址：https://open.anspire.cn/?share_code=3E1FUOUH
ANSPIRE_BASE_URL=https://plugin.anspire.cn/api/ntsearch/search
ANSPIRE_API_KEY=your_api_key
```

### 搜索限制配置

控制数据库查询和搜索的数量限制。

```bash
# 热点内容最大数
DEFAULT_SEARCH_HOT_CONTENT_LIMIT=100

# 全局搜索每表最大数
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=50

# 按日期搜索每表最大数
DEFAULT_SEARCH_TOPIC_BY_DATE_LIMIT_PER_TABLE=100

# 评论获取最大数
DEFAULT_GET_COMMENTS_FOR_TOPIC_LIMIT=500

# 平台搜索最大数
DEFAULT_SEARCH_TOPIC_ON_PLATFORM_LIMIT=200

# 传给 LLM 的最大结果数（0=不限制）
MAX_SEARCH_RESULTS_FOR_LLM=0

# 最大反思次数
MAX_REFLECTIONS=3

# 最大段落数
MAX_PARAGRAPHS=6
```

### GraphRAG 配置

知识图谱增强功能。

```bash
# 是否启用 GraphRAG
GRAPHRAG_ENABLED=false

# 每章节最大查询次数
GRAPHRAG_MAX_QUERIES=3
```

## 配置加载

### 代码中读取配置

```python
from config import settings

# 读取配置
api_key = settings.INSIGHT_ENGINE_API_KEY
model_name = settings.INSIGHT_ENGINE_MODEL_NAME

# 运行时重载
from config import reload_settings
settings = reload_settings()
```

### 子模块配置继承

子模块自动继承根目录配置：

```python
# InsightEngine/utils/config.py
from config import Settings as RootSettings

class Settings(RootSettings):
    """InsightEngine 继承根配置"""
    # 可以添加特定配置
    OUTPUT_DIR: str = "insight_engine_streamlit_reports"
```

## 推荐配置

### 本地开发

```bash
# 使用较便宜的模型
QUERY_ENGINE_MODEL_NAME=deepseek-chat
MEDIA_ENGINE_MODEL_NAME=gemini-2.0-flash
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview

# 降低搜索限制
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=20
MAX_REFLECTIONS=1
```

### 生产环境

```bash
# 使用最强的模型
QUERY_ENGINE_MODEL_NAME=deepseek-chat
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview

# 完整搜索
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=50
MAX_REFLECTIONS=3
```

### 快速测试

```bash
# 最小配置
MAX_REFLECTIONS=0
MAX_PARAGRAPHS=2
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=10
```

## 配置验证

### 检查配置

```bash
# Python 脚本检查
python -c "from config import settings; print(settings)"

# 检查必需配置
python -c "
from config import settings
required = ['DB_HOST', 'DB_NAME', 'QUERY_ENGINE_API_KEY']
for key in required:
    value = getattr(settings, key)
    if not value:
        print(f'Missing: {key}')
"
```

### 测试连接

```python
# 测试数据库
from InsightEngine.utils.db import test_connection
test_connection()

# 测试 LLM
from QueryEngine.llms import LLMClient
client = LLMClient(api_key="...", model_name="...")
client.generate("Hello")
```

## 常见问题

### Q1: 配置未生效？

**排查步骤**：

```bash
# 1. 确认 .env 文件位置
ls -la .env  # 应在项目根目录

# 2. 检查配置是否被读取
python -c "from config import settings; print(f'API Key: {settings.QUERY_ENGINE_API_KEY[:20]}...')"

# 3. 重启应用
# 配置修改后需要重启才能生效

# 4. 检查环境变量优先级
echo $QUERY_ENGINE_API_KEY  # 环境变量优先级更高
```

### Q2: 子模块无法读取配置？

**解决方案**：

```python
# 检查继承关系
# 子模块应该这样导入：

# 正确方式 ✅
from config import Settings as RootSettings

class Settings(RootSettings):
    pass

# 错误方式 ❌
# 不要在子模块中创建新的 Settings 类
# 这样会导致配置无法继承
```

### Q3: 如何动态更新配置？

```python
from config import reload_settings
from QueryEngine.llms import LLMClient

# 重载配置
new_settings = reload_settings()

# 更新客户端
agent.llm_client = LLMClient(
    api_key=new_settings.INSIGHT_ENGINE_API_KEY,
    model_name=new_settings.INSIGHT_ENGINE_MODEL_NAME,
    base_url=new_settings.INSIGHT_ENGINE_BASE_URL
)
```

## 实践练习

### 练习 1：配置验证 ⭐

**任务**：验证所有必需配置项是否正确设置

```python
# verify_config.py
from config import settings

def verify_config():
    """验证配置完整性"""
    required_configs = {
        "数据库": ["DB_HOST", "DB_NAME", "DB_USER"],
        "QueryEngine": ["QUERY_ENGINE_API_KEY", "QUERY_ENGINE_MODEL_NAME"],
        "数据库连接": "test_connection"
    }

    print("=== 配置验证 ===\n")

    # 检查数据库配置
    db_ok = all(hasattr(settings, attr) and getattr(settings, attr) for attr in required_configs["数据库"])
    print(f"数据库配置: {'✓' if db_ok else '✗'}")

    # 检查 QueryEngine 配置
    query_ok = all(hasattr(settings, attr) and getattr(settings, attr) for attr in required_configs["QueryEngine"])
    print(f"QueryEngine 配置: {'✓' if query_ok else '✗'}")

    # 测试数据库连接
    try:
        from InsightEngine.utils.db import test_connection
        test_connection()
        print("数据库连接: ✓")
    except Exception as e:
        print(f"数据库连接: ✗ ({e})")

    print("\n建议:")
    if not db_ok:
        print("- 请配置数据库相关环境变量")
    if not query_ok:
        print("- 请配置 QUERY_ENGINE_API_KEY")
    if query_ok and db_ok:
        print("- 配置检查通过，可以启动系统")

if __name__ == "__main__":
    verify_config()
```

### 练习 2：配置切换 ⭐⭐

**任务**：创建开发/生产环境配置切换脚本

```bash
#!/bin/bash
# switch_env.sh <env>

ENV=${1:-dev}

case $ENV in
    dev)
        echo "切换到开发环境..."
        cp .env.dev .env
        ;;
    prod)
        echo "切换到生产环境..."
        cp .env.prod .env
        ;;
    test)
        echo "切换到测试环境..."
        cp .env.test .env
        ;;
    *)
        echo "用法: $0 {dev|prod|test}"
        exit 1
        ;;
esac

echo "已切换到 $ENV 环境"
echo "当前配置:"
grep "ENGINE_MODEL_NAME" .env
```

### 练习 3：配置优化 ⭐⭐⭐

**任务**：根据不同场景优化配置

<details>
<summary>查看完整代码与使用示例</summary>

```python
# optimize_config.py

SCENARIOS = {
    "快速测试": {
        "MAX_REFLECTIONS": 0,
        "MAX_PARAGRAPHS": 2,
        "DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE": 10,
        "INSIGHT_ENGINE_MODEL_NAME": "deepseek-chat"  # 更便宜的模型
    },
    "标准分析": {
        "MAX_REFLECTIONS": 2,
        "MAX_PARAGRAPHS": 4,
        "DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE": 30,
        "INSIGHT_ENGINE_MODEL_NAME": "kimi-k2-0711-preview"
    },
    "深度分析": {
        "MAX_REFLECTIONS": 3,
        "MAX_PARAGRAPHS": 6,
        "DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE": 50,
        "INSIGHT_ENGINE_MODEL_NAME": "kimi-k2-0711-preview",
        "GRAPHRAG_ENABLED": "true",
        "GRAPHRAG_MAX_QUERIES": 5
    }
}

def apply_scenario(scenario_name: str):
    """应用场景配置

    Args:
        scenario_name: 场景名称
    """
    if scenario_name not in SCENARIOS:
        print(f"未知场景: {scenario_name}")
        print(f"可用场景: {', '.join(SCENARIOS.keys())}")
        return

    config = SCENARIOS[scenario_name]

    print(f"应用 [{scenario_name}] 配置:")
    for key, value in config.items():
        print(f"  {key} = {value}")

    # 写入 .env 文件
    with open('.env', 'a') as f:
        f.write(f"\n# {scenario_name} 配置\n")
        for key, value in config.items():
            f.write(f"{key}={value}\n")

    print(f"\n配置已更新到 .env 文件")

if __name__ == "__main__":
    import sys
    scenario = sys.argv[1] if len(sys.argv) > 1 else "快速测试"
    apply_scenario(scenario)
```

**使用示例**：
```bash
# 应用快速测试配置
python optimize_config.py 快速测试

# 应用深度分析配置
python optimize_config.py 深度分析
```

**输出示例**：
```
应用 [快速测试] 配置:
  MAX_REFLECTIONS = 0
  MAX_PARAGRAPHS = 2
  DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE = 10
  INSIGHT_ENGINE_MODEL_NAME = deepseek-chat

配置已更新到 .env 文件
```

</details>

---

> 💡 **学习提示**：配置管理是系统运行的基础。建议先用最小配置验证系统可用，再逐步添加高级功能配置。

## 相关文档

- **[部署指南](deployment.md)** - 生产环境配置
- **[故障排查](troubleshooting.md)** - 配置问题排查
- **[快速开始](../getting-started/quickstart.md)** - 基础配置
