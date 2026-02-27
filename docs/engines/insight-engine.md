# InsightEngine 洞察引擎

InsightEngine 是 BettaFish 系统中的数据库挖掘 Agent，负责对私有数据库进行深度分析和情感分析。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 InsightEngine 的数据库挖掘能力
- [ ] 掌握 MediaCrawlerDB 工具集的 5 种查询方法
- [ ] 了解情感分析模型的工作原理
- [ ] 熟练使用关键词优化功能
- [ ] 能够独立配置和运行 InsightEngine

## 概述

### 核心职责

InsightEngine 专注于：

- **数据库查询**：对私有舆情数据库进行灵活查询
- **情感分析**：自动分析文本的情感倾向
- **数据挖掘**：发现隐藏的数据模式和趋势
- **关键词优化**：使用 AI 优化搜索关键词

### 技术基础

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| 数据库 | PostgreSQL/MySQL | 通过 SQLAlchemy 访问 |
| LLM | Kimi-K2 | 长上下文，适合深度分析 |
| 情感分析 | 多模型集成 | 22 语言支持 |
| 关键词优化 | Qwen AI | 智能关键词生成 |

### 为什么需要数据库挖掘？

> 💡 **设计理念**：公网搜索（QueryEngine）获取的是公开信息，而数据库挖掘（InsightEngine）获取的是私有、历史、结构化的数据。两者结合才能形成完整的认知。

**公网搜索 vs 数据库挖掘对比**：

```
┌─────────────────────────────────────────────────────────────┐
│                      公网搜索 (QueryEngine)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  数据来源: 新闻网站、公开网页、社交媒体公开内容              │
│                                                             │
│  优势:                                                      │
│  ✓ 实时性强 - 获取最新信息                                  │
│  ✓ 覆盖面广 - 多个信息源                                    │
│  ✓ 无需准备 - 直接搜索即可                                  │
│                                                             │
│  局限:                                                      │
│  ✗ 信息碎片化 - 缺乏历史关联                                │
│  ✗ 无法深度分析 - 受限于 API 返回量                         │
│  ✗ 情感分析困难 - 无法对大量数据进行情感分析                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            │ 结合
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    数据库挖掘 (InsightEngine)                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  数据来源: 私有舆情数据库（MindSpider 爬虫采集）            │
│                                                             │
│  优势:                                                      │
│  ✓ 历史数据 - 可以分析长期趋势                              │
│  ✓ 结构化查询 - 灵活的数据筛选                              │
│  ✓ 情感分析 - 批量处理大量数据                              │
│  ✓ 多平台整合 - 统一的查询接口                              │
│                                                             │
│  局限:                                                      │
│  ✗ 需要数据准备 - 依赖爬虫系统                              │
│  ✗ 实时性弱 - 数据有延迟                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 架构设计

### 目录结构

```
InsightEngine/
├── __init__.py
├── agent.py                  # Agent 主类
├── nodes/                    # 处理节点
│   ├── __init__.py
│   ├── base_node.py          # 基础节点
│   ├── search_node.py        # 搜索节点（集成关键词优化）
│   ├── summary_node.py       # 总结节点
│   ├── formatting_node.py    # 格式化节点
│   └── report_structure_node.py
├── tools/                    # 工具集
│   ├── __init__.py
│   ├── search.py             # MediaCrawlerDB 数据库工具
│   ├── keyword_optimizer.py  # 关键词优化中间件
│   └── sentiment_analyzer.py # 情感分析工具
├── llms/                     # LLM 客户端
│   ├── __init__.py
│   └── base.py
├── state/                    # 状态管理
│   ├── __init__.py
│   └── state.py
├── prompts/                  # 提示词模板
│   ├── __init__.py
│   └── prompts.py
└── utils/                    # 工具函数
    ├── __init__.py
    ├── config.py
    ├── db.py                 # 数据库连接封装
    └── text_processing.py
