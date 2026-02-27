# 模板系统

ReportEngine 的模板系统定义了报告的结构框架。系统通过解析 Markdown 模板自动生成报告结构。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解模板系统的解析机制
- [ ] 掌握模板编写的基本规范
- [ ] 学会创建自定义报告模板
- [ ] 了解模板自动选择的工作原理
- [ ] 能够优化模板以获得更好的生成效果

## 概述

### 模板的作用

模板定义了：

1. **报告结构**：章节划分和层级关系
2. **内容要求**：每个章节应包含的内容
3. **生成指引**：LLM 生成章节时的参考

### 设计原则

| 原则 | 说明 | 实践 |
|------|------|------|
| **简洁性** | 使用标准 Markdown 格式 | 避免复杂的语法 |
| **可读性** | 模板本身应清晰易懂 | 添加清晰的说明 |
| **可维护性** | 易于修改和扩展 | 模块化设计 |
| **自动识别** | 系统自动发现新模板 | 放在指定目录即可 |

### 模板解析流程

```
Markdown 模板文件
       │
       ▼
┌──────────────────┐
│  模板文件读取     │
│  - 扫描目录       │
│  - 识别 .md 文件  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  模板解析         │
│  - 按章节分割     │
│  - 生成 slug      │
│  - 提取要求       │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  章节对象列表     │
│  [               │
│   {              │
│     "slug": "...",│
│     "title": "...",│
│     "content": "..."│
│   }, ...          │
│  ]               │
└──────────────────┘
```

## 模板格式

### 基本结构

模板使用标准 Markdown 格式，以 `## `（二级标题）作为章节分隔：

```markdown
# 报告主标题

## 1. 第一章节标题

本章的要求和说明...

### 子章节

子章节的要求...

## 2. 第二章节标题

本章的要求和说明...
```

### 解析规则

```python
def parse_template_sections(template_path: str) -> List[TemplateSection]:
    """
    解析模板规则：

    1. 以 ## 开头的行作为章节分隔符
    2. ## 后的内容作为章节标题
    3. 章节间的内容作为生成要求
    4. 自动生成唯一 slug
    """
```

### Slug 生成

每个章节会自动生成唯一标识符：

```python
def generate_slug(title: str) -> str:
    """
    从标题生成 slug

    示例：
    "1. 总体概况" → "section-1-summary"
    "2. 详细分析" → "section-2-analysis"
    """
```

## 内置模板

### 1. 企业品牌声誉分析报告

**适用场景**：品牌形象、声誉监测

**章节结构**：
- 执行摘要
- 品牌形象分析
- 舆情趋势分析
- 平台对比分析
- 结论与建议

### 2. 市场竞争格局舆情分析报告

**适用场景**：竞品分析、市场研究

**章节结构**：
- 市场概况
- 竞品对比
- 用户反馈分析
- 优势劣势分析
- 战略建议

### 3. 日常或定期舆情监测报告

**适用场景**：定期舆情汇总

**章节结构**：
- 监测概述
- 本期热点
- 情感分布
- 风险提示
- 下期关注

### 4. 特定政策或行业动态舆情分析报告

**适用场景**：政策影响评估

**章节结构**：
- 政策背景
- 舆情反响
- 影响分析
- 趋势预测
- 应对建议

### 5. 社会公共热点事件分析报告

**适用场景**：热点事件追踪

**章节结构**：
- 事件概述
- 发展脉络
- 舆论焦点
- 情感演变
- 舆情总结

### 6. 突发事件与危机公关舆情报告

**适用场景**：危机公关支持

**章节结构**：
- 事件概述
- 舆情态势
- 负面分析
- 应对措施
- 后续关注

## 创建自定义模板

### 步骤

1. 在 `ReportEngine/report_template/` 创建新文件
2. 使用 Markdown 格式编写模板
3. 以 `## ` 分隔章节
4. 为每章添加内容要求

### 模板示例

```markdown
# [模板名称] 报告模板

## 1. 执行摘要

请简要概括本报告的主要发现和结论。包括：
- 核心发现（3-5点）
- 总体结论
- 关键建议

字数要求：300-500字

## 2. 背景分析

请从以下方面进行分析：
- 事件/话题背景
- 相关时间线
- 主要参与方

## 3. 详细分析

### 3.1 数据概览

请提供以下数据：
- 总体数据量
- 时间范围
- 覆盖平台

### 3.2 情感分析

请分析：
- 情感分布比例
- 情感变化趋势
- 影响因素

## 4. 结论与建议

请总结：
- 主要结论
- 风险提示
- 行动建议
```

### 模板编写最佳实践

