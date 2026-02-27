# BettaFish 微舆系统文档

欢迎来到 BettaFish（微舆情）系统文档！本文档库将帮助你从零开始了解和使用这个多智能体舆情分析系统。

## 学习目标

通过本文档库的学习，你将能够：

- [ ] **理解** BettaFish 的系统架构和设计理念
- [ ] **部署** 并运行完整的舆情分析系统
- [ ] **使用** 三大分析引擎进行多角度分析
- [ ] **定制** 报告模板满足特定需求
- [ ] **扩展** 系统功能，添加新的分析模块
- [ ] **排查** 和解决常见问题

## 文档导航

### 📖 入门指南

如果你是第一次接触 BettaFish，建议按以下顺序阅读：

1. **[系统概述](getting-started/overview.md)** - 了解系统的核心概念、架构和设计理念
2. **[快速开始](getting-started/quickstart.md)** - 5 分钟快速部署和运行
3. **[核心概念](getting-started/concepts.md)** - 掌握 Agent、论坛机制、报告生成等核心概念

### 🤖 分析引擎

深入了解三大分析引擎的工作原理：

- **[QueryEngine 查询引擎](engines/query-engine.md)** - Web/新闻搜索代理，使用 Tavily API
- **[MediaEngine 多模态引擎](engines/media-engine.md)** - 多模态内容分析，使用 Bocha API
- **[InsightEngine 洞察引擎](engines/insight-engine.md)** - 私有数据库挖掘，带情感分析

### 🎭 论坛协作机制

- **[ForumEngine 论坛引擎](forum/forum-engine.md)** - Agent 协作机制与主持人模型

### 📄 报告生成系统

- **[ReportEngine 报告引擎](reporting/report-engine.md)** - 多阶段报告生成与模板系统
- **[报告模板](reporting/templates.md)** - 内置模板与自定义模板开发
- **[IR 架构](reporting/ir-architecture.md)** - 文档中间表示设计

### 🕷️ 爬虫系统

- **[MindSpider 爬虫系统](crawler/mindspider.md)** - 社交媒体数据采集

### 👨‍💻 开发者指南

- **[配置指南](development/config.md)** - 环境变量与配置说明
- **[API 参考](api/overview.md)** - 核心 API 文档
- **[部署指南](development/deployment.md)** - Docker 部署与生产环境配置
- **[故障排查](development/troubleshooting.md)** - 常见问题与解决方案

## 系统架构概览

```
┌─────────────────────────────────────────────────────────────────────┐
│                           BettaFish 系统架构                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐         │
│   │ QueryEngine  │    │ MediaEngine  │    │ InsightEngine│         │
│   │  (Web 搜索)  │    │ (多模态分析) │    │  (数据挖掘)  │         │
│   └──────┬───────┘    └──────┬───────┘    └──────┬───────┘         │
│          │                   │                   │                  │
│          └───────────────────┼───────────────────┘                  │
│                              ▼                                       │
│                   ┌──────────────────┐                               │
│                   │  ForumEngine     │                               │
│                   │  (论坛协作机制)   │                               │
│                   └─────────┬────────┘                               │
│                             │                                        │
│                             ▼                                        │
│                   ┌──────────────────┐                               │
│                   │  ReportEngine    │                               │
│                   │  (报告生成)       │                               │
│                   └─────────┬────────┘                               │
│                             │                                        │
│                             ▼                                        │
│                    ┌───────────────┐                                 │
│                    │  交互式报告    │                                 │
│                    │  (HTML/PDF)   │                                 │
│                    └───────────────┘                                 │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

## 核心特性

| 特性 | 说明 |
|------|------|
| **AI 驱动的全域监控** | 覆盖 10+ 国内外主流社媒平台 |
| **复合分析引擎** | LLM + 微调模型 + 统计模型协同工作 |
| **多模态能力** | 支持视频、图片、文本的综合分析 |
| **Agent 论坛协作** | 独特的"论坛"机制实现多 Agent 辩论与融合 |
| **公私域融合** | 支持内部业务数据库与舆情数据无缝集成 |
| **轻量化框架** | 模块化设计，易于扩展和定制 |

## 学习路径

### 用户路径 ⭐ → ⭐⭐⭐⭐

1. 阅读[快速开始](getting-started/quickstart.md)部署系统
2. 阅读[核心概念](getting-started/concepts.md)理解基本术语
3. 通过 Web 界面提交第一个分析任务
4. 阅读[报告模板](reporting/templates.md)了解报告定制

### 开发路径 ⭐ → ⭐⭐⭐⭐

1. 阅读[系统概述](getting-started/overview.md)了解架构
2. 阅读[配置指南](development/config.md)配置环境
3. 阅读各引擎文档，理解[节点模式](engines/query-engine.md#节点架构)
4. 阅读[API 参考](api/overview.md)学习二次开发

### 进阶路径 ⭐ → ⭐⭐⭐⭐

1. 深入研究[ForumEngine](forum/forum-engine.md)协作机制
2. 研究[IR 架构](reporting/ir-architecture.md)设计思想
3. 学习如何添加新的分析引擎
4. 贡献代码和文档改进

## 版本信息

- **当前版本**：v1.2.1
- **许可证**：GPL-2.0
- **Python 要求**：3.9+

## 获取帮助

- **GitHub Issues**：[提交问题](https://github.com/666ghj/BettaFish/issues)
- **邮箱**：hangjiang@bupt.edu.cn
- **QQ 群**：见项目主页二维码

## 目录结构

```
docs/
├── README.md                    # 本文档
├── getting-started/             # 入门指南
│   ├── overview.md              # 系统概述
│   ├── quickstart.md            # 快速开始
│   └── concepts.md              # 核心概念
├── engines/                     # 分析引擎
│   ├── query-engine.md          # QueryEngine 文档
│   ├── media-engine.md          # MediaEngine 文档
│   └── insight-engine.md        # InsightEngine 文档
├── forum/                       # 论坛协作
│   └── forum-engine.md          # ForumEngine 文档
├── reporting/                   # 报告生成
│   ├── report-engine.md         # ReportEngine 文档
│   ├── templates.md             # 模板系统
│   └── ir-architecture.md       # IR 架构
├── crawler/                     # 爬虫系统
│   └── mindspider.md            # MindSpider 文档
├── development/                 # 开发指南
│   ├── config.md                # 配置指南
│   ├── deployment.md            # 部署指南
│   └── troubleshooting.md       # 故障排查
└── api/                         # API 参考
    └── overview.md              # API 概览
