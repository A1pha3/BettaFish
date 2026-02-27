# MediaEngine 多模态引擎

MediaEngine 是 BettaFish 系统中的多模态分析 Agent，负责理解和分析视频、图片等多模态内容。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 MediaEngine 的多模态处理能力
- [ ] 掌握 BochaMultimodalSearch 工具的使用方法
- [ ] 了解多模态内容分析的工作流程
- [ ] 能够独立运行和调试 MediaEngine
- [ ] 学会将 MediaEngine 与其他引擎协作

## 概述

### 核心职责

MediaEngine 专注于：

- **多模态搜索**：支持图片、视频内容的搜索
- **内容理解**：解析视频、图片中的信息
- **跨模态分析**：结合文本和视觉信息进行综合分析
- **视觉验证**：通过图像分析验证文本信息的真实性

### 技术基础

| 组件 | 技术选型 | 说明 |
|------|---------|------|
| 搜索 API | Bocha API | 多模态搜索引擎 |
| LLM | Gemini-2.5-Pro | 强大的多模态理解能力 |
| 视觉能力 | 原生支持 | 图片和视频理解 |

### 为什么需要多模态分析？

> 💡 **设计理念**：现实世界的信息是多模态的。仅分析文本会丢失大量有价值的信息——产品图片、宣传视频、用户截图等。MediaEngine 的作用就是弥补这一盲区，提供完整的视觉分析能力。

**传统文本分析的局限性**：

```
┌─────────────────────────────────────────────────────────────┐
│                     传统文本分析                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   输入："这款手机外观很漂亮"                                 │
│                                                             │
│   分析结果：用户对外观评价正面                               │
│                                                             │
│   ❌ 缺失信息：                                             │
│   - 实际外观是什么？                                        │
│   - 用户展示的是哪些部位？                                  │
│   - 图片是否有修图痕迹？                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**多模态分析的优势**：

```
┌─────────────────────────────────────────────────────────────┐
│                     多模态分析                               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   输入：文本 + 产品图片 + 开箱视频                           │
│                                                             │
│   分析结果：                                                │
│   ✓ 文本分析：用户对外观评价正面                            │
│   ✓ 图片分析：手机采用直角边框，蓝色渐变背面，无显著修图     │
│   ✓ 视频分析：开箱视频中展示了实际质感，相机凸起明显        │
│                                                             │
│   ✅ 综合结论：手机外观设计现代化，但相机模块较为突出        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 架构设计

### 目录结构

```
MediaEngine/
├── __init__.py
├── agent.py                  # Agent 主类
├── nodes/                    # 处理节点
│   ├── __init__.py
│   ├── base_node.py          # 基础节点
│   ├── search_node.py        # 搜索节点
│   ├── summary_node.py       # 总结节点
│   ├── formatting_node.py    # 格式化节点
│   └── report_structure_node.py
├── tools/                    # 工具集
│   ├── __init__.py
│   └── search.py             # Bocha 多模态搜索封装
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
    └── text_processing.py
```

## 核心组件

### 1. DeepSearchAgent

MediaEngine 的主类，与 QueryEngine 和 InsightEngine 采用相同的架构模式。

```python
class DeepSearchAgent:
    """Media Engine 主类"""

    def __init__(self, config: Optional[Settings] = None):
        # 初始化 LLM 客户端
        self.llm_client = self._initialize_llm()

        # 初始化搜索工具集
        self.search_agency = BochaMultimodalSearch()

        # 初始化节点
        self._initialize_nodes()

        # 状态
        self.state = State()

    def _initialize_llm(self) -> LLMClient:
        """初始化 LLM 客户端"""
        return LLMClient(
            api_key=settings.MEDIA_ENGINE_API_KEY,
            model_name=settings.MEDIA_ENGINE_MODEL_NAME,
            base_url=settings.MEDIA_ENGINE_BASE_URL
        )
```

### 2. BochaMultimodalSearch

多模态搜索工具的封装类，提供统一的搜索接口。

