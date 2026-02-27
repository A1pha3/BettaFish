# IR 架构（文档中间表示）

IR（Intermediate Representation）是 ReportEngine 的核心设计，它是一种格式无关的文档表示形式，支持渲染为多种输出格式。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 IR 的设计理念和核心价值
- [ ] 掌握各种块类型的使用方法
- [ ] 了解 IR 的校验和验证机制
- [ ] 学会扩展 IR 添加新的块类型
- [ ] 能够使用 IR 进行格式转换

## 概述

### 为什么需要 IR？

> 💡 **核心问题**：传统报告生成直接绑定输出格式，难以支持多种输出。

**传统方式的问题**：

```
问题1：格式绑定
├── 生成 HTML → HTML 生成逻辑
├── 生成 PDF → PDF 生成逻辑
└── 生成 MD → MD 生成逻辑
   结果：每种格式需要独立实现，代码重复

问题2：无法复用
├── HTML → PDF 需要重新解析
├── 内容修改 → 需要重新生成所有格式
└── 无法增量更新

问题3：质量不一致
├── 不同渲染器的实现细节不同
└── 输出质量差异大
```

**IR 解决方案**：

```
内容生成 → IR（格式无关）→ 多种输出
                │
                ├──→ HTMLRenderer → HTML
                ├──→ PDFRenderer → PDF
                └──── MarkdownRenderer → MD

优势：
✓ 内容生成与输出格式解耦
✓ 同一 IR 可渲染为多种格式
✓ 支持增量更新章节
✓ 输出质量一致
```

### 设计目标

| 目标 | 说明 | 实现方式 |
|------|------|----------|
| **格式无关** | 不绑定任何特定输出格式 | 使用通用数据结构 |
| **可扩展** | 易于添加新的块类型 | 定义统一接口 |
| **可验证** | 结构可自动校验 | IRValidator |
| **可序列化** | 支持 JSON 存储和加载 | 标准化 Schema |

### IR 与其他系统的对比

```
Markdown:
├── 优点: 简单、易读、易写
├── 缺点: 结构化程度低、难以程序化处理
└── 适用: 人为编写文档

HTML:
├── 优点: 结构化、支持样式
├── 缺点: 复杂、与显示耦合
└── 适用: Web 展示

IR (BettaFish):
├── 优点: 格式无关、可验证、可扩展
├── 缺点: 需要学习新的数据结构
└── 适用: 报告生成、格式转换
```

## IR Schema

### 块类型定义

```python
# ReportEngine/ir/schema.py

BLOCK_TYPES = {
    # 文本块
    "heading": "标题",
    "paragraph": "段落",
    "list": "列表",
    "quote": "引用",
    "code": "代码块",

    # 视觉块
    "chart": "图表",
    "image": "图片",
    "table": "表格",
    "divider": "分隔线"
}
```

### 块结构规范

每种块类型都有特定的结构要求：

```python
# heading 块
{
    "type": "heading",
    "level": 1,              # 1-6
    "id": "section-id",      # 锚点 ID
    "content": "标题文字"
}

# paragraph 块
{
    "type": "paragraph",
    "content": "段落内容..."
}

# chart 块
{
    "type": "chart",
    "chart_type": "line",    # line/bar/pie/scatter
    "data": {...},           # Plotly JSON
    "title": "图表标题",
    "caption": "图表说明"
}

# image 块
{
    "type": "image",
    "url": "图片路径",
    "alt": "替代文本",
    "caption": "图片说明"
}

# list 块
{
    "type": "list",
    "list_type": "bullet",   # bullet/number
    "items": ["项1", "项2"]
}

# table 块
{
    "type": "table",
    "headers": ["列1", "列2"],
    "rows": [
        ["值1", "值2"],
        ["值3", "值4"]
    ]
}
```

### 文档结构

```python
class DocumentIR(TypedDict):
    """文档 IR 结构"""
    title: str                           # 文档标题
    metadata: Dict[str, Any]             # 元数据
    sections: List[Dict[str, Any]]       # 块列表

class DocumentMetadata(TypedDict):
    """文档元数据"""
    generated_at: str                    # 生成时间
    query: str                           # 分析主题
    total_blocks: int                    # 总块数
    word_count: int                      # 总字数
    template: str                        # 使用的模板
```

## IR 生成流程

