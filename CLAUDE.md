# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BettaFish (微舆情) is a multi-agent sentiment analysis system for public opinion mining. The system uses four specialized agents that collaborate through a "forum" mechanism, with a crawler system for data collection and a sophisticated report generation engine.

## Architecture

### Core Analysis Engines

All three analysis engines share a similar **node-based architecture**:

- **QueryEngine** (`QueryEngine/`) - Web/news search agent using Tavily API
- **MediaEngine** (`MediaEngine/`) - Multimodal content analysis using Bocha API
- **InsightEngine** (`InsightEngine/`) - Private database mining with sentiment analysis

Each engine has:
- `agent.py` - Main agent class orchestrating the workflow
- `nodes/` - Processing nodes (FirstSearchNode, ReflectionNode, SummaryNode, etc.)
- `tools/` - Engine-specific tools (search APIs, database access, etc.)
- `llms/` - OpenAI-compatible LLM client wrapper
- `state/` - Agent state management

### ForumEngine (`ForumEngine/`)

The coordination mechanism that enables agent collaboration:
- `monitor.py` - Monitors agent log files (`logs/insight.log`, `logs/media.log`, `logs/query.log`) in real-time
- `llm_host.py` - Generates "host speeches" to guide agent discussions
- Writes aggregated content to `logs/forum.log`

Agents can read forum guidance via `utils/forum_reader.py`.

### ReportEngine (`ReportEngine/`)

Multi-stage report generation with template system:
- `agent.py` - Main report agent orchestrating the pipeline
- `nodes/` - Processing nodes: template selection → document layout → word budget → chapter generation → GraphRAG
- `core/` - Template parser, chapter storage, document stitcher (IR assembly)
- `ir/` - Document IR schema and validator
- `renderers/` - HTML, PDF (WeasyPrint), Markdown renderers
- `graphrag/` - Knowledge graph builder from state/forum logs
- `report_template/` - Markdown templates for different report types

### MindSpider (`MindSpider/`)

AI-powered social media crawler (git submodule: MediaCrawler):
- `BroadTopicExtraction/` - Hot topic extraction from trending news
- `DeepSentimentCrawling/` - Deep crawling of social platforms (Weibo, XHS, Douyin, etc.)
- `schema/` - Database schema and models
- Run via `python MindSpider/main.py --complete` for full pipeline

## Common Development Commands

### Running the System

```bash
# Full system (Flask + SocketIO)
python app.py

# Individual agents (Streamlit UIs)
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503
streamlit run SingleEngineApp/media_engine_streamlit_app.py --server.port 8502
streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501

# Report generation from existing logs (skip analysis phase)
python report_engine_only.py

# Re-render latest results
python regenerate_latest_html.py   # HTML
python regenerate_latest_md.py     # Markdown
python regenerate_latest_pdf.py    # PDF
```

### MindSpider Crawler

```bash
cd MindSpider
python main.py --broad-topic        # Extract topics from trending news
python main.py --deep-sentiment     # Deep crawl social platforms
python main.py --complete           # Full pipeline
python main.py --complete --test    # Test mode (limited data)
```

### Testing

```bash
# Run all tests
pytest tests/ -v

# Run specific test
pytest tests/test_monitor.py -v

# Or use the custom test runner
python tests/run_tests.py
```

### Docker

```bash
docker compose up -d    # Start all services
```

## Configuration

All configuration is managed through **environment variables** in `.env` (copy from `.env.example`):

- `config.py` - Global `Settings` class using pydantic-settings
- Each engine has its own LLM configuration (API_KEY, BASE_URL, MODEL_NAME)
- Database: PostgreSQL or MySQL via SQLAlchemy (async)

### Recommended LLM Configuration (per README)

| Engine | Recommended Model | Provider |
|--------|------------------|----------|
| Insight Agent | kimi-k2-0711-preview | moonshot.cn |
| Media Agent | gemini-2.5-pro | aihubmix.com |
| Query Agent | deepseek-chat | deepseek.com |
| Report Agent | gemini-2.5-pro | aihubmix.com |
| Forum Host | qwen-plus | aliyun.com |

All LLM calls use **OpenAI-compatible API format**.

## Key Implementation Details

### Node Pattern

Each engine's `nodes/` directory contains modular processing steps:
- `base_node.py` - Base class with logging/state hooks
- Nodes inherit and implement `run()` method
- Examples: `FirstSearchNode`, `ReflectionNode`, `SummaryNode`, `ReportFormattingNode`

