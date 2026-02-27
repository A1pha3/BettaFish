# ReportEngine 报告引擎

ReportEngine 是 BettaFish 的报告生成系统，负责将各分析引擎的输出整合成结构化的专业报告。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 ReportEngine 的多阶段生成流程
- [ ] 掌握报告模板的选择和使用方法
- [ ] 了解 IR（中间表示）的设计理念
- [ ] 熟练使用渲染器生成多种格式报告
- [ ] 能够创建自定义报告模板

## 概述

### 核心职责

ReportEngine 负责：

- **模板选择**：根据分析主题选择最合适的报告模板
- **布局设计**：设计报告标题、目录和主题
- **篇幅规划**：规划各章节的字数分配
- **章节生成**：使用 LLM 生成各章节内容
- **IR 装订**：将章节组装成 Document IR
- **格式渲染**：输出 HTML/PDF/Markdown 格式

### 技术特点

| 特点 | 说明 | 价值 |
|------|------|------|
| **多阶段生成** | 每个阶段由专门的 Node 处理 | 模块化，易于调试 |
| **模板驱动** | 基于 Markdown 模板自动解析 | 灵活定制 |
| **IR 中间表示** | 格式无关的文档表示 | 一次生成，多种输出 |
| **流式输出** | 支持 SSE 实时推送进度 | 更好的用户体验 |

### 为什么需要报告引擎？

> 💡 **设计理念**：三个分析引擎产生的是原始数据和分散的洞察。ReportEngine 的作用是将这些"原材料"加工成一份结构完整、逻辑清晰、可读性强的专业报告。

```
分析引擎输出 → 原始材料
├── QueryEngine:  新闻搜索结果 Markdown
├── MediaEngine:  多模态分析 Markdown
├── InsightEngine: 数据库挖掘 Markdown
└── ForumEngine:  协作讨论日志 forum.log

                    ↓ ReportEngine 加工

专业报告 → 结构化输出
├── HTML: 交互式网页（带图表）
├── PDF: 可打印文档
└── MD: 纯文本格式
```

## 架构设计

### 目录结构

```
ReportEngine/
├── __init__.py
├── agent.py                  # ReportAgent 主类
├── flask_interface.py        # Flask 接口
├── nodes/                    # 处理节点
│   ├── __init__.py
│   ├── base_node.py          # 基础节点
│   ├── template_selection_node.py   # 模板选择
│   ├── document_layout_node.py      # 布局设计
│   ├── word_budget_node.py          # 篇幅规划
│   ├── chapter_generation_node.py   # 章节生成
│   └── graphrag_query_node.py       # GraphRAG 查询
├── core/                     # 核心功能
│   ├── __init__.py
│   ├── template_parser.py    # 模板解析
│   ├── chapter_storage.py    # 章节存储
│   └── stitcher.py           # IR 装订器
├── ir/                       # IR 定义
│   ├── __init__.py
│   ├── schema.py             # IR Schema
│   └── validator.py          # IR 校验器
├── renderers/                # 渲染器
│   ├── __init__.py
│   ├── html_renderer.py      # HTML 渲染
│   ├── pdf_renderer.py       # PDF 渲染
│   └── chart_to_svg.py       # 图表转 SVG
├── graphrag/                 # GraphRAG
│   ├── __init__.py
│   ├── graph_builder.py      # 图谱构建
│   ├── query_engine.py       # 图检索
│   └── ...
├── report_template/          # 报告模板
│   ├── 企业品牌声誉分析报告.md
│   ├── 市场竞争格局舆情分析报告.md
│   └── ...
├── llms/                     # LLM 客户端
├── state/                    # 状态管理
├── prompts/                  # 提示词
└── utils/                    # 工具函数
```

## 核心组件

### 1. ReportAgent

报告生成主类，协调整个生成流程。

```python
class ReportAgent:
    """Report Agent 主类"""

    def __init__(self, config: Optional[Settings] = None):
        # 初始化 LLM 客户端
        self.llm_client = self._initialize_llm()

        # 初始化核心组件
        self.chapter_storage = ChapterStorage()
        self.document_composer = DocumentComposer()
        self.validator = IRValidator()
        self.renderer = HTMLRenderer()

        # 初始化节点
        self._initialize_nodes()

        # 文件基准管理器
        self.file_baseline = FileCountBaseline()
```

