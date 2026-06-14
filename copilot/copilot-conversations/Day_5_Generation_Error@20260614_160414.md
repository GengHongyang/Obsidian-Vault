---
epoch: 1781424254034
modelKey: "deepseek-chat|deepseek"
topic: "Day 5 Generation Error"



tags:
  - copilot-conversation
---

**user**: 生成Day 5
[Context: Notes: copilot/copilot-conversations/Day_4_Content_Generation@20260612_081959.md]
[Timestamp: 2026/06/14 16:04:14]

**ai**: 
<errorChunk>Model request failed: API key is not provided for the model: deepseek-chat|deepseek.</errorChunk>
[Timestamp: 2026/06/14 16:04:14]

**user**: 生成Day 5
[Context: Notes: copilot/copilot-conversations/Day_4_Content_Generation@20260612_081959.md, ai-agent-daily-mentor/SKILL.md]
[Timestamp: 2026/06/14 16:06:29]

**ai**: 📌 今日核心死磕

### 1. 异步并发调用大模型 API

- **它是什么**：异步并发是指使用 `asyncio.gather` 同时发送多个 API 请求，而不是等待一个请求返回后再发下一个。在 Python 中，这通过 `async/await` 语法和事件循环实现。
- **它解决 Agent 工程里的什么痛点**：Agent 经常需要同时调用多个大模型（例如同时分析简历和匹配岗位），或者对一个模型提多个独立问题。串行调用会导致总耗时等于所有请求耗时之和，而异步并发可以将总耗时降低到最慢单个请求的耗时。
- **今天为什么必须学**：昨天你封装了 `LLMClient`，但只做了单次调用。今天你将用 `asyncio.gather` 同时发送 3 个问题，并对比串行和并发的耗时差异。这是 Week 1 核心产出物 `LLM CLI` 的关键性能优化。
- **新手最容易踩什么坑**：混淆并发和并行；忘记在 `async` 函数内部使用 `await`；在 Jupyter Notebook 中重复运行 `asyncio.run()` 导致事件循环冲突。

关联笔记：[[Python 异步编程]]、[[asyncio.gather 并发]]、[[LLM API 并发调用]]

### 2. 异步函数计时装饰器

- **它是什么**：装饰器是一个高阶函数，它接受一个函数作为参数，返回一个增强后的新函数。异步函数计时装饰器专门用于统计 `async` 函数的执行耗时。
- **它解决 Agent 工程里的什么痛点**：在 Agent 开发中，你需要知道每次 API 调用花了多长时间，以便监控性能、排查慢请求、控制成本。手动在每个函数里写 `start = time.time()` 非常冗余且容易遗漏。
- **今天为什么必须学**：今天你要对比串行和并发的耗时，计时装饰器能让你用一行 `@timer` 就完成耗时统计，代码更干净。
- **新手最容易踩什么坑**：装饰器内部忘记 `await` 被装饰的异步函数；装饰器返回值类型与原始函数不一致。

关联笔记：[[装饰器]]、[[异步函数计时装饰器]]、[[LLM API 并发调用]]

---

## 🎯 6小时精细化路线

### 上午 2 小时：理解异步并发与装饰器

- **具体任务**：
  1. 阅读 `asyncio.gather` 的官方文档或一篇中文教程（15 分钟）。
  2. 理解 `async def`、`await`、`asyncio.run` 的执行流程。
  3. 理解装饰器的基本结构：外层函数接收 `func`，内层函数调用 `func` 并返回。
  4. 写一个简单的同步计时装饰器，再改成异步版本。
- **预期产出**：能解释 `asyncio.gather` 的作用，能写出一个可运行的异步计时装饰器。
- **成功检查点**：运行一个 `async def` 函数，用装饰器打印出耗时。
- **Obsidian 双链**：[[Python 异步编程]]、[[装饰器]]

### 下午 3 小时：并发调用大模型 API + 耗时对比

- **具体任务**：
  1. 在 `src/` 下创建 `concurrent_demo.py`。
  2. 导入昨天写的 `LLMClient`。
  3. 定义 3 个不同的问题（例如“请用一句话介绍 Python”“请用一句话介绍 FastAPI”“请用一句话介绍 RAG”）。
  4. 先写一个串行版本：依次调用 `chat()`，记录总耗时。
  5. 再写一个并发版本：用 `asyncio.gather` 同时调用 3 次 `chat()`，记录总耗时。
  6. 打印两次的耗时对比。
- **预期产出**：一个可运行的脚本，能清晰展示串行和并发的耗时差异。
- **成功检查点**：并发版本的总耗时明显小于串行版本（通常接近最慢单个请求的耗时）。
- **Obsidian 双链**：[[asyncio.gather 并发]]、[[LLM API 并发调用]]

### 晚上 1 小时：重构、Debug、笔记、自测

- **具体任务**：
  1. 将计时装饰器提取到 `src/timer.py` 中，方便复用。
  2. 检查代码中是否有未处理的异常（例如某个请求超时导致整个 `gather` 失败）。
  3. 完成今日验收题。
  4. 整理笔记，重点记录 `asyncio.gather` 的用法和装饰器的实现。
- **预期产出**：干净的代码、完整的笔记、通过验收题。
- **成功检查点**：验收题全部完成，计时装饰器可独立导入使用。
- **Obsidian 双链**：[[异步函数计时装饰器]]、[[GIL 全局解释器锁]]

---

## 💻 今日代码实战