```python
class BochaMultimodalSearch:
    """Bocha 多模态搜索 API 封装

    提供四种搜索模式：
    - web_search: 网页文本搜索
    - search_image: 图片内容搜索
    - search_video: 视频内容搜索
    - comprehensive_search: 综合搜索（文本+图片+视频）
    """

    def search_web(self, query: str, max_results: int = 10) -> Response:
        """网页搜索

        Args:
            query: 搜索关键词
            max_results: 最大结果数

        Returns:
            Response: {
                "tool_name": "search_web",
                "parameters": {"query": "..."},
                "results": [...],
                "results_count": 10
            }
        """

    def search_image(self, query: str, max_results: int = 10) -> Response:
        """图片搜索

        搜索场景：
        - 产品图片：搜索品牌产品相关图片
        - 场景图片：了解产品使用场景
        - 对比图片：获取竞品对比图
        """

    def search_video(self, query: str, max_results: int = 10) -> Response:
        """视频搜索

        搜索场景：
        - 开箱视频：了解产品实际外观
        - 评测视频：获取专业评价
        - 事件视频：追踪事件现场
        """

    def comprehensive_search(self, query: str, max_results: int = 10) -> Response:
        """综合搜索（文本+图片+视频）

        返回三种类型的搜索结果，适用于需要全面了解的场景。
        """
```

## 多模态能力

### 支持的内容类型

| 类型 | 说明 | 应用场景 | 分析价值 |
|------|------|----------|----------|
| **文本** | 新闻、文章、帖子 | 基础信息获取 | ⭐⭐ |
| **图片** | 截图、海报、图表 | 视觉内容分析 | ⭐⭐⭐ |
| **视频** | 短视频、长视频、直播片段 | 动态内容理解 | ⭐⭐⭐⭐ |
| **结构化数据** | 天气、日历、股票卡片 | 信息提取 | ⭐⭐⭐ |

### 多模态分析示例

```
输入：分析抖音上的产品评价

搜索策略：
├── 文本搜索："产品名称 评测 评价"
├── 图片搜索："产品名称 实拍 开箱"
└── 视频搜索："产品名称 评测 测评"

搜索结果：
├── 文本内容（150+ 条）
│   ├── 用户评论
│   ├── 产品描述
│   └── 媒体报道
│
├── 图片内容（80+ 张）
│   ├── 产品照片
│   ├── 使用场景图
│   └── 对比图表
│
└── 视频内容（30+ 个）
    ├── 开箱视频
    ├── 使用演示
    └── 专业评测

多模态分析：
├── 视觉分析
│   ├── 产品外观：设计风格、颜色搭配
│   ├── 包装质量：包装材质、开箱体验
│   └── 使用场景：实际使用环境
│
├── 内容分析
│   ├── 用户使用体验
│   ├── 常见问题汇总
│   └── 优缺点对比
│
├── 情感分析
│   ├── 文本情感：评论态度倾向
│   └── 视觉情感：表情、肢体语言
│
└── 综合结论
    ├── 产品优势列表
    ├── 产品劣势列表
    └── 购买建议
```

## 工作流程

MediaEngine 的工作流程与 QueryEngine 类似，但增加了多模态处理能力：

