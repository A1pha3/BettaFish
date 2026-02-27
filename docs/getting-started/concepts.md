# 核心概念

本文档介绍 BettaFish 系统的核心概念，帮助你更好地理解系统的工作原理。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 Agent、Node、State 等核心概念
- [ ] 掌握论坛协作机制的工作原理
- [ ] 了解报告中间表示（IR）的设计思想
- [ ] 理解情感分析和 GraphRAG 的应用
- [ ] 熟练使用系统的配置选项

## 1. Agent（智能体）⭐

### 什么是 Agent？

在 BettaFish 中，Agent 是一个具有特定能力和工具集的智能体，负责完成特定类型的分析任务。它不是"全能"的 AI，而是专精某一领域的"专家"。

### 三大分析 Agent

| Agent | 专长 | 工具集 | 核心能力 |
|-------|------|--------|----------|
| **QueryAgent** | Web 和新闻搜索 | TavilyNewsAgency | 国内外新闻/网页搜索 |
| **MediaAgent** | 多模态内容理解 | BochaMultimodalSearch | 视频/图片理解，多模态搜索 |
| **InsightAgent** | 数据库深度挖掘 | MediaCrawlerDB | 私有数据查询，情感分析 |

### Agent 的构成

每个 Agent 都采用相同的架构模式：

```
Agent/
├── agent.py          # Agent 主类（协调器）
├── nodes/            # 处理节点（执行单元）
│   ├── base_node.py         # 节点基类
│   ├── search_node.py       # 搜索节点（生成查询）
│   ├── summary_node.py      # 总结节点（整合结果）
│   ├── formatting_node.py   # 格式化节点
│   └── report_structure_node.py  # 报告结构节点
├── tools/            # 工具集（与外部交互）
├── llms/             # LLM 客户端（统一接口）
├── state/            # 状态管理（数据容器）
└── prompts/          # 提示词模板（指令设计）
```

### Agent 工作流程

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ 接收任务 │ → │ 报告结构 │ → │ 首次搜索 │ → │ 生成总结 │ → │ 反思循环 │ → │ 最终报告 │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
                    │              │              │              │
                    ▼              ▼              ▼              ▼
              分析查询      生成查询      执行工具      更新总结      整合内容
```

**流程说明**：

1. **报告结构**：分析任务，分解为多个研究段落
2. **首次搜索**：为每个段落生成初始搜索查询
3. **生成总结**：整合搜索结果，生成初步总结
4. **反思循环**：基于总结进行深入搜索（最多 3 轮）
5. **最终报告**：整合所有段落总结

## 2. 节点（Node）⭐⭐

### 什么是节点？

节点是 Agent 中的处理单元，每个节点负责一个特定的处理步骤。节点模式让 Agent 的处理流程更加模块化和可维护。

### 基础节点类型

| 节点 | 职责 | 输入 | 输出 |
|------|------|------|------|
| **FirstSearchNode** | 执行首次搜索 | 段落信息 | 搜索查询 + 工具选择 |
| **FirstSummaryNode** | 生成初始总结 | 搜索结果 | 段落总结 |
| **ReflectionNode** | 生成反思查询 | 当前总结 | 新的搜索查询 |
| **ReflectionSummaryNode** | 更新总结 | 反思搜索结果 | 更新后的总结 |
| **ReportFormattingNode** | 格式化报告 | 所有段落 | 最终报告 |

### 节点接口

所有节点都继承自 `BaseNode`，实现统一的接口：

```python
from nodes.base_node import BaseNode

class CustomNode(BaseNode):
    """自定义节点示例"""

    def run(self, input_data: Dict, state: State = None) -> Dict:
        """
        节点处理逻辑

        Args:
            input_data: 输入数据字典
            state: Agent 当前状态

        Returns:
            处理结果字典
        """
        # 1. 处理输入
        # 2. 执行逻辑
        # 3. 返回结果
        return result
```

### 为什么使用节点？

| 优势 | 说明 |
|------|------|
| **模块化** | 每个节点职责单一，易于测试和维护 |
| **可组合** | 节点可以灵活组合成不同的处理流程 |
| **可扩展** | 添加新功能只需新增节点，不影响现有代码 |
| **可重用** | 通用节点可以在不同 Agent 间复用 |

### 节点执行示例

```python
# 示例：FirstSearchNode 的执行过程

input_data = {
    "title": "市场概况",
    "content": "分析某产品的市场表现"
}

# 节点处理
node = FirstSearchNode(llm_client)
output = node.run(input_data)