```

## 核心组件

### 1. DeepSearchAgent

InsightEngine 的主类，集成了数据库查询、情感分析和关键词优化等功能。

```python
class DeepSearchAgent:
    """Insight Engine 主类

    集成功能：
    - 数据库查询（MediaCrawlerDB）
    - 关键词优化（KeywordOptimizer）
    - 情感分析（MultilingualSentimentAnalyzer）
    - 结果聚类（KMeans）
    """

    def __init__(self, config: Optional[Settings] = None):
        # 初始化 LLM 客户端
        self.llm_client = self._initialize_llm()

        # 初始化搜索工具集
        self.search_agency = MediaCrawlerDB()

        # 初始化聚类小模型（懒加载）
        self._clustering_model = None

        # 初始化情感分析器
        self.sentiment_analyzer = multilingual_sentiment_analyzer

        # 初始化关键词优化器
        self.keyword_optimizer = KeywordOptimizer()

        # 初始化节点
        self._initialize_nodes()

        # 状态
        self.state = State()

    def _initialize_llm(self) -> LLMClient:
        """初始化 LLM 客户端"""
        return LLMClient(
            api_key=settings.INSIGHT_ENGINE_API_KEY,
            model_name=settings.INSIGHT_ENGINE_MODEL_NAME,
            base_url=settings.INSIGHT_ENGINE_BASE_URL
        )
```

### 2. MediaCrawlerDB

数据库查询工具集，提供 5 种专业查询工具。

```python
class MediaCrawlerDB:
    """私有舆情数据库查询工具集

    支持的平台表：
    - xhs_note（小红书）
    - dy_video（抖音）
    - ks_video（快手）
    - bili_video（B站）
    - wb_weibo（微博）
    - tieba_thread（贴吧）
    - zhihu_question（知乎）

    提供 5 种查询工具：
    1. search_hot_content - 查找热点内容
    2. search_topic_globally - 全局话题搜索
    3. search_topic_by_date - 按日期范围搜索
    4. get_comments_for_topic - 获取评论
    5. search_topic_on_platform - 平台专属搜索
    """

    def search_hot_content(self, time_period: str = "week",
                          limit: int = 100) -> Response:
        """查找热点内容

        基于智能热力评分：
        热力值 = 点赞×1.0 + 评论×5.0 + 转发×10.0 + 播放×0.1 + 弹幕×0.5

        Args:
            time_period: 时间周期（day/week/month）
            limit: 最大结果数

        Returns:
            Response with hot content results
        """

    def search_topic_globally(self, topic: str,
                              limit_per_table: int = 50) -> Response:
        """全局话题搜索（跨所有表）

        Args:
            topic: 搜索关键词
            limit_per_table: 每个平台表的最大结果数

        Returns:
            Response with results from all platforms
        """

    def search_topic_by_date(self, topic: str, start_date: str,
                            end_date: str) -> Response:
        """按日期范围搜索话题

        Args:
            topic: 搜索关键词
            start_date: 开始日期（YYYY-MM-DD）
            end_date: 结束日期（YYYY-MM-DD）

        Returns:
            Response with date-filtered results
        """

    def get_comments_for_topic(self, topic: str,
                              limit: int = 500) -> Response:
        """获取话题的评论

        Args:
            topic: 话题关键词（用于匹配内容标题）
            limit: 最大评论数

        Returns:
            Response with comments data
        """

    def search_topic_on_platform(self, platform: str, topic: str,
                                start_date: str = None,
                                end_date: str = None) -> Response:
        """在指定平台搜索话题

        Args:
            platform: 平台代码（xhs/dy/ks/bili/wb/tieba/zhihu）
            topic: 搜索关键词
            start_date: 可选的开始日期
            end_date: 可选的结束日期

        Returns:
            Response with platform-specific results
        """
```

### 3. 关键词优化中间件

使用 Qwen AI 优化 Agent 生成的搜索关键词。

```python
class KeywordOptimizer:
    """关键词优化中间件

    问题：Agent 生成的搜索关键词可能不够精确，
    导致数据库查询结果不理想。

    解决：使用 LLM 分析原始查询意图，
    生成更精确的搜索关键词组合。
    """

    def optimize_keywords(self, original_query: str,
                         context: str = "") -> Dict:
        """优化搜索关键词

        Args:
            original_query: 原始查询
            context: 查询上下文（段落标题、内容等）

        Returns:
            {
                "optimized_keywords": ["关键词1", "关键词2", ...],
                "reasoning": "优化理由说明",
                "search_strategy": "global/platform/date-specific"
            }

        示例:
            输入: original_query="武汉大学品牌"
            输出: {
                "optimized_keywords": [
                    "武汉大学",
                    "武大",
                    "WHU",
                    "武汉大学口碑",
                    "武汉大学评价"
                ],
                "reasoning": "包含全称、简称、英文名和评价相关词汇",
                "search_strategy": "global"
            }
        """
        prompt = f"""
        你是搜索关键词优化专家。请根据用户的查询意图，
        生成一组精确的搜索关键词。

        原始查询: {original_query}
        上下文: {context}

        请生成 5-8 个关键词，包括：
        1. 原始关键词
        2. 同义词/变体
        3. 相关术语
        4. 英文缩写（如有）
        5. 评价相关词汇（如适用）

        返回 JSON 格式。
        """

        response = self.llm_client.generate(prompt)
        return self._parse_response(response)
