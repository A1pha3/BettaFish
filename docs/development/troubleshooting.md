# 故障排查指南

本指南提供 BettaFish 系统常见问题的排查和解决方案。

## 学习目标

完成本章学习后，你将能够：

- [ ] 掌握**系统化的故障排查方法论**
- [ ] 熟练使用**诊断工具**定位问题根源
- [ ] 解决**安装、启动、数据库、LLM、爬虫、报告**六大类常见问题
- [ ] 编写**自动化诊断脚本**
- [ ] 有效收集和提交**Issue 信息**

## 问题分类

| 类别 | 常见问题 |
|------|----------|
| **安装问题** | 依赖安装失败、环境配置错误 |
| **启动问题** | 服务无法启动、端口占用 |
| **数据库问题** | 连接失败、查询错误 |
| **LLM 问题** | API 调用失败、响应异常 |
| **爬虫问题** | 登录失败、数据为空 |
| **报告问题** | 生成失败、格式错误 |

## 安装问题

### 问题：pip 安装依赖失败

**症状**：
```
ERROR: Could not build wheels for xxx
```

**解决方案**：

1. 更新 pip 和 setuptools：
```bash
pip install --upgrade pip setuptools wheel
```

2. 使用国内镜像源：
```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

3. 某些包需要编译依赖：
```bash
# Ubuntu/Debian
sudo apt install build-essential python3-dev

# CentOS/RHEL
sudo yum groupinstall "Development Tools"
sudo yum install python3-devel
```

### 问题：Playwright 安装失败

**症状**：
```
playwright install 失败
```

**解决方案**：

1. 手动安装浏览器：
```bash
playwright install chromium --with-deps
```

2. 如果网络问题，使用代理：
```bash
export PLAYWRIGHT_DOWNLOAD_HOST=https://npmmirror.com/mirrors/playwright/
playwright install chromium
```

## 启动问题

### 问题：端口被占用

**症状**：
```
Address already in use: 5000
```

**解决方案**：

1. 查找占用进程：
```bash
# Linux/macOS
lsof -i :5000
kill -9 <PID>

# Windows
netstat -ano | findstr :5000
taskkill /PID <PID> /F
```

2. 修改端口：
```bash
# .env 文件
PORT=5001
```

### 问题：模块导入失败

**症状**：
```
ModuleNotFoundError: No module named 'xxx'
```

**解决方案**：

1. 确认虚拟环境已激活：
```bash
which python
# 应该指向虚拟环境中的 python
```

2. 重新安装依赖：
```bash
pip install -r requirements.txt
```

3. 检查 Python 路径：
```bash
python -c "import sys; print(sys.path)"
```

## 数据库问题

### 问题：数据库连接失败

**症状**：
```
psycopg2.OperationalError: could not connect to server
```

**解决方案**：

1. 检查数据库服务状态：
```bash
# PostgreSQL
sudo systemctl status postgresql

# 启动数据库
sudo systemctl start postgresql
```

2. 验证连接参数：
```bash
psql -h localhost -U bettafish -d bettafish
```

3. 检查防火墙：
```bash
sudo ufw allow 5432  # PostgreSQL
sudo ufw allow 3306  # MySQL
```

### 问题：表不存在

**症状**：
```
sqlalchemy.exc.ProgrammingError: relation "xxx" does not exist
```

**解决方案**：

1. 初始化数据库：
```bash
cd MindSpider
python main.py --setup
```

2. 手动创建表：
```bash
python -c "
from MindSpider.schema.init_database import init_database
init_database()
"
```

## LLM 问题

### 问题：API 调用失败

**症状**：
```
openai.APIError: The server had an error while processing your request
```

**解决方案**：

1. 验证 API Key：
```bash
curl https://api.moonshot.cn/v1/models \
  -H "Authorization: Bearer $INSIGHT_ENGINE_API_KEY"
```

2. 检查 API 额度：
- 登录提供商控制台
- 查看剩余额度

3. 尝试更换模型：
```bash
# .env
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview
```

### 问题：响应超时

**症状**：
```
Timeout error
```

**解决方案**：

1. 增加超时时间：
```python
# config.py
SEARCH_TIMEOUT = 240  # 增加到 4 分钟
```

2. 使用重试机制：
```python
from utils.retry_helper import with_retry