# 输出
# {
#     "search_query": "产品市场表现 消费者反馈",
#     "search_tool": "search_topic_globally",
#     "reasoning": "需要了解市场反馈和用户评价"
# }
```

## 3. 论坛机制（Forum）⭐⭐⭐

### 什么是论坛机制？

论坛机制是 BettaFish 的核心创新，它让多个 Agent 能够：

1. **独立工作**：每个 Agent 使用自己的工具集进行分析
2. **分享见解**：将分析结果写入共享日志
3. **相互学习**：阅读其他 Agent 的发现
4. **主持人引导**：LLM 主持人总结争议点并引导深入讨论

### 论坛工作流程

```
QueryAgent 总结 ──┐
                   ├──→ forum.log ←── HOST 发言 ←──┐
MediaAgent 总结 ──┤                                │
                   └──→ Agent 读取论坛内容 ←─────────┘
InsightAgent 总结 ──┘
```

### 主持人模型

每 5 条 Agent 发言后，系统会：

1. 收集最近的 Agent 发言
2. 调用 LLM 生成主持人发言
3. 主持人总结争议点、提出新视角
4. 引导 Agent 进行更深入的讨论

### 主持人发言示例

```
[HOST] 三位专家的分析都很有价值。QueryAgent 发现了新闻曝光度的上升，
MediaAgent 展示了社交媒体上的视觉内容，InsightAgent 则提供了深入的
用户情感数据。建议下一轮重点关注：1) 负面舆情的具体来源；2)
不同平台用户反馈的差异。请各位专家针对这些方向进行更深入的分析。
```

### 论坛日志格式

``[时间] [QUERY] QueryAgent 的分析发现...
[时间] [MEDIA] MediaAgent 的分析发现...
[时间] [INSIGHT] InsightAgent 的分析发现...
[时间] [HOST] 主持人总结：三位 Agent 都关注了...
```

### Agent 如何读取论坛？

```python
from utils.forum_reader import get_latest_host_speech

# 在节点中读取论坛
host_speech = get_latest_host_speech()

if host_speech:
    # 根据主持人建议调整搜索策略
    search_query = adjust_query_based_on_host(original_query, host_speech)
```

## 4. 报告中间表示（IR）⭐⭐⭐

### 什么是 IR？

IR（Intermediate Representation）是报告的中间表示格式，它是报告生成过程中的核心数据结构。

### 为什么使用 IR？

| 优势 | 说明 | 对比传统方案 |
|------|------|-------------|
| **格式无关** | 可渲染为多种格式 | 传统方案直接绑定输出格式 |
| **结构化** | 便于程序处理和验证 | 传统方案格式不统一 |
| **可扩展** | 易于添加新的块类型 | 传统方案扩展困难 |

### IR 的块类型

| 块类型 | 说明 | 渲染示例 |
|--------|------|----------|
| `heading` | 标题 | `<h2>标题</h2>` / `## 标题` |
| `paragraph` | 段落 | `<p>内容</p>` / `内容` |
| `chart` | 图表 | Plotly 图表 / ![](chart.png) |
| `image` | 图片 | `<img src="...">` / `![](image.png)` |
| `list` | 列表 | `<ul><li>...</li></ul>` / `- ...` |
| `quote` | 引用 | `<blockquote>...</blockquote>` / `> ...` |
| `table` | 表格 | `<table>...</table>` / Markdown 表格 |
| `code` | 代码块 | `<pre>...</pre>` / ` ```...``` |
| `divider` | 分隔线 | `<hr>` / `---` |

### IR 示例

```json
{
  "title": "武汉大学品牌声誉分析报告",
  "metadata": {
    "generated_at": "2024-01-20T10:00:00",
    "query": "武汉大学品牌声誉",
    "total_blocks": 50
  },
  "sections": [
    {
      "type": "heading",
      "level": 2,
      "id": "section-1",
      "content": "1. 总体概况"
    },
    {
      "type": "paragraph",
      "content": "根据综合分析，武汉大学的品牌形象整体良好..."
    },
    {
      "type": "chart",
      "chart_type": "line",
      "data": {
        "data": [{"x": "2024-01", "y": 75}, ...],
        "layout": {...}
      },
      "title": "情感趋势图"
    }
  ]
}
```

## 5. 工具集（Tools）⭐

### 什么是工具集？

工具集是 Agent 与外部世界交互的接口，每个 Agent 都有自己的专属工具。

### 工具类型

| Agent | 工具集 | 主要功能 |
|-------|--------|----------|
| QueryAgent | TavilyNewsAgency | 国内外新闻/网页搜索 |
| MediaAgent | BochaMultimodalSearch | 多模态内容搜索 |
| InsightAgent | MediaCrawlerDB | 数据库查询与情感分析 |

### 工具接口

所有工具都遵循统一的接口规范：

```python
def tool_function(query: str, **kwargs) -> Response:
    """
    执行工具功能

    Args:
        query: 查询字符串
        **kwargs: 额外参数

    Returns:
        Response: 统一的响应格式
        {
            "tool_name": str,
            "parameters": dict,
            "results": list,
            "results_count": int
        }
    """
    pass