```
┌──────────────────────────────────────────────────────────────┐
│                      IR 生成流程                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  模板文件                                              │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────┐                                       │
│  │ TemplateParser   │                                       │
│  │ - 解析章节        │                                       │
│  │ - 生成 slug       │                                       │
│  └────┬─────────────┘                                       │
│       │                                                      │
│       ▼                                                      │
│  ┌──────────────────┐      ┌──────────────────┐             │
│  │WordBudgetNode    │ ──→  │章节生成指令       │             │
│  │- 规划字数分配    │      │- 字数要求        │             │
│  └────┬─────────────┘      │- 内容要求        │             │
│       │                   └──────────────────┘             │
│       ▼                                                      │
│  ┌──────────────────────────────────────────────┐           │
│  │ ChapterGenerationNode                        │           │
│  │ - 调用 LLM 生成内容                           │           │
│  │ - 输出 JSON 格式的章节                       │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────┐               ┌──────────────────┐     │
│  │ IRValidator      │               │ ChapterStorage   │     │
│  │ - 校验结构        │               │ - 保存章节 JSON  │     │
│  │ - 检查必需字段    │               │ - 记录清单       │     │
│  └──────────────────┘               └──────────────────┘     │
│       │                                     │                │
│       └────────────┬────────────────────────┘                │
│                    │                                         │
│                    ▼                                         │
│  ┌──────────────────────────────────────────────┐           │
│  │ DocumentComposer (Stitcher)                  │           │
│  │ - 加载所有章节                               │           │
│  │ - 补充元数据                                 │           │
│  │ - 生成锚点 ID                                │           │
│  │ - 组装完整 Document IR                       │           │
│  └────┬─────────────────────────────────────┬───┘           │
│       │                                     │                │
│       ▼                                     ▼                │
│  ┌──────────────────┐               ┌──────────────────┐     │
│  │ 保存 IR          │               │ 渲染器           │     │
│  │ final_reports/ir │               │ HTML/PDF/MD      │     │
│  └──────────────────┘               └──────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## IR 校验

### IRValidator

IRValidator 负责校验 IR 的结构完整性。

```python
class IRValidator:
    """IR 校验器"""

    def validate_chapter(self, chapter: Dict) -> ValidationResult:
        """
        校验单章内容

        检查项：
        - 必需字段存在
        - 块类型有效
        - 块结构正确
        - 内容不为空
        """

    def validate_document(self, document: DocumentIR) -> ValidationResult:
        """
        校验完整文档

        检查项：
        - 文档元数据完整
        - 章节顺序正确
        - 锚点 ID 唯一
        - 引用有效
        """
```

### 校验规则

| 规则 | 说明 | 示例 |
|------|------|------|
| **类型有效性** | 块类型必须在预定义列表中 | `"type" in BLOCK_TYPES` |
| **字段完整性** | 必需字段必须存在 | heading 必须有 content |
| **类型约束** | 字段值类型必须正确 | level 必须是整数 |
| **范围约束** | 数值必须在有效范围 | level ∈ [1, 6] |
| **引用有效性** | 引用资源必须存在 | image url 有效 |

## IR 渲染

### 渲染器接口

```python
class BaseRenderer:
    """渲染器基类"""

    def render(self, document: DocumentIR) -> str:
        """渲染文档为特定格式"""
        raise NotImplementedError

class HTMLRenderer(BaseRenderer):
    """HTML 渲染器"""

    def render(self, document: DocumentIR) -> str:
        """渲染为 HTML"""

class PDFRenderer(BaseRenderer):
    """PDF 渲染器"""

    def render(self, document: DocumentIR) -> bytes:
        """渲染为 PDF"""

class MarkdownRenderer(BaseRenderer):
    """Markdown 渲染器"""

    def render(self, document: DocumentIR) -> str:
        """渲染为 Markdown"""
```

### 渲染流程

```
Document IR
     │
     ├──→ HTMLRenderer → HTML 文件
     ├──→ PDFRenderer → PDF 文件
     └──→ MarkdownRenderer → MD 文件
```

## IR 存储结构

### 目录结构

```
chapters/
└── runs/
    └── 20240120_120000/              # 运行时间戳
        ├── chapter_summary.json     # 各章节 JSON
        ├── chapter_analysis.json
        ├── chapter_conclusion.json
        └── manifest.json            # 运行清单

final_reports/
└── ir/
    └── report_20240120_120000.json  # 完整 Document IR