### Agent Forum Communication

1. Each agent writes to its own log file (`logs/{engine}.log`)
2. `ForumEngine.monitor.LogMonitor` watches these files for `SummaryNode` outputs
3. Every 5 agent speeches, a "host" generates a moderation speech via LLM
4. Host speech written to `logs/forum.log`
5. Agents can read forum via `utils/forum_reader.py.get_latest_host_speech()`

### Document IR (Intermediate Representation)

ReportEngine generates reports through:
1. Template selection from `report_template/` (Markdown files)
2. Parse template into sections via `core/template_parser.py`
3. For each section, LLM generates JSON chapter content
4. Validate via `ir/validator.py`
5. Stitch chapters into Document IR via `core/stitcher.py`
6. Render IR to HTML/PDF/MD via `renderers/`

Chapter JSON saved to `CHAPTER_OUTPUT_DIR` (configurable) with manifest.

### Database Access

- `InsightEngine/utils/db.py` - SQLAlchemy async engine wrapper
- Read-only queries via `db.execute_query()`
- ORM models in `MindSpider/schema/models_bigdata.py`

## Important Notes

- **Git Submodule**: `MindSpider/DeepSentimentCrawling/MediaCrawler` is a submodule - use `git submodule update --init --recursive` after cloning
- **Playwright**: Requires `playwright install chromium` for crawler functionality
- **PDF Export**: Requires system dependencies (see `static/Partial README for PDF Exporting/README.md`)
- **Log Format**: ForumEngine parses loguru format; be careful changing log formats
- **GraphRAG**: Optional knowledge graph feature (disabled by default, controlled by `GRAPHRAG_ENABLED`)

## File Structure Highlights

```
BettaFish/
├── app.py                    # Flask main entry point
├── config.py                 # Global Settings (pydantic)
├── QueryEngine/              # Web search agent
├── MediaEngine/              # Multimodal analysis agent
├── InsightEngine/            # Database mining agent
├── ReportEngine/             # Report generation engine
├── ForumEngine/              # Agent coordination (forum)
├── MindSpider/               # Social media crawler
├── SentimentAnalysisModel/   # Sentiment models (BERT, GPT-2, etc.)
├── SingleEngineApp/          # Standalone Streamlit apps per agent
├── utils/                    # Shared utilities
├── logs/                     # Runtime logs
├── final_reports/            # Generated reports (HTML/PDF/MD)
└── tests/                    # Test suite (pytest)
```

## Agent Tool Naming

The three engines use similar `DeepSearchAgent` class patterns but with different tools:
- QueryEngine → `TavilyNewsAgency`
- MediaEngine → `BochaMultimodalSearch`
- InsightEngine → `MediaCrawlerDB` (database queries)

## Engine-Specific Tools

### InsightEngine Tools (`InsightEngine/tools/`)

The `MediaCrawlerDB` class provides 5 specialized database query tools for Agent usage:

1. **`search_hot_content`** - Find hot content by date range with intelligent hotness scoring
2. **`search_topic_globally`** - Global topic search across all tables
3. **`search_topic_by_date`** - Topic search within specific date range
4. **`get_comments_for_topic`** - Extract comments for a specific topic
5. **`search_topic_on_platform`** - Platform-specific search (7 platforms: xhs, dy, ks, bili, wb, tieba, zhihu)

Hotness scoring weights: `W_LIKE=1.0`, `W_COMMENT=5.0`, `W_SHARE=10.0`, `W_VIEW=0.1`, `W_DANMAKU=0.5`

### Keyword Optimizer Middleware

`InsightEngine/tools/keyword_optimizer.py` - Uses Qwen AI to optimize Agent-generated search keywords for better database queries.

### Sentiment Analysis

`InsightEngine/tools/sentiment_analyzer.py` - Integrates multiple sentiment models:
- `WeiboMultilingualSentiment` - 22-language support
- BERT/GPT-2 fine-tuned models
- Traditional ML models (SVM, XGBoost)

## Utilities

### Retry Mechanism (`utils/retry_helper.py`)

Provides `@with_retry` and `@with_graceful_retry` decorators with pre-configured retry settings:

- **`LLM_RETRY_CONFIG`** - 6 retries, 60s initial delay, 10s max delay
- **`SEARCH_API_RETRY_CONFIG`** - 5 retries, 2s initial delay, 25s max delay
- **`DB_RETRY_CONFIG`** - 5 retries, 1s initial delay, 10s max delay

