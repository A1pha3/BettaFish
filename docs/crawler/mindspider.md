# MindSpider 爬虫系统

MindSpider 是 BettaFish 的社交媒体数据采集系统，支持 10+ 主流社交平台的数据爬取。

## 学习目标

完成本章学习后，你将能够：

- [ ] 理解 MindSpider 的架构设计
- [ ] 掌握话题提取和深度爬取的使用方法
- [ ] 了解各平台爬虫的配置方式
- [ ] 学会排查和解决爬虫常见问题
- [ ] 能够扩展新平台支持

## 概述

### 核心功能

MindSpider 负责：

- **话题提取**：从新闻网站提取热点话题
- **深度爬取**：爬取社交媒体的详细内容
- **数据存储**：将数据存入数据库
- **多平台支持**：支持微博、小红书、抖音等平台

### 在系统中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                    BettaFish 数据采集层                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────┐        ┌──────────────┐                │
│   │ 新闻网站      │        │ 社交媒体      │                │
│   │ (热点源)      │        │ (数据源)      │                │
│   └──────┬───────┘        └──────┬───────┘                │
│          │                      │                          │
│          ▼                      ▼                          │
│   ┌──────────────────────────────────────┐                 │
│   │         MindSpider 爬虫系统          │                 │
│   │                                      │                 │
│   │  ┌────────────────┐  ┌─────────────┐│                 │
│   │  │BroadTopicExtraction│  │DeepSentiment││                 │
│   │  │话题提取          │  │深度爬取      ││                 │
│   │  └────────────────┘  └─────────────┘│                 │
│   │                                      │                 │
│   └──────────────┬───────────────────────┘                 │
│                  │                                         │
│                  ▼                                         │
│   ┌──────────────────────────────────────────┐             │
│   │        PostgreSQL/MySQL 数据库          │             │
│   │  (InsightEngine 的数据来源)             │             │
│   └──────────────────────────────────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 支持的平台

| 代码 | 平台 | 类型 | 状态 |
|------|------|------|------|
| `xhs` | 小红书 | 社区电商 | ✅ |
| `dy` | 抖音 | 短视频 | ✅ |
| `ks` | 快手 | 短视频 | ✅ |
| `bili` | B站 | 中长视频 | ✅ |
| `wb` | 微博 | 社交网络 | ✅ |
| `tieba` | 贴吧 | 社区论坛 | ✅ |
| `zhihu` | 知乎 | 问答社区 | ✅ |

## 架构设计

### 目录结构

```
MindSpider/
├── main.py                   # 主程序入口
├── config.py                 # 爬虫配置
├── BroadTopicExtraction/     # 话题提取模块
│   ├── main.py
│   ├── database_manager.py
│   ├── get_today_news.py
│   └── topic_extractor.py
├── DeepSentimentCrawling/    # 深度爬取模块
│   ├── main.py
│   ├── keyword_manager.py
│   ├── platform_crawler.py
│   └── MediaCrawler/         # 爬虫核心（子模块）
│       ├── main.py
│       ├── config/           # 各平台配置
│       └── media_platform/   # 各平台爬虫实现
└── schema/                   # 数据库结构
    ├── db_manager.py
    ├── init_database.py
    ├── models_bigdata.py
    └── models_sa.py
```

## 核心组件

### 1. BroadTopicExtraction

从新闻网站提取热点话题。

```python
class TopicExtractor:
    """话题提取器"""

    def extract_topics(self, date: str = None) -> List[Topic]:
        """
        提取热点话题

        Args:
            date: 目标日期，格式 YYYY-MM-DD

        Returns:
            话题列表
        """
```

### 2. DeepSentimentCrawling

对社交媒体进行深度爬取。

```python
class PlatformCrawler:
    """平台爬虫管理器"""

    def crawl_platform(self, platform: str, keywords: List[str]):
        """
        爬取指定平台

        Args:
            platform: 平台代码
            keywords: 关键词列表
        """
```

### 3. MediaCrawler

爬虫核心实现（Git 子模块）。

## 使用方法

### 初始化

```bash
cd MindSpider
python main.py --setup
```

### 完整流程

```bash
# 运行完整爬虫流程
python main.py --complete

# 指定日期
python main.py --complete --date 2024-01-20

# 测试模式（数据量少）
python main.py --complete --test
```

### 话题提取

```bash
# 提取今日热点
python main.py --broad-topic

# 指定日期
python main.py --broad-topic --date 2024-01-20
```

### 深度爬取