```

### 4. 情感分析器

集成多种情感分析模型。

```python
class MultilingualSentimentAnalyzer:
    """多语言情感分析器

    支持的模型：
    1. WeiboMultilingualSentiment - 22 语言支持（默认）
    2. BertChinese-Lora - 中文 BERT 微调模型
    3. GPT2-Lora - 生成式分析
    4. 传统机器学习 - SVM/XGBoost
    """

    def analyze_single_text(self, text: str,
                          model: str = "multilingual") -> Dict:
        """分析单条文本

        Args:
            text: 待分析文本
            model: 模型选择

        Returns:
            {
                "sentiment": "positive/neutral/negative",
                "confidence": 0.95,
                "scores": {
                    "positive": 0.95,
                    "neutral": 0.03,
                    "negative": 0.02
                }
            }
        """

    def analyze_batch(self, texts: List[str],
                     model: str = "multilingual") -> Dict:
        """批量分析

        Args:
            texts: 文本列表
            model: 模型选择

        Returns:
            {
                "success": true,
                "total_analyzed": 100,
                "sentiment_distribution": {
                    "positive": 65,
                    "neutral": 25,
                    "negative": 10
                },
                "average_confidence": 0.87,
                "results": [...]
            }
        """

    def analyze_query_results(self, query_results: List[Dict],
                            text_field: str = "content") -> Dict:
        """分析查询结果

        Args:
            query_results: 数据库查询结果列表
            text_field: 文本字段名

        Returns:
            带情感标注的查询结果
        """
```

## 数据库查询工具详解

### 1. search_hot_content - 热点内容搜索

查找热点内容，基于智能热力评分。

**热力评分公式**：
```
热力值 = 点赞数 × 1.0 + 评论数 × 5.0 + 转发数 × 10.0 + 播放数 × 0.1 + 弹幕数 × 0.5
```

**权重设计原理**：

| 互动类型 | 权重 | 理由 |
|---------|------|------|
| 点赞 | 1.0 | 门槛低，表示轻度认可 |
| 评论 | 5.0 | 门槛中等，表示中度关注 |
| 转发 | 10.0 | 门槛高，表示强烈认同 |
| 播放 | 0.1 | 被动行为，权重最低 |
| 弹幕 | 0.5 | 介于播放和评论之间 |

**使用示例**：

```python
from InsightEngine import create_agent

agent = create_agent()

# 查找本周热点
response = agent.execute_search_tool(
    tool_name="search_hot_content",
    query="",
    time_period="week",  # day/week/month
    limit=100,
    enable_sentiment=True  # 启用情感分析
)

# 查看结果
print(f"找到 {response.results_count} 条热点内容")
for item in response.results[:5]:
    print(f"{item.title}")
    print(f"  热力值: {item.hot_score:.2f}")
    if hasattr(item, 'sentiment'):
        print(f"  情感: {item.sentiment}")
```

### 2. search_topic_globally - 全局话题搜索

全局话题搜索，跨所有平台表进行查询。

**支持的平台表**：

| 平台代码 | 平台名 | 表名 | 特点 |
|---------|--------|------|------|
| `xhs` | 小红书 | `xhs_note` | 社区电商，图片丰富 |
| `dy` | 抖音 | `dy_video` | 短视频，年轻用户 |
| `ks` | 快手 | `ks_video` | 短视频，下沉市场 |
| `bili` | B站 | `bili_video` | 中长视频，Z世代 |
| `wb` | 微博 | `wb_weibo` | 社交网络，热点传播快 |
| `tieba` | 贴吧 | `tieba_thread` | 社区论坛，话题聚焦 |
| `zhihu` | 知乎 | `zhihu_question` | 问答社区，深度讨论 |

**使用示例**：

```python
response = agent.execute_search_tool(
    tool_name="search_topic_globally",
    query="武汉大学",
    limit_per_table=20  # 每个平台表最多 20 条
)

# 按平台统计
platform_counts = {}
for item in response.results:
    platform = item.platform
    platform_counts[platform] = platform_counts.get(platform, 0) + 1

print("各平台结果分布:")
for platform, count in platform_counts.items():
    print(f"  {platform}: {count} 条")