```
┌──────────────────────────────────────────────────────────────┐
│                   MediaEngine 工作流程                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  用户输入 ──→ "分析某产品的社交媒体反响"                      │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step 1: 生成报告结构                         │           │
│  │ - ReportStructureNode 分析任务               │           │
│  │ - 分解为研究段落（产品外观、功能、评价...）│     │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────┐            ┌──────────────────┐       │
│  │ Step 2: 多模态搜索（并行）        │       │
│  │ ┌──────────────┴──────────────┐ │       │
│  │ │ 文本搜索 + 图片搜索 + 视频搜索│ │       │
│  │ └──────────────┬──────────────┘ │       │
│  └─────────────────┼─────────────────┘       │
│                    │                         │
│                    ▼                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Step 3: 多模态内容理解                     │   │
│  │                                                       │   │
│  │  文本处理:                                            │   │
│  │  └── 提取关键信息、情感分析、主题聚类                │   │
│  │                                                       │   │
│  │  视觉处理:                                            │   │
│  │  ├── 图片理解：OCR、场景识别、物体检测               │   │
│  │  └── 视频理解：关键帧提取、动作识别、场景分析        │   │
│  │                                                       │   │
│  │  跨模态融合:                                          │   │
│  │  └── 文本-视觉对齐、信息验证、综合分析               │   │
│  └─────────────────────┬─────────────────────────────────┘   │
│                        │                                      │
│                        ▼                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ Step 4: 生成总结                            │           │
│  │ - 综合文本和视觉信息                         │           │
│  │ - 生成结构化总结                             │           │
│  │ - 保存到 media.log                          │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │ 反思循环      │                   │ 最终报告      │       │
│  │ (最多3轮)     │                   │ media.log    │       │
│  │ - 根据总结    │                   └──────────────┘       │
│  │   进行深度    │                                          │
│  │   多模态搜索  │                                          │
│  └──────────────┘                                          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## 配置说明

### 环境变量配置

```bash
# MediaEngine LLM 配置（推荐 Gemini，多模态能力强）
MEDIA_ENGINE_API_KEY=your_api_key
MEDIA_ENGINE_BASE_URL=https://aihubmix.com/v1
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro

# Bocha API 配置
SEARCH_TOOL_TYPE=BochaAPI
BOCHA_BASE_URL=https://api.bocha.cn/v1/ai-search
BOCHA_WEB_SEARCH_API_KEY=your_bocha_api_key
```

### 高级参数配置

```python
# MediaEngine/utils/config.py
class Config:
    # 搜索限制
    comprehensive_search_limit = 10  # 综合搜索每种类型限制
    web_search_limit = 15            # 网页搜索限制
    image_search_limit = 20          # 图片搜索限制
    video_search_limit = 10          # 视频搜索限制

    # 反思控制
    max_reflections = 2              # 最大反思轮次（默认 2）

    # 多模态处理
    enable_video_keyframe = True     # 启用视频关键帧提取
    keyframe_interval = 5            # 关键帧间隔（秒）
    max_video_duration = 300         # 最大视频处理时长（秒）
```

## 使用示例

### 基本使用

```python
from MediaEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行搜索任务
report = agent.research(
    query="分析某品牌在抖音上的营销效果",
    save_report=True
)

print(report)
```

### 独立运行 Streamlit 应用

```bash
streamlit run SingleEngineApp/media_engine_streamlit_app.py --server.port 8502
```

### 代码示例：直接使用工具

```python
from MediaEngine.tools import BochaMultimodalSearch

# 创建工具实例
tool = BochaMultimodalSearch()

# 搜索图片
results = tool.search_image(
    query="iPhone 15 Pro 实拍",
    max_results=10
)

# 查看图片结果
for result in results:
    print(f"标题: {result.title}")
    print(f"图片URL: {result.image_url}")
    print(f"来源: {result.source}")
    print("---")
```

### 代码示例：综合搜索

```python
# 综合搜索（文本+图片+视频）
results = tool.comprehensive_search(
    query="小米汽车 SU7 评测",
    max_results=10
)

# 分类查看结果
print(f"网页结果: {len(results['web'])} 条")
print(f"图片结果: {len(results['image'])} 条")
print(f"视频结果: {len(results['video'])} 条")

# 分析图片结果
for img in results['image']:
    print(f"图片: {img['title']}")
    print(f"链接: {img['url']}")
```

## 多模态分析场景

### 场景 1：产品分析 ⭐⭐

**任务**：分析产品的社交媒体反响

MediaEngine 能力：

| 能力 | 具体功能 | 价值 |
|------|----------|------|
| 图片搜索 | 搜索产品相关图片 | 了解产品外观 |
| 视频搜索 | 搜索开箱、评测视频 | 获取实际使用反馈 |
| 视觉分析 | 分析产品展示效果 | 发现营销亮点 |
| 内容整合 | 综合评价产品优缺点 | 形成完整认知 |

**分析流程**：

```
1. 多模态搜索
   ├── 产品图片（官方图、用户实拍图）
   ├── 开箱视频（UP主评测、用户分享）
   └── 文本评论（社交媒体、论坛）

