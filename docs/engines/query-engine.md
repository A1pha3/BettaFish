# QueryEngine 查询引擎

QueryEngine 是 BettaFish 系统中的 Web 搜索 Agent，负责从国内外新闻和网页中获取信息。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 QueryEngine 的核心职责和工作流程
- [ ] 掌握 DeepSearchAgent 的使用方法
- [ ] 了解搜索节点（SearchNode、SummaryNode）的作用
- [ ] 熟练使用 TavilyNewsAgency 工具
- [ ] 能够独立运行和调试 QueryEngine

## 概述

### 核心职责

QueryEngine 专注于：

- **新闻搜索**：实时获取国内外新闻资讯
- **网页搜索**：检索相关网页内容
- **信息整合**：将搜索结果整理为结构化输出
- **时效性分析**：追踪最新动态和发展趋势

### 技术基础

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| 搜索 API | Tavily API | 支持 150+ 新闻源 |
| LLM | DeepSeek-Chat | 响应快，性价比高 |
| 架构模式 | Node Pattern | 模块化处理流程 |

## 架构设计

### 目录结构

```
QueryEngine/
├── __init__.py
├── agent.py                  # Agent 主类
├── nodes/                    # 处理节点
│   ├── base_node.py          # 基础节点
│   ├── search_node.py        # 搜索节点
│   ├── summary_node.py       # 总结节点
│   ├── formatting_node.py    # 格式化节点
│   └── report_structure_node.py
├── tools/                    # 工具集
│   └── search.py             # Tavily 搜索封装
├── llms/                     # LLM 客户端
├── state/                    # 状态管理
├── prompts/                  # 提示词模板
└── utils/                    # 工具函数
```

## 核心组件

### 1. DeepSearchAgent

QueryEngine 的主类，协调整个搜索流程。

```python
from QueryEngine import create_agent

# 创建 Agent 实例
agent = create_agent()

# Agent 内部初始化
self.llm_client = LLMClient(
    api_key=settings.QUERY_ENGINE_API_KEY,
    model_name=settings.QUERY_ENGINE_MODEL_NAME,
    base_url=settings.QUERY_ENGINE_BASE_URL
)
self.search_agency = TavilyNewsAgency()
self._initialize_nodes()
self.state = State()
```

### 2. 搜索节点

#### FirstSearchNode

负责分析段落需求，生成初始搜索查询。

```python
class FirstSearchNode(BaseNode):
    """首次搜索节点"""

    def run(self, search_input: Dict) -> Dict:
        """
        生成搜索查询

        输入示例：
        {
            "title": "市场概况",
            "content": "分析产品的市场表现和用户反馈"
        }

        输出示例：
        {
            "search_query": "产品市场表现 用户反馈 销售数据",
            "search_tool": "tavily_news",
            "reasoning": "需要了解市场反馈和用户评价"
        }
        """
```

#### ReflectionNode

基于当前总结生成反思搜索查询，进行更深入的分析。

```python
class ReflectionNode(BaseNode):
    """反思搜索节点"""

    def run(self, reflection_input: Dict) -> Dict:
        """
        生成反思搜索查询

        输入示例：
        {
            "title": "市场概况",
            "paragraph_latest_state": "初步分析显示产品反响良好..."
        }

        输出示例：
        {
            "search_query": "产品缺陷 用户投诉 负面评价",
            "reasoning": "需要平衡视角，了解存在的问题"
        }
        """
```

### 3. TavilyNewsAgency

Tavily API 的封装类，提供统一的搜索接口。

```python
class TavilyNewsAgency:
    """Tavily 搜索 API 封装"""

    def search_news(self, query: str, max_results: int = 10):
        """
        搜索新闻

        Args:
            query: 搜索关键词
            max_results: 最大结果数

        Returns:
            List[SearchResult]
        """

    def search_web(self, query: str, max_results: int = 10):
        """
        搜索网页

        Args:
            query: 搜索关键词
            max_results: 最大结果数

        Returns:
            List[SearchResult]
        """

    def comprehensive_search(self, query: str, max_results: int = 10):
        """
        综合搜索（新闻+网页）

        Args:
            query: 搜索关键词
            max_results: 每种类型的结果数

        Returns:
            {
                "news": [...],
                "web": [...]
            }
        """
```

## 工作流程

### 完整执行流程