```

### 3. search_topic_by_date - 日期范围搜索

按日期范围搜索话题。

**使用场景**：
- 分析特定事件的发展过程
- 对比不同时期的舆论变化
- 追踪历史数据趋势

**使用示例**：

```python
response = agent.execute_search_tool(
    tool_name="search_topic_by_date",
    query="校招",
    start_date="2024-01-01",
    end_date="2024-01-31",
    enable_sentiment=True
)

# 分析时间趋势
from collections import defaultdict
daily_sentiment = defaultdict(lambda: {"positive": 0, "negative": 0, "neutral": 0})

for item in response.results:
    if hasattr(item, 'created_at') and hasattr(item, 'sentiment'):
        date = item.created_at[:10]  # YYYY-MM-DD
        sentiment = item.sentiment
        daily_sentiment[date][sentiment] += 1

# 输出情感趋势
print("日期\t正面\t负面\t中性")
for date in sorted(daily_sentiment.keys()):
    s = daily_sentiment[date]
    print(f"{date}\t{s['positive']}\t{s['negative']}\t{s['neutral']}")
```

### 4. get_comments_for_topic - 评论获取

获取话题的评论内容。

**使用场景**：
- 深入了解用户观点
- 分析评论的情感倾向
- 发现用户关注的具体问题

**使用示例**：

```python
response = agent.execute_search_tool(
    tool_name="get_comments_for_topic",
    query="某条内容的标题或URL关键词",
    limit=500
)

# 分析评论情感
if "sentiment_analysis" in response.parameters:
    sentiment = response.parameters["sentiment_analysis"]
    total = sentiment["total_analyzed"]

    print("评论情感分布:")
    print(f"  正面: {sentiment['sentiment_distribution']['positive']} ({sentiment['sentiment_distribution']['positive']/total*100:.1f}%)")
    print(f"  中性: {sentiment['sentiment_distribution']['neutral']} ({sentiment['sentiment_distribution']['neutral']/total*100:.1f}%)")
    print(f"  负面: {sentiment['sentiment_distribution']['negative']} ({sentiment['sentiment_distribution']['negative']/total*100:.1f}%)")

    # 查看负面评论示例
    print("\n负面评论示例:")
    for result in sentiment["results"]:
        if result["sentiment"] == "negative":
            print(f"  - {result['text'][:50]}...")
```

### 5. search_topic_on_platform - 平台专属搜索

在指定平台搜索话题。

**使用场景**：
- 针对特定平台进行深入分析
- 对比不同平台的讨论差异
- 了解平台特色内容

**使用示例**：

```python
# 对比两个平台的讨论
platforms = ["xhs", "wb"]
topic = "考研"

platform_analysis = {}
for platform in platforms:
    response = agent.execute_search_tool(
        tool_name="search_topic_on_platform",
        query=topic,
        platform=platform
    )

    # 简单分析
    platform_analysis[platform] = {
        "count": response.results_count,
        "avg_like": sum(r.likes for r in response.results) / response.results_count if response.results_count > 0 else 0
    }

# 输出对比
print(f"{topic} 在不同平台的讨论情况:")
for platform, analysis in platform_analysis.items():
    print(f"  {platform}: {analysis['count']} 条, 平均点赞 {analysis['avg_like']:.1f}")
```

## 情感分析

### 支持的模型

| 模型 | 特点 | 适用场景 | 准确率 |
|------|------|----------|--------|
| WeiboMultilingualSentiment | 22 语言支持 | 多语言内容 | ~85% |
| BertChinese-Lora | BERT 微调 | 中文精准分析 | ~90% |
| GPT2-Lora | GPT-2 微调 | 生成式分析 | ~82% |
| 传统机器学习 | SVM/XGBoost | 快速分析 | ~78% |

### 情感分析结果格式

```json
{
  "success": true,
  "total_analyzed": 100,
  "sentiment_distribution": {
    "positive": 65,
    "neutral": 25,
    "negative": 10
  },
  "average_confidence": 0.87,
  "results": [
    {
      "text": "这个产品真的很好用，强烈推荐！",
      "sentiment": "positive",
      "confidence": 0.95
    },
    {
      "text": "还可以吧，没有特别惊艳。",
      "sentiment": "neutral",
      "confidence": 0.72
    },
    {
      "text": "太失望了，完全不推荐购买。",
      "sentiment": "negative",
      "confidence": 0.91
    }
  ]
}
```

### 情感分析工作流程

```
┌─────────────────────────────────────────────────────────────┐
│                    情感分析工作流程                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  输入: 数据库查询结果（100+ 条评论）                         │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │ Step 1: 文本预处理                            │          │
│  │ - 去除 HTML 标签                              │          │
│  │ - 去除表情符号                                │          │
│  │ - 文本长度过滤（<10字符跳过）                 │          │
│  └──────────────────────────────────────────────┘          │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │ Step 2: 模型推理                              │          │
│  │ - 批量处理（提升效率）                        │          │
│  │ - 多模型集成（可选）                          │          │
│  └──────────────────────────────────────────────┘          │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────────────────────────────────────┐          │
│  │ Step 3: 结果聚合                              │          │
│  │ - 统计情感分布                                │          │
│  │ - 计算平均置信度                              │          │
│  │ - 生成可视化图表                              │          │
│  └──────────────────────────────────────────────┘          │
│       │                                                     │
│       ▼                                                     │
│  输出: 情感分析报告                                         │
│        - 正面: 65%                                          │
│        - 中性: 25%                                          │
│        - 负面: 10%                                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 数据聚类