2. 视觉内容分析
   ├── 产品外观：设计风格、颜色、材质
   ├── 包装质量：包装设计、开箱体验
   └── 使用场景：实际使用环境、搭配方式

3. 综合评价
   ├── 优势列表：设计、功能、价格
   ├── 劣势列表：不足、改进建议
   └── 购买建议：适合人群、使用场景
```

### 场景 2：事件追踪 ⭐⭐⭐

**任务**：追踪突发事件的现场情况

MediaEngine 能力：

```
事件追踪搜索策略：
├── 现场图片：获取第一手视觉资料
├── 现场视频：了解事件动态过程
├── 地图信息：确认事件发生地点
└── 相关报道：获取权威信息

时间线构建：
├── 事件开始：现场图片/视频时间戳
├── 事件发展：多个视觉资料的时间排序
├── 关键时刻：重要视觉节点识别
└── 事件结束：后续影响图片
```

### 场景 3：品牌监控 ⭐⭐

**任务**：监控品牌在社交媒体上的视觉形象

```
品牌视觉分析维度：
├── Logo 使用：是否规范、是否侵权
├── 产品展示：展示方式、展示场景
├── 用户反馈：用户图片中的产品使用状态
└── 竞品对比：与竞品的视觉对比分析
```

## 与其他引擎协作

### 协作模式图

```
┌─────────────────────────────────────────────────────────────┐
│                      三引擎协作示例                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  查询任务："分析某品牌新产品的市场反响"                       │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ QueryEngine  │  │ MediaEngine  │  │ InsightEngine│      │
│  │              │  │              │  │              │      │
│  │ 新闻搜索:    │  │ 图片搜索:    │  │ 数据库查询:  │      │
│  │ - 官方发布   │  │ - 产品图片   │  │ - 用户评论   │      │
│  │ - 媒体报道   │  │ - 实拍图     │  │ - 社交讨论   │      │
│  │ - 行业分析   │  │ - 评测视频   │  │ - 情感分析   │      │
│  │              │  │              │  │              │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                 │               │
│         └─────────────────┼─────────────────┘               │
│                           │                                 │
│                           ▼                                 │
│                  ┌───────────────┐                         │
│                  │  ForumEngine  │                         │
│                  │               │                         │
│                  │  整合发现:    │                         │
│                  │  - 新闻正面   │                         │
│                  │  - 视觉精美   │                         │
│                  │  - 用户评价   │                         │
│                  │    两极分化   │                         │
│                  └───────────────┘                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 协作收益

| 协作组合 | 协作效果 | 应用场景 |
|----------|----------|----------|
| Query + Media | 新闻背景 + 视觉验证 | 品牌活动分析 |
| Query + Insight | 新闻动态 + 用户数据 | 舆情趋势分析 |
| Media + Insight | 视觉内容 + 情感数据 | 产品评价分析 |
| 三引擎协作 | 全方位立体分析 | 综合品牌声誉报告 |

## 性能优化

### 搜索优化

```python
# 1. 按需搜索：根据任务选择搜索类型
def smart_search(query: str, task_type: str):
    """智能搜索选择"""
    if task_type == "visual_analysis":
        # 视觉分析任务：优先图片和视频
        return tool.search_image(query) + tool.search_video(query)
    elif task_type == "news_tracking":
        # 新闻追踪：优先文本搜索
        return tool.search_web(query)
    else:
        # 综合任务：全部搜索
        return tool.comprehensive_search(query)

# 2. 结果过滤：过滤低质量内容
def filter_results(results: List, min_quality: float = 0.7):
    """过滤低质量结果"""
    return [r for r in results if r.quality_score >= min_quality]

# 3. 缓存机制：缓存常用搜索结果
from functools import lru_cache

@lru_cache(maxsize=100)
def cached_search(query: str):
    return tool.search_web(query)
```

### 分析优化