```bash
# 爬取所有平台
python main.py --deep-sentiment

# 爬取指定平台
python main.py --deep-sentiment --platforms xhs dy wb

# 指定日期范围
python main.py --deep-sentiment --start-date 2024-01-01 --end-date 2024-01-31
```

## 数据库结构

### 主要表结构

| 表名 | 说明 | 主要字段 |
|------|------|----------|
| `daily_topics` | 每日话题 | topic_id, topic, date |
| `xhs_note` | 小红书笔记 | note_id, title, content, likes |
| `dy_video` | 抖音视频 | video_id, title, desc, stats |
| `wb_weibo` | 微博内容 | weibo_id, text, reposts_count |
| `bili_video` | B站视频 | bvid, title, desc, view |

### ORM 模型

```python
# MindSpider/schema/models_bigdata.py

class XhsNote(Base):
    """小红书笔记表"""
    __tablename__ = 'xhs_note'

    note_id = Column(String(64), primary_key=True)
    title = Column(String(255))
    content = Column(Text)
    likes = Column(Integer)
    collects = Column(Integer)
    ...
```

## 配置说明

### 环境变量

```bash
# MindSpider LLM 配置（推荐 DeepSeek）
MINDSPIDER_API_KEY=your_api_key
MINDSPIDER_BASE_URL=https://api.deepseek.com
MINDSPIDER_MODEL_NAME=deepseek-chat

# 数据库配置
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=your_password
DB_NAME=bettafish
DB_DIALECT=postgresql
```

### 爬虫配置

```python
# MindSpider/config.py

class CrawlerConfig:
    # 爬取限制
    max_notes_per_keyword = 100
    max_comments_per_note = 50

    # 并发配置
    max_concurrent_tasks = 3

    # 重试配置
    max_retries = 3
    retry_delay = 5
```

## 平台配置

各平台需要单独配置登录信息：

```python
# MediaCrawler/config/

# 小红书
XHS_COOKIES = "your_cookies"

# 抖音
DY_COOKIES = "your_cookies"

# 微博
WB_COOKIES = "your_cookies"
```

## 数据流转

```
┌─────────────────────────────────────────────────────────────┐
│                    MindSpider 数据流程                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                          │
│  │ 新闻网站      │                                          │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────────┐                                     │
│  │BroadTopicExtraction│                                   │
│  │ - 提取热点话题    │                                     │
│  │ - 存入数据库      │                                     │
│  └──────┬───────────┘                                     │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────────┐                                     │
│  │DailyTopic 表     │                                     │
│  └──────┬───────────┘                                     │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────────┐     ┌──────────────┐               │
│  │DeepSentimentCrawling│──→│ MediaCrawler │               │
│  │ - 获取话题列表   │     │ - 各平台爬虫 │               │
│  │ - 分配爬取任务   │     └──────┬───────┘               │
│  └──────┬───────────┘              │                      │
│         │                          │                      │
│         ▼                          ▼                      │
│  ┌──────────────────────────────────────────┐             │
│  │        各平台数据采集                     │             │
│  │  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐    │             │
│  │  │ xhs │ │ dy │ │ ks │ │bili│ │ wb │ ...│             │
│  │  └────┘ └────┘ └────┘ └────┘ └────┘    │             │
│  └──────────────────┬─────────────────────┘             │
│                     │                                    │
│                     ▼                                    │
│  ┌──────────────────────────────────────────┐             │
│  │          PostgreSQL/MySQL 数据库          │             │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐   │             │
│  │  │xhs_  │ │dy_   │ │wb_   │ │bili_ │   │             │
│  │  │note  │ │video │ │weibo │ │video │   │             │
│  │  └──────┘ └──────┘ └──────┘ └──────┘   │             │
│  └──────────────────────────────────────────┘             │
│                     │                                    │
│                     ▼                                    │
│  ┌──────────────────────────────────────────┐             │
│  │        InsightEngine 分析                 │             │
│  └──────────────────────────────────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 实践练习

### 练习 1：初始化 MindSpider ⭐

**任务**：完成 MindSpider 的初始化配置

```bash
cd MindSpider

# 1. 项目初始化
python main.py --setup

# 2. 验证数据库连接
python -c "
from schema.db_manager import db_manager
print('数据库连接成功')
"

# 3. 检查表是否创建
python -c "
from schema.models_bigdata import XhsNote, DyVideo, WbWeibo
print('ORM 模型加载成功')
"
```

### 练习 2：执行测试爬取 ⭐⭐

**任务**：运行测试模式的深度爬取

```bash
cd MindSpider