```

### Manifest 格式

```json
{
  "run_id": "20240120_120000",
  "generated_at": "2024-01-20T12:00:00",
  "query": "分析主题",
  "template": "企业品牌声誉分析报告",
  "chapters": [
    {
      "slug": "summary",
      "title": "执行摘要",
      "file": "chapter_summary.json",
      "status": "completed",
      "word_count": 500
    }
  ],
  "status": "completed"
}
```

## 块类型详解

### 1. Heading（标题）

用于标识文档结构层次。

```json
{
  "type": "heading",
  "level": 2,
  "id": "section-2",
  "content": "2. 详细分析"
}
```

**渲染规则**：
- HTML: `<h2 id="section-2">2. 详细分析</h2>`
- MD: `## 2. 详细分析`
- PDF: 带编号的标题

### 2. Paragraph（段落）

普通文本内容。

```json
{
  "type": "paragraph",
  "content": "根据综合分析，该品牌在..."
}
```

**支持内联格式**：
- 粗体：`**文本**`
- 斜体：`*文本*`
- 链接：`[文本](url)`

### 3. Chart（图表）

使用 Plotly 渲染的交互式图表。

```json
{
  "type": "chart",
  "chart_type": "line",
  "data": {
    "data": [{...}],
    "layout": {...}
  },
  "title": "情感趋势图",
  "caption": "图1：近30天情感变化趋势"
}
```

**支持的图表类型**：
- `line`: 折线图
- `bar`: 柱状图
- `pie`: 饼图
- `scatter`: 散点图

### 4. Image（图片）

图片引用和说明。

```json
{
  "type": "image",
  "url": "images/chart.png",
  "alt": "趋势图",
  "caption": "图1：数据趋势"
}
```

### 5. List（列表）

有序或无序列表。

```json
{
  "type": "list",
  "list_type": "bullet",
  "items": [
    "第一点",
    "第二点",
    "第三点"
  ]
}
```

### 6. Table（表格）

结构化数据表格。

```json
{
  "type": "table",
  "headers": ["平台", "正面", "中性", "负面"],
  "rows": [
    ["微博", "45%", "30%", "25%"],
    ["小红书", "60%", "25%", "15%"]
  ]
}
```

## 扩展 IR

### 添加新块类型

1. 在 `schema.py` 中定义新类型
2. 更新 `BLOCK_TYPES`
3. 在各渲染器中添加渲染逻辑

```python
# 1. 定义新类型
BLOCK_TYPES["callout"] = "提示框"

# 2. 定义结构
{
    "type": "callout",
    "style": "info",  # info/warning/success
    "content": "重要提示内容"
}

# 3. HTMLRenderer 添加渲染逻辑
def render_callout(self, block: Dict) -> str:
    style = block.get("style", "info")
    content = block.get("content", "")
    return f'<div class="callout callout-{style}">{content}</div>'
```

## 实践练习

### 练习 1：理解 IR 结构 ⭐

**任务**：阅读以下 IR 片段，识别块类型

```json
{
  "title": "分析报告",
  "sections": [
    {"type": "heading", "level": 1, "content": "概述"},
    {"type": "paragraph", "content": "本报告分析了..."},
    {"type": "chart", "chart_type": "line", "data": {...}}
  ]
}
```

**问题**：
1. 有多少个块？（3个）
2. 包含哪些块类型？（heading, paragraph, chart）
3. 如果渲染为 HTML，heading 会变成什么？（`<h1>概述</h1>`）

### 练习 2：创建简单 IR ⭐⭐

**任务**：使用 Python 创建一个简单的 Document IR

```python
# 创建一个简单的 IR
document_ir = {
    "title": "测试报告",
    "metadata": {
        "generated_at": "2024-01-20T10:00:00",
        "query": "测试查询",
        "total_blocks": 3,
        "word_count": 200
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
            "content": "这是一份测试报告，用于演示 IR 结构。"
        },
        {
            "type": "list",
            "list_type": "bullet",
            "items": [
                "发现点一",
                "发现点二",
                "发现点三"
            ]
        }
    ]
}

# 验证 IR
from ReportEngine.ir.validator import IRValidator
validator = IRValidator()
result = validator.validate_document(document_ir)

print(f"验证结果: {'通过' if result.is_valid else '失败'}")
if not result.is_valid:
    print(f"错误: {result.errors}")
```

### 练习 3：扩展 IR 添加新块类型 ⭐⭐⭐