```python
# 1. 分批处理：大量内容分批次分析
def batch_analyze(items: List, batch_size: int = 10):
    """分批分析大量内容"""
    results = []
    for i in range(0, len(items), batch_size):
        batch = items[i:i+batch_size]
        results.extend(analyze_batch(batch))
    return results

# 2. 关键帧提取：视频只分析关键帧
def extract_keyframes(video_url: str, interval: int = 5):
    """提取视频关键帧

    Args:
        video_url: 视频 URL
        interval: 提取间隔（秒）
    """
    frames = []
    # 使用 FFmpeg 或视频处理库提取关键帧
    # 只分析关键帧，减少处理量
    return frames

# 3. 异步处理：耗时操作异步执行
import asyncio

async def async_search(queries: List[str]):
    """异步并行搜索"""
    tasks = [tool.search_web(q) for q in queries]
    results = await asyncio.gather(*tasks)
    return results
```

## 常见问题

### Q1: 视频搜索返回空结果？

**可能原因**：
- Bocha API 配置错误
- 搜索关键词过于具体
- API 视频搜索服务暂时不可用

**解决方案**：

```python
# 1. 验证 API 配置
import os
assert os.getenv("BOCHA_WEB_SEARCH_API_KEY"), "API Key 未配置"

# 2. 测试连接
from MediaEngine.tools import BochaMultimodalSearch
tool = BochaMultimodalSearch()
results = tool.search_video("测试", max_results=1)
print(results)

# 3. 使用更通用的关键词
# 原来: "iPhone 15 Pro Max 深蓝色 256G 开箱视频详细评测"
# 改为: "iPhone 15 Pro 开箱"
```

### Q2: 图片内容识别不准确？

**解决方案**：

1. **使用更清晰的图片**
   ```python
   # 添加图片质量过滤
   high_quality_images = [
       img for img in results
       if img.resolution >= (1920, 1080)
   ]
   ```

2. **提供更多上下文信息**
   ```python
   # 增强查询描述
   enhanced_query = f"{query} 产品照片 官方图 高清"
   ```

3. **调整 LLM 提示词**
   ```python
   # 在 prompts.py 中优化提示词
   IMAGE_ANALYSIS_PROMPT = """
   请仔细分析这张图片，关注：
   1. 主体物品的外观特征
   2. 背景环境和使用场景
   3. 任何文字或标识信息
   4. 整体风格和色调

   请提供详细描述。
   """
   ```

### Q3: 多模态分析速度慢？

**优化方案**：

```bash
# 1. 减少搜索结果数量
# .env 文件
IMAGE_SEARCH_LIMIT=10
VIDEO_SEARCH_LIMIT=5

# 2. 降低视频分析精度
ENABLE_VIDEO_KEYFRAME=true
KEYFRAME_INTERVAL=10  # 增加间隔

# 3. 使用更快的模型（可能牺牲准确度）
MEDIA_ENGINE_MODEL_NAME=gemini-2.0-flash
```

## 实践练习

### 练习 1：基础使用 ⭐

**任务**：创建一个 MediaEngine 实例并执行图片搜索

```python
from MediaEngine import create_agent

# 创建 Agent
agent = create_agent()

# 执行图片搜索
response = agent.execute_search_tool(
    tool_name="search_image",
    query="武汉大学 校园风景",
    max_results=10
)

# 打印前 3 条结果
for i, result in enumerate(response.results[:3]):
    print(f"{i+1}. {result.title}")
    print(f"   URL: {result.url}")
```

### 练习 2：多模态综合分析 ⭐⭐

**任务**：对同一主题进行文本、图片、视频搜索并对比结果

```python
from MediaEngine.tools import BochaMultimodalSearch

tool = BochaMultimodalSearch()
query = "小米汽车 SU7"

# 综合搜索
results = {
    "web": tool.search_web(query, max_results=5),
    "image": tool.search_image(query, max_results=5),
    "video": tool.search_video(query, max_results=5)
}

# 对比结果数量
for type_name, result in results.items():
    print(f"{type_name}: {len(result.results)} 条结果")

# 分析：哪种类型的信息最丰富？
most_rich = max(results.items(), key=lambda x: len(x[1].results))
print(f"\n信息最丰富的类型: {most_rich[0]}")
```