```

## 实践练习

### 练习 1：文档导航挑战 ⭐

**任务**：根据不同需求找到合适的文档

| 需求 | 应该阅读的文档 |
|------|---------------|
| 我想快速了解系统是什么 | |
| 我想立即部署系统 | |
| 我想理解多 Agent 如何协作 | |
| 我想添加新的分析引擎 | |
| 报告生成失败了怎么办 | |
| 我想自定义报告模板 | |
| 我需要调用 API 集成系统 | |

<details>
<summary>查看答案</summary>

| 需求 | 应该阅读的文档 |
|------|---------------|
| 我想快速了解系统是什么 | [系统概述](getting-started/overview.md) |
| 我想立即部署系统 | [快速开始](getting-started/quickstart.md) |
| 我想理解多 Agent 如何协作 | [ForumEngine](forum/forum-engine.md) |
| 我想添加新的分析引擎 | [QueryEngine](engines/query-engine.md) → 节点架构 |
| 报告生成失败了怎么办 | [故障排查](development/troubleshooting.md) |
| 我想自定义报告模板 | [报告模板](reporting/templates.md) |
| 我需要调用 API 集成系统 | [API 参考](api/overview.md) |

</details>

---

### 练习 2：学习路径规划 ⭐⭐

**任务**：根据你的角色制定学习计划

```markdown
## 我的角色：__________

### 第 1 周目标
- [ ] 阅读并理解 ________ 文档
- [ ] 完成 ________ 练习
- [ ] 能力达成：____________

### 第 2 周目标
- [ ] 阅读并理解 ________ 文档
- [ ] 完成 ________ 练习
- [ ] 能力达成：____________

### 第 3 周目标
- [ ] 阅读并理解 ________ 文档
- [ ] 完成 ________ 练习
- [ ] 能力达成：____________
```

**参考计划**：

**产品经理角色**：
```
第 1 周: overview.md → concepts.md → forum-engine.md
第 2 周: report-engine.md → templates.md
第 3 周: 实践运行完整分析流程
```

**后端开发角色**：
```
第 1 周: quickstart.md → config.md → deployment.md
第 2 周: query-engine.md → insight-engine.md
第 3 周: api/overview.md → 开发自定义节点
```

**数据分析师角色**：
```
第 1 周: quickstart.md → concepts.md
第 2 周: insight-engine.md → report-engine.md
第 3 周: 实践复杂数据分析任务
```

---

### 练习 3：知识自测 ⭐⭐

**问题 1**：BettaFish 的三大分析引擎是？

<details>
<summary>查看答案</summary>

A. QueryEngine、MediaEngine、InsightEngine

B. SearchEngine、AnalyzeEngine、ReportEngine

C. WebEngine、DatabaseEngine、AIEngine

**正确答案：A**

</details>

---

**问题 2**：ForumEngine 的作用是？

<details>
<summary>查看答案</summary>

ForumEngine 是多 Agent 协作机制，通过"论坛"形式让三个分析引擎进行讨论和辩论，由 LLM 主持人进行协调和总结。

</details>

---

**问题 3**：IR（中间表示）的作用是什么？

<details>
<summary>查看答案</summary>

IR 是格式无关的文档中间表示，使得同一份报告可以渲染成 HTML、PDF、Markdown 等多种格式。它使用 JSON 结构定义文档的各个部分（标题、段落、图表等）。

</details>

---

## 进度追踪

使用以下追踪表记录你的学习进度：

```markdown
### 入门阶段 ⭐

- [ ] [ ] 系统概述 - 已完成 / 进行中 / 待开始
- [ ] [ ] 快速开始 - 已完成 / 进行中 / 待开始
- [ ] [ ] 核心概念 - 已完成 / 进行中 / 待开始

### 核心阶段 ⭐⭐

- [ ] [ ] QueryEngine 文档
- [ ] [ ] MediaEngine 文档
- [ ] [ ] InsightEngine 文档

### 进阶阶段 ⭐⭐⭐

- [ ] [ ] ForumEngine 文档
- [ ] [ ] ReportEngine 文档
- [ ] [ ] IR 架构文档

### 专家阶段 ⭐⭐⭐⭐

- [ ] [ ] 配置与部署
- [ ] [ ] API 开发
- [ ] [ ] 系统扩展
```

## 文档贡献

欢迎改进文档！请遵循以下规范：

1. **风格一致**：遵循现有文档的写作风格
2. **代码示例**：确保代码可运行
3. **练习验证**：添加的练习需提供参考答案
4. **中英文术语**：首次出现时标注英文原文

贡献方式：
1. Fork 项目
2. 修改文档
3. 提交 Pull Request

---

> 💡 **学习提示**：建议按照用户路径 → 开发路径 → 进阶路径的顺序学习。每完成一个阶段后，尝试对应的实践练习巩固知识。