| 实践 | 说明 |
|------|------|
| **明确要求** | 清晰说明每章应包含的内容 |
| **量化指标** | 指定字数、数据点等具体要求 |
| **结构化** | 使用子章节组织复杂内容 |
| **可操作** | 提供具体的分析维度和角度 |

## 模板选择机制

### 自动选择

ReportEngine 根据查询内容自动选择最合适的模板：

```python
class TemplateSelectionNode(BaseNode):
    def run(self, state: ReportState) -> ReportState:
        """
        模板选择逻辑：

        1. 分析查询内容
        2. 提取关键词和主题
        3. 匹配模板特征
        4. 选择最匹配的模板
        """
```

### 匹配特征

| 模板 | 关键词特征 |
|------|-----------|
| 企业品牌声誉 | 品牌、形象、声誉 |
| 市场竞争格局 | 竞品、竞争、市场 |
| 日常舆情监测 | 定期、汇总、周报 |
| 政策舆情分析 | 政策、法规、标准 |
| 热点事件分析 | 热点、事件、突发 |
| 危机公关舆情 | 危机、公关、负面 |

### 手动选择

用户也可以在 Web 界面手动选择模板。

## 模板与章节生成

### 章节生成指令

系统根据模板生成章节生成指令：

```python
def generate_chapter_prompt(
    section: TemplateSection,
    context: Dict
) -> str:
    """
    生成章节提示词：

    1. 章节标题
    2. 内容要求（从模板提取）
    3. 字数限制（从 WordBudgetNode 获取）
    4. 上下文信息（其他引擎的输出）
    """
```

### 提示词结构

```
你是一位专业的{角色}分析师。

请根据以下信息生成报告章节：

# 章节信息
标题：{section.title}
要求：{section.content}

# 字数要求
本章字数：{word_count}字

# 参考材料
{context.materials}

# 输出格式
请按照以下 JSON 格式输出：
{
  "blocks": [
    {"type": "paragraph", "content": "..."},
    {"type": "heading", "level": 3, "content": "..."},
    ...
  ]
}
```

## 高级功能

### 条件章节

某些章节可以根据条件决定是否生成：

```markdown
## [可选] 深度分析

如果数据量充足，请进行深度分析...
```

### 动态内容

模板可以引用动态内容：

```markdown
## 数据统计

本报告分析了 {data_count} 条数据，
时间跨度为 {date_range}。
```

### 子章节嵌套

支持多级子章节：

```markdown
## 主章节

### 子章节 1.1

内容要求...

#### 小节 1.1.1

内容要求...
```

## 模板管理

### 模板目录

```
ReportEngine/report_template/
├── 企业品牌声誉分析报告.md
├── 市场竞争格局舆情分析报告.md
├── 日常或定期舆情监测报告.md
├── 特定政策或行业动态舆情分析报告.md
├── 社会公共热点事件分析报告.md
├── 突发事件与危机公关舆情报告.md
└── custom_template.md  # 自定义模板
```

### 模板验证

系统会在启动时验证模板：

- 检查文件格式
- 验证章节结构
- 生成 slug 映射

```bash
# 验证模板
python -m ReportEngine.core.template_parser --validate
```

## 常见问题

### Q1: 模板未被识别？

**排查步骤**：

```bash
# 1. 确认文件在正确目录
ls -la ReportEngine/report_template/

# 2. 检查文件格式
file 你的模板.md  # 应该显示 Markdown 文件

# 3. 验证章节分隔符
grep "^## " 你的模板.md  # 应该有输出

# 4. 测试解析
python -c "
from ReportEngine.core.template_parser import parse_template_sections
sections = parse_template_sections('ReportEngine/report_template/你的模板.md')
print(f'解析到 {len(sections)} 个章节')
for s in sections:
    print(f'  - {s.slug}: {s.title}')
"
```

### Q2: 生成的章节不符合要求？

**优化策略**：

| 问题 | 解决方案 |
|------|----------|
| 内容太简单 | 增加具体要求和示例 |
| 内容太冗长 | 添加字数限制 |
| 格式混乱 | 明确指定输出格式 |
| 缺少关键信息 | 列出必需包含的要点 |

```markdown
## 优化后的模板示例

### 数据分析

请对收集的数据进行深入分析，必须包含以下内容：

**必需内容**：
1. 数据总量统计（具体数字）
2. 时间范围（起止日期）
3. 覆盖平台列表（至少3个）

**分析要求**：
- 从至少2个角度分析数据
- 使用具体数据支持你的结论
- 识别至少3个关键趋势或模式

**输出格式**：
- 使用列表呈现统计数据
- 使用段落描述分析结论
- 总字数控制在300-500字
```

### Q3: 如何调试模板？