# 测试模式（数据量少，速度快）
python main.py --complete --test

# 检查爬取结果
python -c "
from InsightEngine.utils.db import execute_query
result = execute_query('SELECT COUNT(*) as cnt FROM xhs_note')
print(f'小红书数据: {result[0][\"cnt\"]} 条')
result = execute_query('SELECT COUNT(*) as cnt FROM wb_weibo')
print(f'微博数据: {result[0][\"cnt\"]} 条')
"
```

### 练习 3：自定义爬取任务 ⭐⭐⭐

**任务**：创建一个自定义的爬取脚本

```python
# custom_crawl.py
import sys
sys.path.append('MindSpider')

from DeepSentimentCrawling.platform_crawler import PlatformCrawler
from datetime import datetime, timedelta

def custom_crawl_task(topic: str, days_back: int = 7):
    """自定义爬取任务

    Args:
        topic: 爬取主题
        days_back: 爬取最近N天的数据
    """
    # 计算日期范围
    end_date = datetime.now()
    start_date = end_date - timedelta(days=days_back)

    # 配置爬取参数
    config = {
        "platforms": ["xhs", "wb"],  # 只爬取小红书和微博
        "keywords": [topic],
        "start_date": start_date.strftime("%Y-%m-%d"),
        "end_date": end_date.strftime("%Y-%m-%d"),
        "max_notes": 50  # 每个平台最多50条
    }

    # 执行爬取
    crawler = PlatformCrawler()
    results = crawler.crawl_multiple_platforms(config)

    # 输出统计
    print("\n=== 爬取结果统计 ===")
    for platform, data in results.items():
        print(f"{platform}: {len(data)} 条")

# 运行
if __name__ == "__main__":
    custom_crawl_task("人工智能", days_back=3)
```

## 常见问题

### Q1: 爬虫无法启动？

**症状**：
```
playwright._impl._api_types.Error: Executable doesn't exist
```

**解决方案**：

```bash
# 1. 安装 Playwright
pip install playwright

# 2. 安装浏览器
playwright install chromium

# 3. 如果国内网络慢，使用镜像
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright/
playwright install chromium
```

### Q2: 数据未保存？

**排查步骤**：

```bash
# 1. 检查数据库连接
python -c "
from InsightEngine.utils.db import execute_query
try:
    result = execute_query('SELECT 1')
    print('数据库连接正常')
except Exception as e:
    print(f'数据库连接失败: {e}')
"

# 2. 检查表是否存在
python -c "
from InsightEngine.utils.db import execute_query
result = execute_query(\"\"\"
    SELECT table_name FROM information_schema.tables
    WHERE table_schema = 'public'
    AND table_name IN ('xhs_note', 'dy_video', 'wb_weibo')
\"\"\")
for row in result:
    print(f'表存在: {row[0]}')
"

# 3. 查看爬虫日志
tail -n 50 MindSpider/logs/crawler.log
```

### Q3: 爬取速度慢？

**优化方案**：

```python
# MindSpider/config.py

# 1. 调整并发配置
MAX_CONCURRENT_TASKS = 1  # 降低并发避免被限流

# 2. 减少爬取数量
MAX_NOTES_PER_KEYWORD = 20
MAX_COMMENTS_PER_NOTE = 10

# 3. 添加延迟
CRAWL_DELAY = 2  # 每次请求间隔2秒
```

## 扩展开发

### 添加新平台

```python
# MediaCrawler/media_platform/custom_platform.py

class CustomPlatformCrawler(BaseCrawler):
    """新平台爬虫"""

    def __init__(self):
        self.platform_name = "custom"
        self.base_url = "https://example.com"

    async def search_by_keyword(self, keyword: str, max_limit: int = 20):
        """按关键词搜索

        Args:
            keyword: 搜索关键词
            max_limit: 最大结果数

        Returns:
            搜索结果列表
        """
        # 实现搜索逻辑
        pass

    async def get_comments(self, item_id: str, max_count: int = 50):
        """获取评论

        Args:
            item_id: 内容ID
            max_count: 最大评论数

        Returns:
            评论列表
        """
        # 实现评论获取逻辑
        pass
```

## 相关文档

- **[InsightEngine](../engines/insight-engine.md)** - 数据库查询
- **[配置指南](../development/config.md)** - 配置说明
- **[核心概念](../getting-started/concepts.md)** - 数据采集概念

---

> 💡 **学习提示**：MindSpider 是 BettaFish 的数据基础，没有数据就没有分析。建议先掌握话题提取，再学习深度爬取，最后了解如何扩展新平台。