### 练习 3：自定义多模态分析 ⭐⭐⭐

**任务**：创建一个自定义函数，整合多种搜索结果并生成综合报告

```python
from MediaEngine.tools import BochaMultimodalSearch
from typing import Dict, List

def comprehensive_brand_analysis(brand: str) -> Dict:
    """综合品牌多模态分析

    Args:
        brand: 品牌名称

    Returns:
        {
            "brand": str,
            "web_summary": str,
            "image_count": int,
            "video_count": int,
            "insights": List[str]
        }
    """
    tool = BochaMultimodalSearch()

    # 执行多模态搜索
    web_results = tool.search_web(f"{brand} 品牌 评价", max_results=10)
    image_results = tool.search_image(f"{brand} 产品", max_results=10)
    video_results = tool.search_video(f"{brand} 评测", max_results=5)

    # 分析结果
    insights = []

    # 从文本结果中提取洞察
    if web_results.results:
        top_result = web_results.results[0]
        insights.append(f"新闻热点: {top_result.title}")

    # 从图片结果中统计
    image_themes = {}
    for img in image_results.results[:5]:
        # 简单的主题分类（实际可以用 LLM）
        if "产品" in img.title:
            image_themes["产品"] = image_themes.get("产品", 0) + 1
        if "实拍" in img.title:
            image_themes["实拍"] = image_themes.get("实拍", 0) + 1

    insights.append(f"图片主题分布: {image_themes}")

    # 返回综合报告
    return {
        "brand": brand,
        "web_summary": web_results.results[0].title if web_results.results else "无结果",
        "image_count": len(image_results.results),
        "video_count": len(video_results.results),
        "insights": insights
    }

# 测试
report = comprehensive_brand_analysis("比亚迪")
print(f"品牌: {report['brand']}")
print(f"新闻摘要: {report['web_summary']}")
print(f"图片数量: {report['image_count']}")
print(f"视频数量: {report['video_count']}")
print("洞察:")
for insight in report['insights']:
    print(f"  - {insight}")
```

## 扩展开发

### 添加新的多模态搜索源

```python
# 在 tools/search.py 中添加新的搜索源

class CustomMultimodalSearch:
    """自定义多模态搜索"""

    def search_moments(self, query: str, max_results: int = 10):
        """搜索朋友圈/动态内容

        Args:
            query: 搜索关键词
            max_results: 最大结果数

        Returns:
            List[SearchResult]
        """
        # 实现搜索逻辑
        pass

    def search_livestream(self, query: str, max_results: int = 10):
        """搜索直播内容

        Args:
            query: 搜索关键词
            max_results: 最大结果数

        Returns:
            List[SearchResult]
        """
        # 实现搜索逻辑
        pass
```

### 自定义多模态分析节点

```python
from nodes.base_node import BaseNode

class MultiModalAnalysisNode(BaseNode):
    """多模态分析节点"""

    def run(self, input_data: Dict) -> Dict:
        """
        多模态分析节点处理逻辑

        输入示例：
        {
            "title": "产品分析",
            "content": "分析产品的多模态内容",
            "search_results": {
                "web": [...],
                "image": [...],
                "video": [...]
            }
        }

        输出示例：
        {
            "visual_insights": [...],
            "content_insights": [...],
            "cross_modal_findings": [...]
        }
        """
        # 实现多模态分析逻辑
        pass
```

## 相关文档

- **[核心概念](../getting-started/concepts.md)** - Agent 和节点通用概念
- **[QueryEngine](query-engine.md)** - 查询引擎文档
- **[InsightEngine](insight-engine.md)** - 洞察引擎文档
- **[ForumEngine](../forum/forum-engine.md)** - 论坛协作机制

---

> 💡 **学习提示**：MediaEngine 的核心价值在于多模态信息的整合。建议先掌握基础搜索功能，再学习如何将不同类型的信息进行综合分析。