Use `@with_graceful_retry` for non-critical APIs that should return defaults on failure.

### Forum Reader (`utils/forum_reader.py`)

Functions for agent forum communication:
- `get_latest_host_speech()` - Get latest HOST moderation from `logs/forum.log`
- `get_all_host_speeches()` - Get all HOST speeches
- `get_recent_agent_speeches()` - Get recent agent discussions

### Knowledge Logger (`utils/knowledge_logger.py`)

GraphRAG query logging to `logs/knowledge_query.log`:
- `init_knowledge_log()` - Initialize log file
- `append_knowledge_log()` - Append query records
- `compact_records()` - Compress large data for logging

### GitHub Issues (`utils/github_issues.py`)

Utility for creating GitHub issue URLs with pre-filled error details:
- `create_issue_url(title, body)` - Generate issue URL
- `error_with_issue_link()` - Format error with issue link

## ReportEngine Document IR

### IR Schema (`ReportEngine/ir/schema.py`)

Document Intermediate Representation uses block types:
- `heading` - Section headers
- `paragraph` - Text content
- `chart` - Plotly charts (saved as JSON)
- `image` - Image references
- `list` - Bullet/numbered lists
- `quote` - Blockquotes
- `table` - Data tables
- `code` - Code blocks
- `divider` - Horizontal rules

### IR Validator (`ReportEngine/ir/validator.py`)

`IRValidator` class validates chapter JSON structure against schema requirements.

### Chapter Storage (`ReportEngine/core/chapter_storage.py`)

Saves generated chapters to `CHAPTER_OUTPUT_DIR` with:
- `runs/{timestamp}/` - Per-run directory
- `chapter_{slug}.json` - Individual chapter files
- `manifest.json` - Run metadata

### Template Parser (`ReportEngine/core/template_parser.py`)

Parses Markdown templates in `ReportEngine/report_template/`:
- Splits by `## ` headers into sections
- Generates unique slugs for each section
- Returns `TemplateSection` objects

## Log Files

All runtime logs stored in `logs/`:
- `insight.log` - InsightEngine agent output
- `media.log` - MediaEngine agent output
- `query.log` - QueryEngine agent output
- `forum.log` - ForumEngine aggregated content
- `knowledge_query.log` - GraphRAG query logs
- `report.log` - ReportEngine output

## Configuration Inheritance

- Root `config.py` defines global `Settings` class with all environment variables
- Individual engines (e.g., `InsightEngine/utils/config.py`) can define their own `Settings`
- Engines inherit from root `.env` file - no separate env files needed
- Use `from config import settings` to access configuration

## Platform Codes (MindSpider)

| Code | Platform | Notes |
|------|----------|-------|
| xhs | 小红书 | Requires QR login |
| dy | 抖音 | Requires QR login |
| ks | 快手 | Requires QR login |
| bili | B站 | Requires QR login |
| wb | 微博 | Requires QR login |
| tieba | 贴吧 | Requires QR login |
| zhihu | 知乎 | Requires QR login |

## Development Workflow

### Adding a New Agent Tool

1. Create tool function in `{Engine}/tools/search.py` or new file
2. Expose via `{Engine}/tools/__init__.py`
3. Add tool description to Agent prompts
4. Register in agent's `search_agency` initialization

### Adding a New Report Template

1. Create Markdown file in `ReportEngine/report_template/`
2. Use `## ` for section headers (level 2)
3. Template parser auto-splits and generates slugs
4. ReportEngine auto-selects based on query analysis

### Adding a New Processing Node

1. Inherit from `base_node.BaseNode` in `{Engine}/nodes/`
2. Implement `run(state, **kwargs)` method
3. Register in agent's `_initialize_nodes()`
4. Add to agent workflow in `agent.py`

### Debugging Tips

- **Check logs**: `logs/` directory contains all engine outputs
- **Forum debug**: `logs/forum.log` shows agent coordination
- **Report debug**: Use `report_engine_only.py --verbose` for detailed output
- **Single agent mode**: Run individual Streamlit apps to isolate issues
- **Loguru format**: Most logs use loguru format with timestamp + level

### Git Workflow

- Branch naming: `feature/xxx` or `fix/xxx`
- PR target: `main` branch
- Follow [Conventional Commits](https://www.conventionalcommits.org/zh-hans/)
- MediaCrawler is a git submodule - use `git submodule update --init --recursive`