### KMeans 聚类原理

InsightEngine 使用 KMeans 对搜索结果进行聚类，确保结果的多样性。

**为什么需要聚类？**

```
问题：搜索"iPhone"可能返回100条结果，但大部分内容相似。

解决：通过聚类将相似内容分组，从每组选取代表性结果。

效果：用10条聚类结果代表100条原始结果，信息覆盖更全面。
```

### 聚类工作流程

```python
def _cluster_and_sample_results(self, results: List, max_results: int = 50):
    """
    对搜索结果进行聚类并采样

    流程：
    1. 使用 SentenceTransformer 编码文本
       - 将文本转换为向量表示
       - 相似文本在向量空间中距离接近

    2. KMeans 聚类
       - 将结果划分为 k 个聚类
       - k = min(结果数/5, max_results)

    3. 从每个聚类中采样
       - 选择距离聚类中心最近的结果
       - 确保每个聚类都有代表

    Args:
        results: 搜索结果列表
        max_results: 最大返回数

    Returns:
        采样后的结果列表（具有多样性）
    """
    if not results:
        return results

    # 懒加载聚类模型
    if self._clustering_model is None:
        from sentence_transformers import SentenceTransformer
        self._clustering_model = SentenceTransformer(
            'paraphrase-multilingual-MiniLM-L12-v2'
        )

    # 文本编码
    texts = [r.content for r in results if r.content]
    embeddings = self._clustering_model.encode(texts)

    # KMeans 聚类
    n_clusters = min(len(texts) // RESULTS_PER_CLUSTER, max_results)
    if n_clusters < 2:
        return results[:max_results]

    kmeans = KMeans(n_clusters=n_clusters, random_state=42)
    labels = kmeans.fit_predict(embeddings)

    # 从每个聚类采样
    sampled_results = []
    for i in range(n_clusters):
        cluster_indices = [j for j, label in enumerate(labels) if label == i]
        # 选择距离聚类中心最近的结果
        center = kmeans.cluster_centers_[i]
        distances = [np.linalg.norm(embeddings[j] - center) for j in cluster_indices]
        best_idx = cluster_indices[np.argmin(distances)]
        sampled_results.append(results[best_idx])

    return sampled_results
```

### 聚类参数配置

```python
# InsightEngine/utils/config.py
ENABLE_CLUSTERING = True          # 是否启用聚类
MAX_CLUSTERED_RESULTS = 50        # 聚类后最大返回数
RESULTS_PER_CLUSTER = 5           # 每个聚类返回结果数
```

## 配置说明

### 环境变量配置

```bash
# InsightEngine LLM 配置（推荐 Kimi，长上下文）
INSIGHT_ENGINE_API_KEY=your_api_key
INSIGHT_ENGINE_BASE_URL=https://api.moonshot.cn/v1
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview

# 关键词优化器配置（推荐 Qwen）
KEYWORD_OPTIMIZER_API_KEY=your_api_key
KEYWORD_OPTIMIZER_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
KEYWORD_OPTIMIZER_MODEL_NAME=qwen-plus

# 数据库配置
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=your_password
DB_NAME=bettafish
DB_DIALECT=postgresql

# 搜索限制配置
DEFAULT_SEARCH_HOT_CONTENT_LIMIT=100
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=50
DEFAULT_SEARCH_TOPIC_BY_DATE_LIMIT_PER_TABLE=100
DEFAULT_GET_COMMENTS_FOR_TOPIC_LIMIT=500
DEFAULT_SEARCH_TOPIC_ON_PLATFORM_LIMIT=200
MAX_SEARCH_RESULTS_FOR_LLM=0
MAX_REFLECTIONS=3
```