### 2. 处理节点

#### TemplateSelectionNode

从模板库中选择最合适的模板。

```python
class TemplateSelectionNode(BaseNode):
    """模板选择节点"""

    def run(self, state: ReportState) -> ReportState:
        """
        分析查询并选择最合适的报告模板

        Returns:
            更新后的状态，包含选定的模板
        """
```

#### DocumentLayoutNode

设计报告的整体布局。

```python
class DocumentLayoutNode(BaseNode):
    """文档布局节点"""

    def run(self, state: ReportState) -> ReportState:
        """
        设计报告标题、目录和主题

        Returns:
            更新后的状态，包含文档布局信息
        """
```

#### WordBudgetNode

规划各章节的字数分配。

```python
class WordBudgetNode(BaseNode):
    """篇幅规划节点"""

    def run(self, state: ReportState) -> ReportState:
        """
        根据总字数要求规划各章节篇幅

        Returns:
            更新后的状态，包含各章节字数分配
        """
```

#### ChapterGenerationNode

生成具体的章节内容。

```python
class ChapterGenerationNode(BaseNode):
    """章节生成节点"""

    def run(self, state: ReportState, section: TemplateSection) -> Dict:
        """
        生成单个章节的内容

        Args:
            state: 当前报告状态
            section: 章节模板信息

        Returns:
            章节内容（JSON 格式）
        """
```

### 3. 核心模块

#### TemplateParser

解析 Markdown 模板文件。

```python
def parse_template_sections(template_path: str) -> List[TemplateSection]:
    """
    解析 Markdown 模板

    - 以 ## 作为章节分隔符
    - 自动生成唯一 slug
    - 提取章节标题和内容要求
    """
```

#### ChapterStorage

管理章节的存储。

```python
class ChapterStorage:
    """章节存储管理器"""

    def save_chapter(self, run_id: str, slug: str, content: Dict):
        """保存章节到文件"""

    def load_chapter(self, run_id: str, slug: str) -> Dict:
        """加载章节内容"""

    def save_manifest(self, run_id: str, manifest: Dict):
        """保存运行清单"""
```

#### DocumentComposer

组装 Document IR。

```python
class DocumentComposer:
    """文档装订器"""

    def stitch_document(self, run_id: str, manifest: Dict) -> DocumentIR:
        """
        将各章节装订成完整的 Document IR

        - 添加缺失的元数据
        - 补充锚点信息
        - 验证结构完整性
        """
```

### 4. IR（中间表示）

#### IR Schema

```python
# 块类型定义
BLOCK_TYPES = {
    "heading": "标题",
    "paragraph": "段落",
    "chart": "图表",
    "image": "图片",
    "list": "列表",
    "quote": "引用",
    "table": "表格",
    "code": "代码块",
    "divider": "分隔线"
}

# IR 结构
class DocumentIR(TypedDict):
    title: str
    metadata: Dict[str, Any]
    sections: List[Dict[str, Any]]
```

#### IR 示例

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
      "level": 1,
      "id": "summary",
      "content": "执行摘要"
    },
    {
      "type": "paragraph",
      "content": "根据综合分析..."
    },
    {
      "type": "chart",
      "chart_type": "line",
      "data": {...}
    }
  ]
}
```

## 工作流程

### 完整生成流程

```
┌──────────────────────────────────────────────────────────────┐
│                  ReportEngine 工作流程                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  输入: 三个引擎的 Markdown 报告 + forum.log                  │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 1: 准备                                  │           │
│  │ - 加载输入文件                                │           │
│  │ - 初始化状态                                  │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 2: 模板选择                             │           │
│  │ - TemplateSelectionNode 分析主题            │           │
│  │ - 从模板库选择最合适的模板                   │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 3: 布局设计                             │           │
│  │ - DocumentLayoutNode 设计标题、目录         │           │
│  │ - 确定报告主题和风格                         │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 4: 篇幅规划                             │           │
│  │ - WordBudgetNode 规划各章节字数             │           │
│  │ - 生成章节生成指令                           │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 5: 章节生成（循环）                      │           │
│  │ ┌──────────────────────────────────────┐   │           │
│  │ │ 对每个章节:                            │   │           │
│  │ │ - GraphRAG 查询（可选）               │   │           │
│  │ │ - ChapterGenerationNode 生成内容      │   │           │
│  │ │ - IRValidator 校验结构                │   │           │
│  │ │ - 失败重试（最多3次）                  │   │           │
│  │ └──────────────────────────────────────┘   │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 6: IR 装订                              │           │
│  │ - DocumentComposer 组装 Document IR        │           │
│  │ - 补充元数据和锚点                           │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────────────────────────────────┐           │
│  │ 阶段 7: 渲染输出                             │           │
│  │ - HTMLRenderer 渲染 HTML                    │           │
│  │ - PDFRenderer 导出 PDF（可选）              │           │
│  │ - MarkdownRenderer 输出 MD（可选）          │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────┐                   ┌──────────────┐       │
│  │ 最终报告      │                   │ 保存清单      │       │
│  │ HTML/PDF/MD │                   │ manifest.json│       │
│  └──────────────┘                   └──────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## 配置说明