```python
# 调试模板解析
from ReportEngine.core.template_parser import parse_template_sections

# 解析模板
sections = parse_template_sections("你的模板.md")

# 查看解析结果
print(f"=== 模板解析结果 ===")
print(f"章节数: {len(sections)}\n")

for i, section in enumerate(sections, 1):
    print(f"章节 {i}:")
    print(f"  Slug: {section.slug}")
    print(f"  标题: {section.title}")
    print(f"  要求预览: {section.content[:100]}...")
    print()
```

## 实践练习

### 练习 1：解析现有模板 ⭐

**任务**：解析内置模板，理解其结构

```python
from ReportEngine.core.template_parser import parse_template_sections
import os

# 列出所有模板
template_dir = "ReportEngine/report_template"
templates = [f for f in os.listdir(template_dir) if f.endswith('.md')]

print(f"=== 发现 {len(templates)} 个模板 ===\n")

for template in templates:
    path = os.path.join(template_dir, template)
    sections = parse_template_sections(path)

    print(f"模板: {template}")
    print(f"  章节数: {len(sections)}")
    print(f"  章节列表:")
    for s in sections:
        print(f"    - {s.title}")
    print()
```

### 练习 2：创建自定义模板 ⭐⭐

**任务**：创建一个产品分析报告模板

```python
template_content = """# 产品分析报告

## 1. 产品概述

请简要介绍产品的基本信息：
- 产品名称和定位
- 主要功能特点
- 目标用户群体

## 2. 市场表现

### 2.1 销售数据

请分析产品的销售情况，包括：
- 销售额和销量
- 同比/环比变化
- 市场份额

### 2.2 用户反馈

请分析用户反馈，包括：
- 正面评价汇总（至少3条）
- 负面评价汇总（至少3条）
- 改进建议提取

## 3. 竞品对比

请与至少2个主要竞品进行对比：
- 功能对比
- 价格对比
- 用户评价对比

## 4. 结论与建议

请基于以上分析，提出：
- 核心竞争优势（2-3点）
- 主要不足（2-3点）
- 具体改进建议（至少3条）
"""

# 保存模板
import os
template_path = "ReportEngine/report_template/产品分析报告.md"
os.makedirs(os.path.dirname(template_path), exist_ok=True)

with open(template_path, 'w', encoding='utf-8') as f:
    f.write(template_content)

print(f"模板已创建: {template_path}")
print("\n验证解析:")
from ReportEngine.core.template_parser import parse_template_sections
sections = parse_template_sections(template_path)
print(f"  解析到 {len(sections)} 个章节")
```

### 练习 3：优化模板生成效果 ⭐⭐⭐

**任务**：对比不同版本的模板，分析对生成效果的影响

<details>
<summary>查看完整代码与预期输出</summary>

```python
# 版本A：简单模板
template_v1 = """
## 分析

请分析数据。
"""

# 版本B：详细模板
template_v2 = """
## 数据分析

请从以下维度分析数据：

1. **整体趋势**：描述数据的整体变化趋势
2. **关键发现**：列出至少3个关键发现
3. **异常点**：识别任何异常数据点并解释

请确保每个分析点都有数据支持。
"""

# 测试不同版本的效果
from ReportEngine.core.template_parser import parse_template_sections

sections_v1 = parse_template_sections(template_v1)
sections_v2 = parse_template_sections(template_v2)

print("=== 模板对比 ===")
print(f"版本A - 章节数: {len(sections_v1)}")
print(f"版本A - 内容长度: {len(sections_v1[0].content)} 字符")

print(f"\n版本B - 章节数: {len(sections_v2)}")
print(f"版本B - 内容长度: {len(sections_v2[0].content)} 字符")

print("\n结论:")
print("版本B 提供了更详细的指导，可能会生成更完整的内容。")
```

**预期输出**：
```
=== 模板对比 ===
版本A - 章节数: 1
版本A - 内容长度: 5 字符

版本B - 章节数: 1
版本B - 内容长度: 65 字符

结论:
版本B 提供了更详细的指导，可能会生成更完整的内容。
```

**关键发现**：
- 版本A内容太简略，LLM 无法理解具体要求
- 版本B提供了明确的分析维度和数量要求
- 内容长度差异显著影响生成质量

</details>

## 相关文档

- **[ReportEngine](report-engine.md)** - 报告引擎详解
- **[IR 架构](ir-architecture.md)** - 中间表示详解
- **[核心概念](../../getting-started/concepts.md)** - 报告生成概念

---

> 💡 **学习提示**：编写模板就像编写提示词，需要不断迭代优化。建议先从简单模板开始，逐步增加细节要求，观察生成效果的变化。