## 使用示例

### 基本使用

```python
from InsightEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索任务
report = agent.research(
    query="分析武汉大学在社交媒体上的形象",
    save_report=True
)

print(report)
```

### 执行特定工具

```python
# 查找热点内容
response = agent.execute_search_tool(
    tool_name="search_hot_content",
    query="",
    time_period="week",
    limit=100,
    enable_sentiment=True  # 启用情感分析
)

# 查看情感分析结果
if "sentiment_analysis" in response.parameters:
    sentiment = response.parameters["sentiment_analysis"]
    print(f"正面: {sentiment['positive']}")
    print(f"中性: {sentiment['neutral']}")
    print(f"负面: {sentiment['negative']}")
```

### 独立运行 Streamlit 应用

```bash
streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501
```

### 代码示例：高级查询

```python
from InsightEngine import create_agent

agent = create_agent()

# 场景：分析特定话题在不同平台的情感差异
topic = "新能源汽车"
platforms = ["xhs", "wb", "zhihu"]

platform_sentiments = {}
for platform in platforms:
    response = agent.execute_search_tool(
        tool_name="search_topic_on_platform",
        query=topic,
        platform=platform,
        enable_sentiment=True
    )

    if "sentiment_analysis" in response.parameters:
        sa = response.parameters["sentiment_analysis"]
        platform_sentiments[platform] = {
            "positive": sa["sentiment_distribution"]["positive"],
            "neutral": sa["sentiment_distribution"]["neutral"],
            "negative": sa["sentiment_distribution"]["negative"],
            "total": sa["total_analyzed"]
        }

# 输出对比
print(f"{topic} - 平台情感对比:")
print(f"{'平台':<10} {'总数':>8} {'正面':>8} {'中性':>8} {'负面':>8}")
print("-" * 42)
for platform, data in platform_sentiments.items():
    total = data["total"]
    print(f"{platform:<10} {total:>8} {data['positive']:>8} {data['neutral']:>8} {data['negative']:>8}")
```

## 工作流程

```
┌──────────────────────────────────────────────────────────────┐
│                   InsightEngine 工作流程                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  用户输入 ──→ "分析某话题的舆情"                              │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step 1: 关键词优化                          │           │
│  │ - KeywordOptimizer 分析查询意图              │           │
│  │ - 生成优化关键词列表                         │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────┐            ┌──────────────────┐       │
│  │ Step 2: 数据库   │            │ 多关键词查询      │       │
│  │ 查询             │            │ 并行执行          │       │
│  │ - MediaCrawlerDB │            │ 结果合并去重      │       │
│  └────┬─────────────┘            └────────┬─────────┘       │
│       │                                  │                  │
│       ▼                                  ▼                  │
│  ┌──────────────────┐            ┌──────────────────┐       │
│  │ Step 3: 结果聚类 │            │ 情感分析          │       │
│  │ - KMeans 聚类   │            │ - 多模型并行      │       │
│  │ - 代表性采样    │            │ - 统计分布        │       │
│  └────┬─────────────┘            └────────┬─────────┘       │
│       │                                  │                  │
│       └────────────┬─────────────────────┘                  │
│                    │                                       │
│                    ▼                                       │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step 4: 生成总结                            │           │
│  │ - 整合搜索结果和情感分析                     │           │
│  │ - 生成结构化总结                             │           │
│  │ - 保存到 insight.log                        │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │ 反思循环      │                   │ 最终报告      │       │
│  │ (最多3轮)     │                   │ insight.log  │       │
│  │ - 根据总结    │                   └──────────────┘       │
│  │   优化搜索    │                                          │
│  │   深化分析    │                                          │
│  └──────────────┘                                          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## 平台代码参考

| 代码 | 平台 | 特点 | 用户画像 |
|------|------|------|----------|
| `xhs` | 小红书 | 社区电商、图片丰富 | 年轻女性、消费能力强 |
| `dy` | 抖音 | 短视频、娱乐性强 | 年轻用户、下沉市场 |
| `ks` | 快手 | 短视频、真实接地气 | 下沉市场、中年用户 |
| `bili` | B站 | 中长视频、ACG文化 | Z世代、二次元用户 |
| `wb` | 微博 | 社交网络、热点传播 | 各年龄段、意见领袖 |
| `tieba` | 贴吧 | 社区论坛、话题聚焦 | 兴趣群体、男性偏多 |
| `zhihu` | 知乎 | 问答社区、深度讨论 | 高学历、专业人士 |

## 常见问题

### Q1: 数据库连接失败？

**症状**：
```
psycopg2.OperationalError: could not connect to server
```

**排查步骤**：

```bash
# 1. 检查数据库服务
sudo systemctl status postgresql  # Linux
brew services list | grep postgresql  # macOS