```
┌──────────────────────────────────────────────────────────────┐
│                    QueryEngine 工作流程                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  用户输入 ──→ "分析某产品的市场表现"                          │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step 1: 生成报告结构                         │           │
│  │ - ReportStructureNode 分析任务               │           │
│  │ - 生成研究段落列表（如：市场概况、产品...） │     │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────┐            ┌──────────────────┐       │
│  │ Step 2-N: 处理每个段落（并行）          │       │
│  │                                        │       │
│  │  ┌────────────────────────────────────┐   │       │
│  │  │ Step 2.1: 首次搜索              │   │       │
│  │  │ - FirstSearchNode 生成查询      │   │       │
│  │  │ - TavilyNewsAgency 执行搜索     │   │       │
│  │  │ - FirstSummaryNode 生成总结     │   │       │
│  │  └────────────┬─────────────────────┘   │       │
│  │               │                          │       │
│  │  ┌────────────▼─────────────────────┐   │       │
│  │  │ Step 2.2: 反思循环（最多3轮）   │   │       │
│  │  │ - ReflectionNode 生成新查询     │   │       │
│  │  │ - 搜索并更新总结                │   │       │
│  │  └────────────┬─────────────────────┘   │       │
│  │               │                          │       │
│  └───────────────┼──────────────────────────┘       │
│                  │                                  │
│                  ▼                                  │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step N+1: 生成最终报告                       │           │
│  │ - ReportFormattingNode 整合所有段落         │           │
│  │ - 生成 Markdown 格式报告                    │           │
│  └────┬─────────────────────────────────────┬───┘           │
│      │                                     │                │
│      ▼                                     ▼                │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │ 保存报告     │                   │ 写入日志     │       │
│ │ query_*.md   │                   │ query.log    │       │
│ └──────────────┘                   └──────────────┘       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## 配置说明

### 环境变量配置

```bash
# QueryEngine LLM 配置（推荐 DeepSeek）
QUERY_ENGINE_API_KEY=your_api_key
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-chat

# Tavily API 配置（可选，用于 Web 搜索）
TAVILY_API_KEY=your_tavily_api_key
```

### 高级参数配置

```python
# QueryEngine/utils/config.py
class Config:
    max_reflections = 2           # 最大反思轮次（默认 2）
    max_search_results = 15       # 最大搜索结果数（默认 15）
    max_content_length = 8000     # 最大内容长度（默认 8000）
```

## 使用示例

### 基本使用

```python
from QueryEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索任务
report = agent.research(
    query="分析武汉大学的品牌声誉",
    save_report=True
)

print(report)
```

### 独立运行 Streamlit 应用

```bash
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503
```

### 代码示例：直接使用工具

```python
from QueryEngine.tools import TavilyNewsAgency

# 创建工具实例
tool = TavilyNewsAgency()

# 搜索新闻
results = tool.search_news(
    query="武汉大学",
    max_results=10
)

# 查看结果
for result in results:
    print(f"标题: {result.title}")
    print(f"URL: {result.url}")
    print(f"发布时间: {result.published_date}")
    print("---")
```

### 代码示例：自定义节点

```python
from nodes.base_node import BaseNode

class CustomSearchNode(BaseNode):
    """自定义搜索节点"""

    def run(self, input_data: Dict) -> Dict:
        """生成自定义搜索查询"""
        query = input_data.get("title", "")

        # 自定义逻辑：添加时间范围
        search_query = f"{query} 2024年"

        return {
            "search_query": search_query,
            "search_tool": "tavily_news",
            "reasoning": "添加了时间限制"
        }

# 使用自定义节点
custom_node = CustomSearchNode(llm_client)
result = custom_node.run({"title": "市场分析"})
```

## 与论坛交互

### 读取论坛内容

QueryEngine 可以读取论坛中其他 Agent 的发言：

```python
from utils.forum_reader import get_latest_host_speech

# 在节点中读取论坛
host_speech = get_latest_host_speech()

if host_speech:
    # 根据主持人建议调整搜索策略
    print(f"主持人建议: {host_speech}")
    # 调整搜索查询
```

### 写入论坛

QueryEngine 的总结会自动被 ForumEngine 捕获并写入论坛日志。

## 常见问题

### Q1: 搜索结果为空？

**可能原因**：
- API Key 无效
- 网络连接问题
- 搜索关键词过于具体

**解决方案**：

```python
# 1. 验证 API Key
import os
assert os.getenv("QUERY_ENGINE_API_KEY"), "API Key 未配置"