**任务**：添加一个新的 "callout" 块类型

```python
# 1. 定义 callout 块的创建函数
def create_callout(content: str, style: str = "info") -> dict:
    """创建提示框块

    Args:
        content: 提示内容
        style: 样式类型 (info/warning/success/error)

    Returns:
        callout 块的 IR 表示
    """
    return {
        "type": "callout",
        "style": style,
        "content": content
    }

# 2. 添加到文档
callout_block = create_callout(
    "注意：本报告基于公开数据分析，结论仅供参考。",
    style="warning"
)

# 3. 定义 HTML 渲染逻辑
def render_callout_html(block: dict) -> str:
    """渲染 callout 为 HTML"""
    style = block.get("style", "info")
    content = block.get("content", "")

    style_map = {
        "info": "background-color: #e3f2fd;",
        "warning": "background-color: #fff3cd;",
        "success": "background-color: #d4edda;",
        "error": "background-color: #f8d7da;"
    }

    return f'<div class="callout" style="{style_map.get(style, "")}">{content}</div>'

# 测试
print(render_callout_html(callout_block))
```

## 常见问题

### Q1: IR 校验失败？

**排查步骤**：

```python
# 1. 检查块类型是否有效
from ReportEngine.ir.schema import BLOCK_TYPES
block_type = "my_custom_type"  # 替换为你的类型
if block_type not in BLOCK_TYPES:
    print(f"无效的块类型: {block_type}")

# 2. 检查必需字段
required_fields = {
    "heading": ["level", "content"],
    "paragraph": ["content"],
    "chart": ["chart_type", "data"]
}

# 3. 使用详细错误信息
from ReportEngine.ir.validator import IRValidator
validator = IRValidator()
result = validator.validate_document(your_ir)
print(result.errors)  # 查看具体错误
```

### Q2: 渲染输出异常？

**解决方案**：

```python
# 1. 验证 IR 结构正确
from ReportEngine.ir.validator import IRValidator
result = IRValidator().validate_document(ir)

if not result.is_valid:
    print("IR 校验失败，请修复后再渲染")
    print(result.errors)

# 2. 检查特殊字符
# 确保文本中的特殊字符正确转义
import json
safe_json = json.dumps(ir, ensure_ascii=False)

# 3. 测试单个块
from ReportEngine.renderers.html_renderer import HTMLRenderer
renderer = HTMLRenderer()
for section in ir['sections']:
    try:
        html = renderer._render_block(section)
        print(f"{section['type']}: OK")
    except Exception as e:
        print(f"{section['type']}: {e}")
```

### Q3: 章节顺序错乱？

**原因**：manifest 中的章节顺序与实际文件不匹配

```python
# 检查 manifest
from ReportEngine.core.chapter_storage import ChapterStorage
storage = ChapterStorage()
run_id = "20240120_120000"

manifest = storage.load_manifest(run_id)
print("Manifest 章节顺序:")
for chapter in manifest['chapters']:
    print(f"  - {chapter['slug']}: {chapter['title']}")

# 确保文件按顺序组装
from ReportEngine.core.stitcher import DocumentComposer
composer = DocumentComposer()
ir = composer.stitch_document(run_id, manifest)

# 验证组装后的顺序
print("\n组装后的章节:")
for section in ir['sections']:
    if section['type'] == 'heading':
        print(f"  - {section['content']}")
```

## 相关文档

- **[ReportEngine](report-engine.md)** - 报告引擎详解
- **[模板系统](templates.md)** - 模板开发指南
- **[核心概念](../getting-started/concepts.md)** - IR 概念说明

---

> 💡 **学习提示**：IR 是理解报告生成系统的关键。建议先掌握基本块类型的使用，再学习如何校验和扩展 IR。

## 常见问题

### Q1: IR 校验失败？

- 检查块类型是否有效
- 确认必需字段完整
- 查看详细错误信息

### Q2: 渲染输出异常？

- 确认 IR 结构正确
- 检查特殊字符转义
- 验证渲染器配置

### Q3: 章节顺序错乱？

- 检查 manifest 中的章节顺序
- 确认 slug 命名规范
- 验证组装逻辑

## 相关文档

- **[ReportEngine](report-engine.md)** - 报告引擎详解
- **[模板系统](templates.md)** - 模板开发指南
- **[核心概念](../getting-started/concepts.md)** - IR 概念说明