@with_retry(**LLM_RETRY_CONFIG)
def llm_call():
    ...
```

3. 减少请求数据量：
```bash
MAX_SEARCH_RESULTS_FOR_LLM=50
```

## 爬虫问题

### 问题：登录失败

**症状**：
```
Login failed for platform xxx
```

**解决方案**：

1. 更新 Cookies：
- 手动登录平台
- 导出 Cookies
- 更新配置文件

2. 检查账号状态：
- 确认账号未被封禁
- 验证是否需要验证码

3. 使用代理：
```bash
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080
```

### 问题：爬取数据为空

**症状**：
```
No data found for keyword
```

**解决方案**：

1. 验证关键词：
```bash
# 使用更通用的关键词
python main.py --deep-sentiment --keywords "测试"
```

2. 检查平台状态：
- 手动访问平台
- 确认内容存在

3. 调整爬取参数：
```python
# config.py
max_notes_per_keyword = 100  # 增加数量
```

## 报告问题

### 问题：报告生成失败

**症状**：
```
Chapter generation failed
```

**解决方案**：

1. 检查输入文件：
```bash
ls -la *_engine_streamlit_reports/
```

2. 验证 LLM 配置：
```bash
# .env
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro
```

3. 降低单次生成量：
```bash
# 减少章节字数要求
python report_engine_only.py --max-chapter-words 1000
```

### 问题：PDF 生成失败

**症状**：
```
WeasyPrint error
```

**解决方案**：

1. 安装系统依赖：
```bash
# Ubuntu/Debian
sudo apt install python3-dev libxml2-dev libxslt-dev libpango1.0-dev libcairo2-dev

# macOS
brew install pango libxml2 libxslt
```

2. 安装中文字体：
```bash
sudo apt install fonts-wqy-microhei fonts-wqy-zenhei
```

3. 使用 HTML 作为备选：
```bash
python report_engine_only.py --skip-pdf
```

## 性能问题

### 问题：响应速度慢

**症状**：
```
分析时间过长
```

**解决方案**：

1. 减少搜索范围：
```bash
DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=20
```

2. 减少反思次数：
```bash
MAX_REFLECTIONS=1
```

3. 使用更快的模型：
```bash
QUERY_ENGINE_MODEL_NAME=deepseek-chat
```

### 问题：内存占用高

**症状**：
```
Memory usage too high
```

**解决方案**：

1. 减少 Worker 数量：
```bash
gunicorn -w 2 app:app
```

2. 限制数据加载：
```bash
MAX_SEARCH_RESULTS_FOR_LLM=50
```

3. 定期清理日志：
```bash
find logs/ -name "*.log" -mtime +7 -delete
```

## 日志分析

### 查看日志

```bash
# 实时查看
tail -f logs/report.log