```

## 6. 状态管理（State）⭐

### 什么是状态？

状态是 Agent 在执行过程中的数据容器，记录了搜索历史、总结内容、进度等信息。

### 状态结构

```python
class State:
    query: str                    # 查询内容
    report_title: str             # 报告标题
    paragraphs: List[Paragraph]   # 段落列表
        ├── title: str             # 段落标题
        ├── content: str           # 段落要求
        └── research: ResearchState # 研究状态
            ├── search_history: List
            ├── latest_summary: str
            └── is_completed: bool
    search_history: List          # 搜索历史
    final_report: str             # 最终报告
    status: str                   # 执行状态
```

### 状态的作用

| 作用 | 说明 |
|------|------|
| **持久化** | 支持中断后继续执行 |
| **追踪** | 记录完整的执行过程 |
| **调试** | 便于问题排查 |

## 7. 情感分析 ⭐⭐

### 什么是情感分析？

InsightAgent 集成了多种情感分析模型，可以自动分析文本的情感倾向。

### 支持的模型

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| WeiboMultilingualSentiment | 支持 22 种语言 | 多语言内容 |
| BertChinese-Lora | BERT 微调模型 | 中文精准分析 |
| GPT2-Lora | GPT-2 微调模型 | 生成式分析 |
| 传统机器学习 | SVM、XGBoost | 快速分析 |

### 情感分析结果

```json
{
  "sentiment": "positive",
  "confidence": 0.95,
  "scores": {
    "positive": 0.95,
    "neutral": 0.03,
    "negative": 0.02
  }
}
```

### 使用示例

```python
from InsightEngine import create_agent

agent = create_agent()

# 执行带情感分析的搜索
response = agent.execute_search_tool(
    tool_name="search_topic_globally",
    query="武汉大学",
    enable_sentiment=True  # 启用情感分析
)

# 查看情感分析结果
sentiment = response.parameters.get("sentiment_analysis")
print(f"正面: {sentiment['positive']}%")
print(f"负面: {sentiment['negative']}%")
```

## 8. 模板系统 ⭐

### 什么是模板？

模板是报告的结构框架，定义了报告的章节结构和内容要求。

### 内置模板

- 企业品牌声誉分析报告
- 市场竞争格局舆情分析报告
- 日常或定期舆情监测报告
- 特定政策或行业动态舆情分析报告
- 社会公共热点事件分析报告
- 突发事件与危机公关舆情报告

### 模板格式

模板使用 Markdown 格式，以 `## ` 作为章节分隔：

```markdown
# 报告标题

## 第一章 总体概况

本章内容要求...

## 第二章 详细分析

本章内容要求...
```

## 9. GraphRAG ⭐⭐⭐

### 什么是 GraphRAG？

GraphRAG（Graph Retrieval Augmented Generation）是基于知识图谱的检索增强生成，用于提升报告的知识密度和准确性。

### GraphRAG 工作流程

```
State/Forum 日志 → 知识图谱构建 → 图检索 → 上下文增强 → 报告生成
```

### GraphRAG 配置

```bash
# .env 配置
GRAPHRAG_ENABLED=true    # 启用 GraphRAG
GRAPHRAG_MAX_QUERIES=3  # 每章节最大查询次数
```

### 适用场景

- 需要深入分析历史数据时
- 需要发现隐藏关联时
- 需要验证分析结论时

## 10. 重试机制 ⭐

### 重试配置

```python
LLM_RETRY_CONFIG = {
    "max_retries": 6,
    "initial_delay": 60,
    "max_delay": 10
}

SEARCH_API_RETRY_CONFIG = {
    "max_retries": 5,
    "initial_delay": 2,
    "max_delay": 25
}

DB_RETRY_CONFIG = {
    "max_retries": 5,
    "initial_delay": 1,
    "max_delay": 10
}
```

### 使用重试