# 2. 测试连接
psql -h localhost -U bettafish -d bettafish

# 3. 验证配置
python -c "
from config import settings
print(f'Host: {settings.DB_HOST}')
print(f'Port: {settings.DB_PORT}')
print(f'Database: {settings.DB_NAME}')
"

# 4. 检查防火墙
sudo ufw allow 5432  # Linux
```

### Q2: 搜索结果为空？

**可能原因**：

1. **数据库没有数据**：运行 MindSpider 爬虫采集数据
2. **关键词不匹配**：尝试使用更通用的关键词
3. **日期范围过窄**：扩大日期范围

**解决方案**：

```python
# 1. 检查数据库是否有数据
from InsightEngine.utils.db import execute_query

# 检查各表数据量
tables = ["xhs_note", "dy_video", "wb_weibo"]
for table in tables:
    result = execute_query(f"SELECT COUNT(*) as cnt FROM {table}")
    print(f"{table}: {result[0]['cnt']} 条")

# 2. 使用更通用的关键词
# 原来: "武汉大学2024年计算机学院研究生录取分数线"
# 改为: "武汉大学 录取分数线"

# 3. 扩大日期范围
response = agent.execute_search_tool(
    tool_name="search_topic_by_date",
    query="关键词",
    start_date="2023-01-01",  # 扩大起始日期
    end_date="2024-12-31"
)
```

### Q3: 情感分析结果不准？

**优化方案**：

```python
# 1. 尝试不同的情感模型
# 配置文件中修改
SENTIMENT_MODEL = "BertChinese-Lora"  # 更精准的中文模型

# 2. 过滤低置信度结果
def filter_low_confidence(results, threshold=0.7):
    """过滤低置信度结果"""
    return [r for r in results if r.confidence >= threshold]

# 3. 文本预处理
# 清理URL、表情符号等干扰信息

# 4. 使用多模型集成
# 综合多个模型的结果，取平均值或投票
```

## 实践练习

### 练习 1：基础查询 ⭐

**任务**：查询本周热点内容并分析情感

```python
from InsightEngine import create_agent

agent = create_agent()

# 查询热点
response = agent.execute_search_tool(
    tool_name="search_hot_content",
    query="",
    time_period="week",
    limit=20,
    enable_sentiment=True
)

# 输出结果
print(f"本周热点 Top 5:")
for i, item in enumerate(response.results[:5], 1):
    print(f"{i}. {item.title}")
    print(f"   平台: {item.platform}")

# 输出情感分布
if "sentiment_analysis" in response.parameters:
    sa = response.parameters["sentiment_analysis"]
    print(f"\n情感分布:")
    print(f"  正面: {sa['sentiment_distribution']['positive']}")
    print(f"  中性: {sa['sentiment_distribution']['neutral']}")
    print(f"  负面: {sa['sentiment_distribution']['negative']}")
```

### 练习 2：跨平台对比 ⭐⭐

**任务**：对比同一话题在不同平台的情感差异

```python
from InsightEngine import create_agent

agent = create_agent()
topic = "人工智能"

platforms = ["xhs", "wb", "zhihu"]
results = {}

for platform in platforms:
    response = agent.execute_search_tool(
        tool_name="search_topic_on_platform",
        query=topic,
        platform=platform,
        enable_sentiment=True
    )

    if "sentiment_analysis" in response.parameters:
        sa = response.parameters["sentiment_analysis"]
        total = sa["total_analyzed"]
        results[platform] = {
            "positive_pct": sa["sentiment_distribution"]["positive"] / total * 100 if total > 0 else 0,
            "neutral_pct": sa["sentiment_distribution"]["neutral"] / total * 100 if total > 0 else 0,
            "negative_pct": sa["sentiment_distribution"]["negative"] / total * 100 if total > 0 else 0,
        }