**关联知识：** [[Python 异步编程]]、[[asyncio.gather 并发]]、[[装饰器]]、[[异步函数计时装饰器]]、[[LLM API 并发调用]]

```python
# src/timer.py
"""
异步函数计时装饰器。

用于统计 async 函数的执行耗时，支持打印或返回耗时信息。
"""

import asyncio
import time
from functools import wraps
from typing import Any, Callable, Coroutine


def async_timer(func: Callable[..., Coroutine]) -> Callable[..., Coroutine]:
    """
    异步函数计时装饰器。

    用法：
        @async_timer
        async def my_async_func():
            ...
    """
    @wraps(func)
    async def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.perf_counter()  # 高精度计时起点
        result = await func(*args, **kwargs)  # 等待被装饰的异步函数执行完毕
        elapsed = time.perf_counter() - start  # 计算耗时
        print(f"[{func.__name__}] 耗时: {elapsed:.3f} 秒")
        return result
    return wrapper
```

```python
# src/concurrent_demo.py
"""
串行 vs 并发调用大模型 API 的耗时对比演示。

使用昨天封装的 LLMClient 和今天的计时装饰器。
"""

import asyncio
import time

from llm_client import LLMClient
from timer import async_timer


# 实例化 LLMClient（确保 .env 中已配置 API Key 和 Base URL）
client = LLMClient()


# 定义三个不同的问题
QUESTIONS = [
    "请用一句话介绍 Python",
    "请用一句话介绍 FastAPI",
    "请用一句话介绍 RAG",
]


@async_timer
async def call_single(prompt: str) -> str:
    """单次调用大模型 API（计时装饰器会自动打印耗时）"""
    return await client.chat(user_prompt=prompt)


async def serial_calls() -> None:
    """串行调用：依次等待每个请求完成"""
    print("=" * 40)
    print("串行调用开始")
    print("=" * 40)
    start = time.perf_counter()
    for i, question in enumerate(QUESTIONS, 1):
        print(f"\n问题 {i}: {question}")
        response = await call_single(question)
        print(f"回答 {i}: {response[:100]}...")  # 只打印前 100 个字符
    total = time.perf_counter() - start
    print(f"\n串行总耗时: {total:.3f} 秒")


async def concurrent_calls() -> None:
    """并发调用：同时发送所有请求"""
    print("=" * 40)
    print("并发调用开始")
    print("=" * 40)
    start = time.perf_counter()
    # 创建 3 个协程对象（此时还没有真正执行）
    tasks = [call_single(q) for q in QUESTIONS]
    # 用 asyncio.gather 同时执行所有协程
    responses = await asyncio.gather(*tasks)
    for i, (question, response) in enumerate(zip(QUESTIONS, responses), 1):
        print(f"\n问题 {i}: {question}")
        print(f"回答 {i}: {response[:100]}...")
    total = time.perf_counter() - start
    print(f"\n并发总耗时: {total:.3f} 秒")


async def main() -> None:
    """主函数：先串行，再并发，对比耗时"""
    await serial_calls()
    print("\n" + "=" * 40)
    await concurrent_calls()


if __name__ == "__main__":
    asyncio.run(main())
```

### 运行方式

```bash
# 确保在 week1_llm_cli 目录下，虚拟环境已激活
cd week1_llm_cli
python src/concurrent_demo.py
```

### 你需要观察什么

1. 串行版本的总耗时 ≈ 3 个请求耗时之和。
2. 并发版本的总耗时 ≈ 最慢那个请求的耗时（通常远小于串行总耗时）。
3. 每个请求的单独耗时在串行和并发中基本一致（因为 API 响应时间不受客户端并发影响）。

### 可扩展任务

1. 增加 `return_exceptions=True` 参数给 `asyncio.gather`，让某个请求失败时不影响其他请求。
2. 将并发数量从 3 增加到 5 或 10，观察耗时变化。
3. 在 `LLMClient` 中增加一个 `concurrent_chat()` 方法，封装 `asyncio.gather` 的逻辑。

---

## 🎯 每日自我验收题

1. **概念解释**：请解释 `asyncio.gather` 的作用，并说明它为什么适合用于并发调用大模型 API。（提示：参考 [[asyncio.gather 并发]]）

2. **小实现或设计题**：请为 `async_timer` 装饰器增加一个可选参数 `unit: str = "s"`，支持 `"s"`（秒）和 `"ms"`（毫秒）两种单位。当 `unit="ms"` 时，打印 `耗时: 1234.567 毫秒`。

3. **代码改错或 Bug 排查**：以下代码试图并发调用大模型 API，但运行时报错。请找出错误并修复。

```python
import asyncio
from llm_client import LLMClient

client = LLMClient()

async def ask(question: str) -> str:
    return await client.chat(user_prompt=question)

questions = ["问题1", "问题2", "问题3"]
tasks = [ask(q) for q in questions]

# 错误代码
results = asyncio.gather(tasks)
print(results)
```

---

## 📥 明日同步接口

请你明天把下面这段填好发给我：

【Day 5 学习反馈】
1. 今天实际学习时长：
2. 完成了哪些代码/文件：
3. 哪个概念最清楚了：
4. 哪个概念还没懂：
5. 卡住的 Bug / 报错全文：
6. 自我验收题完成情况：
7. 明天希望：正常推进 / 放慢复习 / 加难扩展
[Timestamp: 2026/06/14 16:06:56]