```python
from utils.retry_helper import with_retry, LLM_RETRY_CONFIG

@with_retry(**LLM_RETRY_CONFIG)
def llm_call():
    return llm_client.generate(prompt)
```

## 概念关系图

```
┌────────────────────────────────────────────────────────────────┐
│                        BettaFish 概念关系图                     │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   ┌────────────────┐      ┌────────────────┐                  │
│   │   QueryAgent   │      │   MediaAgent   │                  │
│   │   - Tools      │      │   - Tools      │                  │
│   │   - Nodes      │      │   - Nodes      │                  │
│   │   - State      │      │   - State      │                  │
│   └────────┬───────┘      └────────┬───────┘                  │
│            │                      │                            │
│            └──────────┬───────────┘                            │
│                       ▼                                         │
│            ┌────────────────┐                                  │
│            │  ForumEngine   │ ← LLM Host                       │
│            │  - forum.log   │                                  │
│            └────────┬───────┘                                  │
│                     │                                          │
│                     ▼                                          │
│            ┌────────────────┐                                  │
│            │ ReportEngine   │                                  │
│            │  - Templates   │                                  │
│            │  - IR          │                                  │
│            │  - Nodes       │                                  │
│            └────────┬───────┘                                  │
│                     │                                          │
│                     ▼                                          │
│            ┌────────────────┐                                  │
│            │ Final Report   │                                  │
│            │  HTML/PDF/MD   │                                  │
│            └────────────────┘                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## 实践练习

### 练习 1：概念匹配 ⭐

将以下概念与对应的描述匹配：

| 概念 | 描述 |
|------|------|
| A. Agent | 1. 报告的中间表示格式 |
| B. Node | 2. 具有特定能力的智能体 |
| C. IR | 3. 处理特定步骤的单元 |
| D. Forum | 4. 多 Agent 协作机制 |

<details>
<summary>查看答案</summary>

**答案**：A-2, B-3, C-1, D-4
</details>

### 练习 2：节点职责 ⭐⭐

将以下任务与对应的节点匹配：

| 任务 | 节点 |
|------|------|
| 1. 分析段落需求，生成初始搜索查询 | |
| 2. 基于初步总结，进行更深入的搜索 | |
| 3. 整合搜索结果，生成初步总结 | |
| 4. 整合所有段落，生成最终报告 | |

选项：A. FirstSearchNode, B. FirstSummaryNode, C. ReflectionNode, D. ReportFormattingNode

<details>
<summary>查看答案</summary>

**答案**：1-A, 2-C, 3-B, 4-D
</details>

### 练习 3：IR 结构 ⭐⭐⭐

以下 IR 片段描述的是什么内容？

```json
{
  "type": "chart",
  "chart_type": "pie",
  "data": {...}
}
```

A. 一个饼图
B. 一个折线图
C. 一个柱状图
D. 一个散点图

<details>
<summary>查看答案</summary>

**答案**：A（chart_type 为 pie 表示饼图）
</details>

### 练习 4：代码理解 ⭐⭐⭐

阅读以下代码，说出它的功能：

```python
response = agent.execute_search_tool(
    tool_name="search_topic_globally",
    query="武汉大学",
    enable_sentiment=True
)
sentiment = response.parameters.get("sentiment_analysis")
```

<details>
<summary>查看答案</summary>

**答案**：执行全局话题搜索，并对结果进行情感分析，然后提取情感分析结果。
</details>

## 进阶思考

### 思考 1：论坛机制的价值

> 如果没有论坛机制，三个 Agent 各自工作会产生什么问题？论坛机制是如何解决这些问题的？

### 思考 2：IR 的设计权衡

> IR 增加了一层抽象，这带来了什么好处？有什么代价？在什么情况下 IR 的代价可能超过收益？

### 思考 3：Agent 专业化 vs 通用化

> BettaFish 采用专业化 Agent（每个 Agent 专精一个领域），而一些系统采用通用化 Agent（一个 Agent 能做所有事）。各自的优缺点是什么？什么场景下哪种更合适？

## 下一步

理解这些核心概念后，你可以：

- **[阅读引擎文档](../engines/)** - 深入了解各分析引擎的实现
- **[阅读论坛文档](../forum/forum-engine.md)** - 学习协作机制的细节
- **[阅读报告文档](../reporting/)** - 了解报告生成系统的完整流程

---

> 💡 **学习路径建议**：建议按 Agent → Node → State → Forum → IR 的顺序深入理解，这是从微观到宏观的认知路径。