# 输出对比表
print(f"\n【{topic}】各平台情感对比:")
print(f"{'平台':<10} {'正面':>10} {'中性':>10} {'负面':>10}")
print("-" * 40)
for platform, data in results.items():
    print(f"{platform:<10} {data['positive_pct']:>9.1f}% {data['neutral_pct']:>9.1f}% {data['negative_pct']:>9.1f}%")
```

### 练习 3：时间趋势分析 ⭐⭐⭐

**任务**：分析话题在一段时间内的情感变化趋势

```python
from InsightEngine import create_agent
from datetime import datetime, timedelta

agent = create_agent()
topic = "新能源汽车"

# 按周分析
results_by_week = {}
for i in range(4):  # 最近4周
    end_date = datetime.now() - timedelta(weeks=i)
    start_date = end_date - timedelta(weeks=1)

    response = agent.execute_search_tool(
        tool_name="search_topic_by_date",
        query=topic,
        start_date=start_date.strftime("%Y-%m-%d"),
        end_date=end_date.strftime("%Y-%m-%d"),
        enable_sentiment=True
    )

    if "sentiment_analysis" in response.parameters:
        sa = response.parameters["sentiment_analysis"]
        total = sa["total_analyzed"]
        week_label = f"第{4-i}周"
        results_by_week[week_label] = {
            "positive": sa["sentiment_distribution"]["positive"] / total * 100 if total > 0 else 0,
            "count": total
        }

# 输出趋势
print(f"\n【{topic}】最近4周情感趋势:")
print(f"{'周次':<10} {'讨论量':>10} {'正面比例':>10}")
print("-" * 30)
for week, data in results_by_week.items():
    print(f"{week:<10} {data['count']:>10} {data['positive']:>9.1f}%")

# 分析趋势
positive_values = [data["positive"] for data in results_by_week.values()]
if positive_values[0] > positive_values[-1]:
    print("\n结论: 舆情呈下降趋势")
elif positive_values[0] < positive_values[-1]:
    print("\n结论: 舆情呈上升趋势")
else:
    print("\n结论: 舆情相对稳定")
```

## 扩展开发

### 添加自定义查询工具

```python
# 在 tools/search.py 中添加新工具

def custom_query_by_engagement(self, topic: str,
                              min_likes: int = 1000) -> Response:
    """查询高互动内容

    Args:
        topic: 搜索关键词
        min_likes: 最小点赞数

    Returns:
        Response with high-engagement content
    """
    query = f"""
    SELECT * FROM xhs_note
    WHERE title LIKE %s
      AND liked_count >= %s
    ORDER BY liked_count DESC
    LIMIT 50
    """

    results = execute_query(query, params=[f"%{topic}%", min_likes])

    return Response(
        tool_name="custom_query_by_engagement",
        parameters={
            "topic": topic,
            "min_likes": min_likes
        },
        results=results,
        results_count=len(results)
    )
```

### 自定义情感分析

```python
from transformers import pipeline

class CustomSentimentAnalyzer:
    """自定义情感分析器"""

    def __init__(self, model_name: str = "cardiffnlp/twitter-roberta-base-sentiment"):
        self.classifier = pipeline(
            "sentiment-analysis",
            model=model_name,
            return_all_scores=True
        )

    def analyze(self, text: str) -> Dict:
        """分析文本情感

        Returns:
            {
                "sentiment": "positive/neutral/negative",
                "confidence": float,
                "scores": {...}
            }
        """
        results = self.classifier(text)[0]
        # 转换为统一格式
        sentiment_map = {
            "LABEL_0": "negative",
            "LABEL_1": "neutral",
            "LABEL_2": "positive"
        }

        best_result = max(results, key=lambda x: x["score"])

        return {
            "sentiment": sentiment_map[best_result["label"]],
            "confidence": best_result["score"],
            "scores": {
                sentiment_map[r["label"]]: r["score"]
                for r in results
            }
        }
```

## 相关文档

- **[核心概念](../getting-started/concepts.md)** - 节点和 Agent 概念
- **[QueryEngine](query-engine.md)** - 查询引擎文档
- **[MediaEngine](media-engine.md)** - 多模态引擎文档
- **[ForumEngine](../forum/forum-engine.md)** - 论坛协作机制
- **[MindSpider](../crawler/mindspider.md)** - 数据采集系统

---

> 💡 **学习提示**：InsightEngine 是 BettaFish 中最复杂的引擎，建议按以下顺序学习：数据库查询 → 情感分析 → 关键词优化 → 完整工作流程。