### 环境变量

```bash
# ReportEngine LLM 配置（推荐 Gemini）
REPORT_ENGINE_API_KEY=your_api_key
REPORT_ENGINE_BASE_URL=https://aihubmix.com/v1
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro

# GraphRAG 配置
GRAPHRAG_ENABLED=false
GRAPHRAG_MAX_QUERIES=3

# 输出目录配置
OUTPUT_DIR=final_reports
CHAPTER_OUTPUT_DIR=chapters
DOCUMENT_IR_OUTPUT_DIR=final_reports/ir
```

### 高级参数

```python
# ReportEngine/utils/config.py
class Settings(BaseSettings):
    # 章节生成重试配置
    _CONTENT_SPARSE_MIN_ATTEMPTS = 3
    _STRUCTURAL_RETRY_ATTEMPTS = 2

    # 输出目录
    OUTPUT_DIR: str = "final_reports"
    CHAPTER_OUTPUT_DIR: str = "chapters"
```

## 使用示例

### Web 界面使用

1. 完成三个引擎的分析后，访问 Web 界面
2. 点击"生成报告"按钮
3. 选择报告模板（可选）
4. 等待生成完成
5. 查看或下载报告

### 命令行使用

```bash
# 基本使用
python report_engine_only.py

# 指定主题
python report_engine_only.py --query "分析主题"

# 跳过 PDF 生成
python report_engine_only.py --skip-pdf

# 显示详细日志
python report_engine_only.py --verbose

# 启用 GraphRAG
python report_engine_only.py --graphrag-enabled true
```

### Python API 使用

```python
from ReportEngine import create_report_agent

# 创建 ReportAgent
agent = create_report_agent()

# 加载输入文件
agent.load_input_files(directories={
    'insight': 'insight_engine_streamlit_reports',
    'media': 'media_engine_streamlit_reports',
    'query': 'query_engine_streamlit_reports'
})

# 生成报告
report = agent.generate_report()

# 保存报告
agent.save_report(report, format='html')
```

## 报告模板

### 内置模板

| 模板文件 | 适用场景 |
|----------|----------|
| `企业品牌声誉分析报告.md` | 品牌形象、声誉监测 |
| `市场竞争格局舆情分析报告.md` | 竞品分析、市场研究 |
| `日常或定期舆情监测报告.md` | 定期舆情汇总 |
| `特定政策或行业动态舆情分析报告.md` | 政策影响评估 |
| `社会公共热点事件分析报告.md` | 热点事件追踪 |
| `突发事件与危机公关舆情报告.md` | 危机公关支持 |

### 自定义模板

1. 在 `ReportEngine/report_template/` 创建新的 `.md` 文件
2. 使用 `## ` 作为章节分隔
3. 为每个章节添加内容要求
4. 系统会自动识别并使用

### 模板格式示例

```markdown
# 报告标题模板

## 1. 总体概况

请从以下几个方面分析：
- 整体舆情态势
- 主要情感倾向
- 关键发现

## 2. 详细分析

### 2.1 平台分布

分析各平台的舆情特点...

### 2.2 时间趋势

分析舆情的时间演变...

## 3. 结论与建议

总结主要发现并提出建议...
```

## 渲染器

### HTMLRenderer

生成交互式 HTML 报告。

- 支持响应式布局
- 内置图表渲染（Plotly）
- 优雅的样式设计

### PDFRenderer

将 HTML 转换为 PDF。