# 查看错误
grep ERROR logs/*.log

# 查看最近日志
ls -lt logs/*.log | head -5
```

### 日志级别

```python
# 调整日志级别
import logging
logging.getLogger().setLevel(logging.DEBUG)
```

## 获取帮助

### 收集诊断信息

```bash
# 系统信息
python --version
pip list | grep -E "flask|loguru|pydantic"

# 配置检查
python -c "from config import settings; print(settings.dict())"

# 数据库检查
python -c "from InsightEngine.utils.db import test_connection; test_connection()"
```

### 提交 Issue

提交 Issue 时请包含：

1. 系统信息（OS、Python 版本）
2. 错误信息（完整堆栈跟踪）
3. 配置信息（隐藏敏感信息）
4. 复现步骤

```bash
# 生成诊断报告
python -c "
import sys
import platform
import traceback
from config import settings

print('=== System Info ===')
print(f'OS: {platform.system()}')
print(f'Python: {sys.version}')
print(f'Config: {settings}')
"
```

### 社区支持

- **GitHub Issues**: https://github.com/666ghj/BettaFish/issues
- **邮箱**: hangjiang@bupt.edu.cn
- **QQ 群**: 见项目主页二维码

## 故障排查方法论

### 系统化排查流程

```
┌─────────────────────────────────────────────────────────────┐
│                    故障排查系统化流程                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐                                          │
│  │  1. 问题定义  │  ← 明确问题症状、影响范围、复现步骤          │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  2. 信息收集  │  ← 日志、配置、系统状态、错误信息           │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  3. 假设形成  │  ← 基于经验和模式，推测可能原因             │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  4. 假设验证  │  ← 设计最小成本验证方案                     │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  5. 根因分析  │  ← 确认根本原因                           │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  6. 解决方案  │  ← 实施修复并验证                          │
│  └──────┬───────┘                                          │
│         │                                                  │
│         ▼                                                  │
│  ┌──────────────┐                                          │
│  │  7. 预防措施  │  ← 记录教训，防止复发                      │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 快速诊断决策树

```
问题表现
    │
    ├─→ 无法启动？
    │   ├─→ 端口占用 → lsof -i :5000
    │   ├─→ 模块缺失 → pip install -r requirements.txt
    │   └─→ 配置错误 → 检查 .env 文件
    │
    ├─→ 数据库错误？
    │   ├─→ 连接失败 → 检查数据库服务状态
    │   ├─→ 表不存在 → 运行初始化脚本
    │   └─→ 查询错误 → 检查 SQL 语法
    │
    ├─→ LLM 调用失败？
    │   ├─→ API Key 无效 → 验证 Key 有效性
    │   ├─→ 额度不足 → 充值或更换模型
    │   └─→ 超时 → 增加超时时间或减少数据量
    │
    ├─→ 爬虫无数据？
    │   ├─→ 登录失败 → 更新 Cookies
    │   ├─→ 关键词无效 → 使用更通用的词
    │   └─→ 平台限制 → 检查账号状态
    │
    └─→ 报告生成失败？
        ├─→ 输入文件缺失 → 检查引擎输出
        ├─→ 模板错误 → 验证模板格式
        └─→ PDF 错误 → 安装系统依赖或使用 HTML
```

## 实践练习

### 练习 1：诊断脚本编写 ⭐

**任务**：编写一个自动化诊断脚本，检查系统健康状态

```python
# diagnostic.py
import sys
import subprocess
from pathlib import Path

def check_python_version():
    """检查 Python 版本"""
    version = sys.version_info
    if version.major == 3 and version.minor >= 9:
        print("✓ Python 版本:", f"{version.major}.{version.minor}.{version.micro}")
        return True
    else:
        print("✗ Python 版本过低，需要 3.9+")
        return False

def check_dependencies():
    """检查核心依赖"""
    required = ["flask", "loguru", "pydantic", "sqlalchemy"]
    missing = []
    for pkg in required:
        try:
            __import__(pkg)
            print(f"✓ {pkg} 已安装")
        except ImportError:
            print(f"✗ {pkg} 未安装")
            missing.append(pkg)
    return len(missing) == 0

def check_database():
    """检查数据库连接"""
    try:
        from InsightEngine.utils.db import test_connection
        test_connection()
        print("✓ 数据库连接正常")
        return True
    except Exception as e:
        print(f"✗ 数据库连接失败: {e}")
        return False

def check_env_file():
    """检查环境配置"""
    env_path = Path(".env")
    if not env_path.exists():
        print("✗ .env 文件不存在")
        return False

    # 检查必需配置
    with open(env_path) as f:
        content = f.read()
        required_vars = ["DB_HOST", "DB_NAME", "QUERY_ENGINE_API_KEY"]
        missing = [var for var in required_vars if f"{var}=" not in content]

        if missing:
            print(f"✗ 缺少配置项: {', '.join(missing)}")
            return False
        else:
            print("✓ 环境配置完整")
            return True

def check_log_directory():
    """检查日志目录"""
    log_dir = Path("logs")
    if not log_dir.exists():
        print("✗ logs 目录不存在")
        return False
    print("✓ logs 目录存在")
    return True

def main():
    """运行所有诊断"""
    print("=== BettaFish 系统诊断 ===\n")

    checks = [
        ("Python 版本", check_python_version),
        ("依赖安装", check_dependencies),
        ("环境配置", check_env_file),
        ("数据库连接", check_database),
        ("日志目录", check_log_directory),
    ]

    results = []
    for name, check_func in checks:
        print(f"\n检查 {name}:")
        try:
            results.append(check_func())
        except Exception as e:
            print(f"✗ 检查出错: {e}")
            results.append(False)

    print("\n" + "="*50)
    passed = sum(results)
    total = len(results)
    print(f"诊断结果: {passed}/{total} 项通过")

    if passed == total:
        print("✓ 系统状态良好")
        return 0
    else:
        print("✗ 发现问题，请根据上述提示修复")
        return 1

if __name__ == "__main__":
    sys.exit(main())
```

**练习任务**：
1. 运行上述诊断脚本
2. 根据诊断结果修复发现的问题
3. 扩展脚本添加更多检查项（如 LLM API 连接）

---

### 练习 2：日志分析工具 ⭐⭐

**任务**：编写日志分析工具，快速定位错误

```python
# log_analyzer.py
import re
from pathlib import Path
from collections import Counter
from datetime import datetime

def parse_loguru_line(line):
    """解析 loguru 格式日志"""
    # 格式: 2024-01-20 12:00:00 | LEVEL | message
    pattern = r'(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \|\s*(\w+) \|\s*(.+)'
    match = re.match(pattern, line)
    if match:
        return {
            "timestamp": match.group(1),
            "level": match.group(2),
            "message": match.group(3).strip()
        }
    return None

def analyze_errors(log_files, limit=50):
    """分析错误日志"""
    print("=== 错误日志分析 ===\n")

    all_errors = []

    for log_file in log_files:
        path = Path(log_file)
        if not path.exists():
            continue

        with open(path, encoding='utf-8') as f:
            for line in f:
                parsed = parse_loguru_line(line)
                if parsed and parsed["level"] in ["ERROR", "CRITICAL"]:
                    all_errors.append({
                        "file": path.name,
                        **parsed
                    })

    if not all_errors:
        print("✓ 未发现错误日志")
        return

    # 显示最近的错误
    print(f"发现 {len(all_errors)} 条错误\n")

    for error in all_errors[-limit:]:
        print(f"[{error['file']}] {error['timestamp']}")
        print(f"  {error['message']}")
        print()

    # 错误类型统计
    error_types = Counter()
    for error in all_errors:
        # 提取错误类型（通常是第一行）
        first_line = error['message'].split('\n')[0]
        error_types[first_line[:80]] += 1

    print("\n=== 错误类型统计 ===")
    for error_msg, count in error_types.most_common(10):
        print(f"{count}x: {error_msg}")

def analyze_performance(log_files):
    """分析性能问题（慢操作）"""
    print("\n=== 性能分析 ===")

    slow_operations = []

    for log_file in log_files:
        path = Path(log_file)
        if not path.exists():
            continue

        with open(path, encoding='utf-8') as f:
            content = f.read()
            # 查找耗时记录
            pattern = r'耗时[：:]\s*(\d+\.?\d*)\s*秒'
            matches = re.findall(pattern, content)
            for time_str in matches:
                time_val = float(time_str)
                if time_val > 5:  # 超过5秒的操作
                    slow_operations.append((path.name, time_val))

    if slow_operations:
        print(f"\n发现 {len(slow_operations)} 个慢操作（>5秒）:")
        for file_name, time_val in sorted(slow_operations, key=lambda x: -x[1])[:10]:
            print(f"  [{file_name}] {time_val:.1f}秒")
    else:
        print("✓ 未发现明显性能问题")

def main():
    """主函数"""
    log_dir = Path("logs")
    if not log_dir.exists():
        print("错误: logs 目录不存在")
        return

    log_files = list(log_dir.glob("*.log"))

    if not log_files:
        print("未找到日志文件")
        return

    print(f"分析 {len(log_files)} 个日志文件\n")

    # 错误分析
    analyze_errors(log_files)

    # 性能分析
    analyze_performance(log_files)

if __name__ == "__main__":
    main()
```

**练习任务**：
1. 运行日志分析工具
2. 根据分析结果识别最常见的问题
3. 添加更多分析维度（如警告统计、按时间段分析）

---

### 练习 3：综合问题排查 ⭐⭐⭐

**场景**：系统报告"分析完成但无结果"，请完成以下排查流程

<details>
<summary>查看排查步骤参考答案</summary>

**完整排查流程**：

```bash
# 步骤 1: 确认问题症状
echo "=== 步骤 1: 问题定义 ==="
echo "症状: 分析任务完成但最终报告为空"

# 步骤 2: 检查各引擎输出
echo -e "\n=== 步骤 2: 检查引擎输出 ==="

for engine in query media insight; do
    log_file="logs/${engine}.log"
    if [ -f "$log_file" ]; then
        echo "--- $engine engine ---"
        # 检查是否有 ERROR
        error_count=$(grep -c "ERROR" "$log_file" 2>/dev/null || echo "0")
        echo "错误数量: $error_count"
        # 检查最后输出
        echo "最后输出:"
        tail -5 "$log_file"
    fi
done

# 步骤 3: 验证数据库数据
echo -e "\n=== 步骤 3: 验证数据库数据 ==="
python -c "
from InsightEngine.utils.db import execute_query
for table in ['xhs_note', 'dy_video', 'wb_weibo']:
    try:
        result = execute_query(f'SELECT COUNT(*) FROM {table} LIMIT 1')
        print(f'{table}: {result[0][0]} 条')
    except:
        print(f'{table}: 查询失败')
"

# 步骤 4: 检查报告引擎
echo -e "\n=== 步骤 4: 检查报告引擎 ==="
if [ -f "logs/report.log" ]; then
    echo "报告引擎日志:"
    grep -E "(ERROR|WARNING|失败)" logs/report.log | tail -10
fi

# 步骤 5: 根据发现制定解决方案
echo -e "\n=== 步骤 5: 解决建议 ==="
echo "根据上述检查结果:"
echo "- 如果数据库无数据 → 运行 MindSpider 爬虫"
echo "- 如果引擎有错误 → 查看对应错误日志并修复"
echo "- 如果报告生成失败 → 检查 REPORT_ENGINE 配置"
```

**常见原因与解决方案**：

| 问题现象 | 可能原因 | 解决方案 |
|---------|---------|---------|
| 数据库无数据 | 未运行爬虫 | 执行 `python MindSpider/main.py --complete` |
| InsightEngine 报错 | 数据库连接失败 | 检查 DB_* 配置 |
| QueryEngine 无结果 | API 额度不足 | 更换 API Key 或模型 |
| 报告为空 | 模板解析失败 | 验证模板格式，检查 `report_template/` |

</details>

**练习任务**：
1. 创建一个可执行的排查脚本
2. 针对每种可能的原因设计验证命令
3. 添加自动修复建议功能

---

### 练习 4：性能问题定位 ⭐⭐

**任务**：定位和优化响应慢的问题

```python
# performance_profiler.py
import time
import functools
from pathlib import Path

def profile_function(func):
    """函数性能分析装饰器"""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"[PROFILE] {func.__name__} 耗时: {elapsed:.2f}秒")
        return result
    return wrapper

def analyze_query_performance():
    """分析查询性能"""
    from InsightEngine.utils.db import execute_query
    import time

    queries = [
        "SELECT COUNT(*) FROM xhs_note LIMIT 1",
        "SELECT COUNT(*) FROM dy_video LIMIT 1",
        "SELECT COUNT(*) FROM wb_weibo LIMIT 1",
    ]

    print("=== 数据库查询性能 ===")
    for query in queries:
        start = time.time()
        try:
            execute_query(query)
            elapsed = time.time() - start
            status = "✓" if elapsed < 1 else "⚠"
            print(f"{status} {query[:40]}... ({elapsed:.2f}s)")
        except Exception as e:
            print(f"✗ {query[:40]}... 失败: {e}")

def optimize_suggestions():
    """优化建议"""
    print("\n=== 优化建议 ===")

    suggestions = [
        ("减少反思次数", "MAX_REFLECTIONS=1"),
        ("限制搜索结果", "DEFAULT_SEARCH_TOPIC_GLOBALLY_LIMIT_PER_TABLE=20"),
        ("使用更快的模型", "QUERY_ENGINE_MODEL_NAME=deepseek-chat"),
        ("减少数据加载", "MAX_SEARCH_RESULTS_FOR_LLM=50"),
        ("降低并发", "MAX_CONCURRENT_TASKS=1"),
    ]

    for i, (desc, config) in enumerate(suggestions, 1):
        print(f"{i}. {desc}: {config}")

if __name__ == "__main__":
    analyze_query_performance()
    optimize_suggestions()
```

**练习任务**：
1. 使用上述脚本分析系统性能
2. 根据建议优化配置
3. 对比优化前后的性能差异

---

### 练习 5：自动化诊断报告 ⭐⭐⭐

**任务**：创建完整的诊断报告生成器

```python
# generate_diagnostic_report.py
import sys
import json
from pathlib import Path
from datetime import datetime

class DiagnosticReporter:
    """诊断报告生成器"""

    def __init__(self):
        self.results = {}

    def collect_system_info(self):
        """收集系统信息"""
        import platform
        import subprocess

        self.results["system"] = {
            "os": platform.system(),
            "python_version": platform.python_version(),
            "timestamp": datetime.now().isoformat()
        }

    def collect_config_info(self):
        """收集配置信息（脱敏）"""
        try:
            from config import settings
            config_dict = settings.dict()

            # 脱敏处理
            safe_config = {}
            for key, value in config_dict.items():
                if "API_KEY" in key or "PASSWORD" in key or "SECRET" in key:
                    safe_config[key] = "***HIDDEN***" if value else None
                else:
                    safe_config[key] = value

            self.results["config"] = safe_config
        except Exception as e:
            self.results["config"] = {"error": str(e)}

    def check_database(self):
        """检查数据库"""
        try:
            from InsightEngine.utils.db import execute_query, test_connection
            test_connection()

            # 获取数据统计
            tables = ['xhs_note', 'dy_video', 'wb_weibo', 'bili_video']
            stats = {}
            for table in tables:
                try:
                    result = execute_query(f"SELECT COUNT(*) FROM {table}")
                    stats[table] = result[0][0] if result else 0
                except:
                    stats[table] = "N/A"

            self.results["database"] = {
                "status": "connected",
                "tables": stats
            }
        except Exception as e:
            self.results["database"] = {
                "status": "error",
                "error": str(e)
            }

    def check_recent_errors(self):
        """检查最近的错误"""
        log_dir = Path("logs")
        if not log_dir.exists():
            self.results["recent_errors"] = "no_logs"
            return

        recent_errors = []
        for log_file in log_dir.glob("*.log"):
            try:
                with open(log_file) as f:
                    lines = f.readlines()
                    for line in lines[-100:]:  # 只看最近100行
                        if "ERROR" in line or "CRITICAL" in line:
                            recent_errors.append({
                                "file": log_file.name,
                                "line": line.strip()[:200]  # 限制长度
                            })
            except:
                pass

        self.results["recent_errors"] = recent_errors[:20]  # 最多20条

    def generate_report(self, output_file="diagnostic_report.json"):
        """生成报告"""
        print("收集诊断信息...")
        self.collect_system_info()
        self.collect_config_info()
        self.check_database()
        self.check_recent_errors()

        # 保存报告
        with open(output_file, 'w', encoding='utf-8') as f:
            json.dump(self.results, f, ensure_ascii=False, indent=2)

        print(f"\n诊断报告已生成: {output_file}")

        # 打印摘要
        print("\n=== 诊断摘要 ===")
        print(f"操作系统: {self.results.get('system', {}).get('os')}")
        print(f"Python: {self.results.get('system', {}).get('python_version')}")
        print(f"数据库: {self.results.get('database', {}).get('status')}")
        print(f"最近错误: {len(self.results.get('recent_errors', []))} 条")

        return output_file

if __name__ == "__main__":
    reporter = DiagnosticReporter()
    reporter.generate_report()
```

**练习任务**：
1. 运行诊断报告生成器
2. 分析生成的 JSON 报告
3. 将报告提交给 Issue 或技术支持

---

## 自检清单

完成故障排查学习后，请自检以下能力：

### 基础技能
- [ ] 能够识别六大类常见问题的症状
- [ ] 知道每类问题的基本排查步骤
- [ ] 能够使用日志定位错误

### 进阶技能
- [ ] 能够编写诊断脚本
- [ ] 能够分析性能瓶颈
- [ ] 能够生成完整的诊断报告

### 专家技能
- [ ] 能够快速定位根本原因
- [ ] 能够设计预防性措施
- [ ] 能够帮助他人排查问题

## 相关文档

- **[配置指南](config.md)** - 配置说明
- **[部署指南](deployment.md)** - 部署相关
- **[快速开始](../getting-started/quickstart.md)** - 快速上手