# 2. 测试连接
from QueryEngine.tools import TavilyNewsAgency
tool = TavilyNewsAgency()
results = tool.search_news("测试", max_results=1)
print(results)

# 3. 使用更通用的关键词
# 原来: "武汉大学2024年录取分数线具体是多少分"
# 改为: "武汉大学 录取分数线"
```

### Q2: LLM 响应缓慢？

**解决方案**：

1. 更换更快的模型
2. 减少搜索结果数量
3. 调整 `max_content_length` 参数

```bash
# .env 文件
QUERY_ENGINE_MODEL_NAME=deepseek-chat  # 使用 DeepSeek
MAX_CONTENT_LENGTH=4000  # 减少内容长度
```

### Q3: 报告质量不佳？

**优化策略**：

1. 增加反思轮次
2. 优化提示词模板
3. 使用更强大的 LLM

```python
# 增加反思轮次
from QueryEngine.utils.config import Config
Config.max_reflections = 3  # 从 2 增加到 3
```

## 性能优化

### 搜索优化

```python
# 控制结果数量
agent.research(
    query="分析主题",
    max_search_results=10  # 减少结果数
)

# 使用更快的模型
# 在 .env 中设置
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

### 缓存机制

```python
# 实现简单的缓存
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_search(query: str):
    return tool.search_news(query)
```

## 实践练习

### 练习 1：基础使用 ⭐

**任务**：创建一个 QueryEngine 实例并执行搜索

```python
from QueryEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索
report = agent.research(
    query="Python 3.12 新特性",
    save_report=False  # 不保存文件
)

# 打印报告标题
print("报告标题:", report[:100])
```

### 练习 2：自定义节点 ⭐⭐

**任务**：创建一个自定义搜索节点，添加时间限制

```python
from nodes.base_node import BaseNode

class TimeBoundSearchNode(BaseNode):
    """有时间限制的搜索节点"""

    def run(self, input_data: Dict) -> Dict:
        query = input_data.get("title", "")
        time_range = input_data.get("time_range", "recent")

        # 添加时间范围关键词
        time_keywords = {
            "recent": "最近",
            "past_week": "过去一周",
            "past_month": "过去一个月"
        }

        enhanced_query = f"{query} {time_keywords.get(time_range, '')}"

        return {
            "search_query": enhanced_query,
            "search_tool": "tavily_news",
            "reasoning": f"添加了时间限制: {time_range}"
        }

# 测试
node = TimeBoundSearchNode(llm_client)
result = node.run({"title": "AI技术", "time_range": "past_week"})
print(result)
```

### 练习 3：调试技巧 ⭐⭐⭐

**任务**：启用详细日志并追踪执行过程

```python
from loguru import logger

# 配置日志
logger.remove()  # 移除默认处理器
logger.add(lambda msg: print(f"[DEBUG] {msg}"), level="DEBUG")

# 创建 Agent 并执行
agent = create_agent()

# 观察日志输出
report = agent.research(query="测试查询")
```

## 扩展开发

### 添加新的搜索工具

```python
# 1. 在 tools/search.py 中添加新工具
class CustomSearchTool:
    def search_custom_source(self, query: str):
        """自定义搜索源"""
        # 实现搜索逻辑
        pass

# 2. 在 FirstSearchNode 中添加工具选择逻辑
# 3. 更新配置文件
```

### 自定义提示词

```python
# 在 prompts/prompts.py 中修改提示词模板
FIRST_SEARCH_PROMPT = """
你是一个专业的搜索专家。根据用户需求生成精确的搜索查询。

当前分析主题：{title}
分析要求：{content}

请生成搜索查询...
"""
```

## 相关文档

- **[核心概念](../getting-started/concepts.md)** - Agent 和节点通用概念
- **[MediaEngine](media-engine.md)** - 多模态引擎文档
- **[InsightEngine](insight-engine.md)** - 洞察引擎文档
- **[ForumEngine](../forum/forum-engine.md)** - 论坛协作机制

---

> 💡 **学习提示**：QueryEngine 是理解其他引擎的基础，建议先掌握 QueryEngine 再学习 MediaEngine 和 InsightEngine。