- 使用 WeasyPrint 渲染
- 支持中文
- 保持原 HTML 样式

### MarkdownRenderer

输出 Markdown 格式。

- 纯文本格式
- 易于编辑和版本控制
- 支持大部分块类型

## GraphRAG 集成

GraphRAG 可以在章节生成前提供知识增强：

```python
# 启用 GraphRAG
if settings.GRAPHRAG_ENABLED:
    # 构建知识图谱
    graph = GraphBuilder.build(state_logs, forum_log)

    # 查询相关知识
    context = QueryEngine.query(graph, section.topic)

    # 将知识注入提示词
    prompt = enhance_with_graph(prompt, context)
```

## 实践练习

### 练习 1：基础报告生成 ⭐

**任务**：使用 Python API 生成一份基础报告

```python
from ReportEngine import create_report_agent

# 创建 ReportAgent
agent = create_report_agent()

# 加载输入文件（假设已有引擎输出）
agent.load_input_files(directories={
    'insight': 'insight_engine_streamlit_reports',
    'media': 'media_engine_streamlit_reports',
    'query': 'query_engine_streamlit_reports'
})

# 生成报告
try:
    report = agent.generate_report()
    print("报告生成成功!")
    print(f"标题: {report['title']}")
    print(f"章节数: {len(report['sections'])}")
except Exception as e:
    print(f"报告生成失败: {e}")
```

### 练习 2：自定义模板创建 ⭐⭐

**任务**：创建一个简单的自定义报告模板

```python
# 1. 在 ReportEngine/report_template/ 目录创建新文件
# 文件名: my_custom_report.md

template_content = """# 我的定制报告

## 1. 核心发现

请总结本次分析的 3-5 个核心发现，每个发现用一句话概括。

## 2. 详细分析

### 2.1 数据概览

请提供以下数据：
- 分析的数据量
- 覆盖的时间范围
- 涉及的平台数量

### 2.2 情感分析

请分析数据的情感分布，包括：
- 正面、中性、负面的比例
- 情感变化趋势
- 影响情感的主要因素

## 3. 结论与建议

请基于分析结果，提出：
- 3 条主要结论
- 2-3 条可操作的建议
"""

# 2. 保存模板
import os
template_path = "ReportEngine/report_template/my_custom_report.md"
os.makedirs(os.path.dirname(template_path), exist_ok=True)

with open(template_path, 'w', encoding='utf-8') as f:
    f.write(template_content)

print(f"模板已创建: {template_path}")
```

### 练习 3：报告质量优化 ⭐⭐⭐

**任务**：分析一份生成的报告，并提出优化建议

```python
from ReportEngine.ir.validator import IRValidator
from ReportEngine.core.chapter_storage import ChapterStorage

# 1. 加载最新生成的报告
storage = ChapterStorage()
latest_run = storage.get_latest_run_id()

if latest_run:
    # 2. 加载 IR
    with open(f"final_reports/ir/report_{latest_run}.json", 'r') as f:
        import json
        ir = json.load(f)

    # 3. 分析报告质量
    validator = IRValidator()
    result = validator.validate_document(ir)

    print("报告质量分析:")
    print(f"  验证状态: {'通过' if result.is_valid else '失败'}")
    print(f"  总块数: {ir['metadata']['total_blocks']}")
    print(f"  总字数: {ir['metadata'].get('word_count', 0)}")

    # 4. 统计块类型分布
    block_types = {}
    for section in ir['sections']:
        bt = section['type']
        block_types[bt] = block_types.get(bt, 0) + 1

    print("\n块类型分布:")
    for bt, count in sorted(block_types.items()):
        print(f"  {bt}: {count}")

    # 5. 优化建议
    if block_types.get('chart', 0) == 0:
        print("\n💡 建议: 报告缺少图表，可以添加数据可视化")
    if block_types.get('table', 0) == 0:
        print("\n💡 建议: 报告缺少表格，可以添加结构化数据")
    if ir['metadata'].get('word_count', 0) < 1000:
        print("\n💡 建议: 报告篇幅较短，可以增加分析深度")
```

## 常见问题

### Q1: 章节生成失败？

**症状**：
```
ChapterGenerationNode: 生成章节时出错
```

**可能原因**：
- LLM API 连接问题
- 提示词过长被截断
- 输出格式不正确

**解决方案**：

```bash
# 1. 检查 LLM 配置
python -c "
from config import settings
print(f'API Key: {settings.REPORT_ENGINE_API_KEY[:20]}...')
print(f'Model: {settings.REPORT_ENGINE_MODEL_NAME}')
"

# 2. 测试 LLM 连接
python -c "
from ReportEngine.llms.base import LLMClient
client = LLMClient(
    api_key=settings.REPORT_ENGINE_API_KEY,
    model_name=settings.REPORT_ENGINE_MODEL_NAME
)
response = client.generate('测试')
print(response)
"

# 3. 降低单次生成字数
# 在 prompts.py 中调整 max_tokens 参数
```

### Q2: PDF 生成失败？

**症状**：
```
PDFRenderer: WeasyPrint 报错
```

**解决方案**：

```bash
# 1. 安装系统依赖
# Ubuntu/Debian
sudo apt-get install python3-dev python3-pip libpango-1.0-0 libharfbuzz0b libpangoft2-0-0

# macOS
brew install pango

# 2. 安装中文字体
# 下载中文字体并配置到系统字体目录

# 3. 验证 PDF 生成
python -c "
from ReportEngine.renderers.pdf_renderer import PDFRenderer
renderer = PDFRenderer()
print('PDF 渲染器初始化成功')
"

# 4. 跳过 PDF 生成（调试用）
python report_engine_only.py --skip-pdf
```

### Q3: 报告质量不佳？

**优化策略**：

| 问题 | 解决方案 |
|------|----------|
| 内容空洞 | 启用 GraphRAG，提供更多上下文 |
| 结构混乱 | 优化模板内容要求 |
| 字数不符 | 调整 WordBudgetNode 的分配策略 |
| 格式错误 | 检查 IRValidator 是否正确校验 |

```python
# 启用 GraphRAG
# .env 文件
GRAPHRAG_ENABLED=true
GRAPHRAG_MAX_QUERIES=3

# 使用更强大的模型
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro
```

## 扩展开发

### 添加新的渲染器

```python
# ReportEngine/renderers/docx_renderer.py

from .base import BaseRenderer

class DocxRenderer(BaseRenderer):
    """Word 文档渲染器"""

    def render(self, document: DocumentIR) -> bytes:
        """渲染为 Word 文档

        Args:
            document: Document IR

        Returns:
            Word 文档字节流
        """
        from docx import Document

        doc = Document()

        # 添加标题
        doc.add_heading(document['title'], 0)

        # 处理各个章节
        for section in document['sections']:
            if section['type'] == 'heading':
                doc.add_heading(section['content'], section['level'])
            elif section['type'] == 'paragraph':
                doc.add_paragraph(section['content'])
            # ... 其他类型

        # 保存到字节流
        from io import BytesIO
        buffer = BytesIO()
        doc.save(buffer)
        return buffer.getvalue()
```

### 自定义章节生成策略

```python
# ReportEngine/nodes/custom_generation_node.py

from .base_node import BaseNode

class CustomChapterGenerationNode(BaseNode):
    """自定义章节生成节点"""

    def run(self, state: ReportState, section: TemplateSection) -> Dict:
        """自定义章节生成逻辑

        策略：
        1. 先尝试从历史数据中检索相似内容
        2. 如果有相似内容，作为参考
        3. 调用 LLM 生成新内容
        """
        # 1. 检索相似历史章节
        similar = self._retrieve_similar_chapter(section)

        # 2. 构建提示词
        if similar:
            prompt = f"""
            请参考以下历史章节，生成新的内容：

            历史参考:
            {similar['content']}

            新章节要求:
            {section.content}

            请生成类似风格的新内容。
            """
        else:
            prompt = f"请根据要求生成内容：{section.content}"

        # 3. 调用 LLM
        response = self.llm_client.generate(prompt)

        # 4. 解析为 IR
        return self._parse_to_ir(response)
```

## 相关文档

- **[IR 架构](ir-architecture.md)** - 中间表示详解
- **[模板系统](templates.md)** - 模板开发指南
- **[核心概念](../getting-started/concepts.md)** - 报告生成概念

---

> 💡 **学习提示**：ReportEngine 是 BettaFish 的输出层，它决定了最终交付的质量。建议按以下顺序学习：模板系统 → IR 架构 → 章节生成 → 渲染器。
