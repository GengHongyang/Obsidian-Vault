---
epoch: 1781145086809
modelKey: "deepseek-chat|deepseek"
topic: "Day 3 Generation Request"



tags:
  - copilot-conversation
---

**user**: "生成 Day 3"
[Context: Notes: ai-agent-daily-mentor/SKILL.md, copilot/copilot-conversations/AI_Agent学习计划-Day1@20260610_121019.md]
[Timestamp: 2026/06/11 10:31:26]

**ai**: 好的，收到你的 Day 2 学习反馈。

**关键信息提取：**
- 实际学习时长 4 小时（持续超出预期，投入度很高）
- 完成了 `config.py` 和 `history.py`（与 Day 2 代码实战内容一致）
- 虚拟环境概念清楚，无卡住 Bug
- 验收题完成 2/3，明天希望正常推进

**自适应判断：**
- 你连续两天提前完成，且额外写了文件，说明基础扎实、学习效率高。
- 根据 `[[SKILL]]` 自适应规则：**提前完成 → 主线不跳跃，增加一个工程扩展**。
- 今天在 Day 3 主线内容（HTTP 与 API 调用）基础上，增加一个**工程扩展**：将昨天的 `data_structures_demo.py` 中的 JSON 读写功能封装成一个独立的 `HistoryManager` 类，为 Week 1 的 `LLM CLI` 项目做准备。

---

## 📌 今日核心死磕

### 1. HTTP 请求结构与 API 调用

- **它是什么**：HTTP（超文本传输协议）是客户端与服务器通信的协议。一个 HTTP 请求包含方法（GET/POST/PUT/DELETE）、URL、请求头（Headers）和请求体（Body）。API（应用程序接口）通过 HTTP 协议暴露服务，让客户端可以调用。
- **它解决 Agent 工程里的什么痛点**：AI Agent 的核心能力是调用外部工具和 API。无论是调用大模型 API（如 DeepSeek、OpenAI）、查询数据库、还是调用其他微服务，本质上都是 HTTP 请求。不理解 HTTP，就无法理解 Agent 的通信机制。
- **今天为什么必须学**：这是 Week 1 的核心产出物 `LLM CLI` 的基础。今天你将亲手用 Python 发送 HTTP 请求，调用一个公开 API，并封装一个通用的 API Client。这是 Week 3 调用大模型 API 的前置技能。
- **新手最容易踩什么坑**：混淆 GET 和 POST 的使用场景；忘记设置 `Content-Type: application/json` 请求头；不处理超时和网络异常，导致程序卡死。

### 2. `httpx` 库：同步与异步请求

- **它是什么**：`httpx` 是一个功能强大的 Python HTTP 客户端库，支持同步和异步请求，是 `requests` 库的现代替代品。
- **它解决 Agent 工程里的什么痛点**：Agent 经常需要并发调用多个 API（如同时调用多个大模型、或同时查询多个知识库）。`httpx` 的异步支持（`httpx.AsyncClient`）让并发调用变得简单高效，是构建高性能 Agent 的关键。
- **今天为什么必须学**：Week 1 的 Day 4 将专门学习异步并发，今天先用同步方式掌握 HTTP 请求的基本流程，为明天的异步学习打下基础。
- **新手最容易踩什么坑**：忘记安装 `httpx`；混淆 `httpx.Client()` 和 `httpx.AsyncClient()`；不处理 `httpx.HTTPStatusError` 和 `httpx.TimeoutException`。

---

## 🎯 2小时精细化路线

### 上午 1 小时：HTTP 基础与 `httpx` 同步请求

- **具体任务**：
  1. 安装 `httpx`：`pip install httpx`。
  2. 阅读 `httpx` 官方文档的“Quickstart”部分（5 分钟）。
  3. 在 `src/` 下创建 `http_demo.py`。
  4. 用 `httpx.Client` 发送 GET 请求到公开 API（如 `https://api.github.com`）。
  5. 打印响应状态码、响应头和响应体（JSON 格式）。
  6. 发送 POST 请求到 JSONPlaceholder 等测试 API。
- **预期产出**：一个可运行的脚本，能发送 GET 和 POST 请求，并正确解析 JSON 响应。
- **成功检查点**：运行脚本后，能正确打印出 GitHub API 返回的 JSON 数据。
- **Obsidian 双链**：`[[HTTP 请求结构]]`、`[[httpx 同步请求]]`

### 下午 1 小时：封装通用 API Client + 工程扩展

- **具体任务**：
  1. 在 `src/` 下创建 `api_client.py`。
  2. 封装一个 `APIClient` 类，包含 `get()` 和 `post()` 方法。
  3. 加入超时处理、异常捕获和日志打印。
  4. **工程扩展**：将昨天的 JSON 读写功能封装成 `HistoryManager` 类，放在 `src/history_manager.py`。
- **预期产出**：一个可复用的 `APIClient` 类和一个 `HistoryManager` 类。
- **成功检查点**：`APIClient` 能成功调用 GitHub API 并返回数据；`HistoryManager` 能保存和加载对话历史。
- **Obsidian 双链**：`[[API 客户端封装]]`、`[[超时处理]]`、`[[异常处理]]`

---

## 💻 今日代码实战

**关联知识：** `[[HTTP 请求结构]]`、`[[httpx 同步请求]]`、`[[API 客户端封装]]`、`[[超时处理]]`、`[[JSON 序列化]]`

```python
# src/api_client.py
"""
通用 HTTP API 客户端封装
用于调用外部 API，包含超时、异常处理和日志
"""

import json
import logging
from pathlib import Path
from typing import Any, Dict, Optional

import httpx

# 配置日志
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")
logger = logging.getLogger(__name__)


class APIClient:
    """
    通用 HTTP API 客户端。

    封装了 GET 和 POST 请求，自动处理超时、异常和 JSON 解析。
    """

    def __init__(self, base_url: str, timeout: float = 10.0):
        """
        初始化 API 客户端。

        Args:
            base_url: API 的基础 URL。
            timeout: 请求超时时间（秒），默认 10 秒。
        """
        self.base_url = base_url.rstrip("/")  # 去掉末尾的斜杠，方便拼接
        self.timeout = timeout
        # 创建 httpx 客户端，设置超时和默认请求头
        self.client = httpx.Client(
            timeout=httpx.Timeout(timeout),
            headers={"User-Agent": "AI-Interview/1.0"},
        )
        logger.info(f"API 客户端初始化完成，基础 URL: {self.base_url}，超时: {timeout}s")

    def get(self, endpoint: str, params: Optional[Dict[str, Any]] = None) -> Dict[str, Any]:
        """
        发送 GET 请求。

        Args:
            endpoint: API 端点路径，例如 "/users"。
            params: 查询参数，例如 {"page": 1}。

        Returns:
            解析后的 JSON 响应数据。

        Raises:
            httpx.HTTPStatusError: 如果响应状态码不是 2xx。
            httpx.TimeoutException: 如果请求超时。
            httpx.RequestError: 如果网络错误。
        """
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        logger.info(f"发送 GET 请求: {url}，参数: {params}")

        try:
            response = self.client.get(url, params=params)
            response.raise_for_status()  # 如果状态码不是 2xx，抛出异常
            data = response.json()
            logger.info(f"GET 请求成功，状态码: {response.status_code}")
            return data
        except httpx.HTTPStatusError as e:
            logger.error(f"HTTP 状态码错误: {e.response.status_code}，响应: {e.response.text}")
            raise
        except httpx.TimeoutException as e:
            logger.error(f"请求超时: {url}")
            raise
        except httpx.RequestError as e:
            logger.error(f"请求失败: {e}")
            raise

    def post(
        self,
        endpoint: str,
        json_data: Optional[Dict[str, Any]] = None,
        headers: Optional[Dict[str, str]] = None,
    ) -> Dict[str, Any]:
        """
        发送 POST 请求。

        Args:
            endpoint: API 端点路径。
            json_data: 请求体（JSON 格式）。
            headers: 额外的请求头。

        Returns:
            解析后的 JSON 响应数据。
        """
        url = f"{self.base_url}/{endpoint.lstrip('/')}"
        logger.info(f"发送 POST 请求: {url}")

        try:
            # 合并默认请求头和额外请求头
            request_headers = self.client.headers.copy()
            if headers:
                request_headers.update(headers)

            response = self.client.post(url, json=json_data, headers=request_headers)
            response.raise_for_status()
            data = response.json()
            logger.info(f"POST 请求成功，状态码: {response.status_code}")
            return data
        except httpx.HTTPStatusError as e:
            logger.error(f"HTTP 状态码错误: {e.response.status_code}，响应: {e.response.text}")
            raise
        except httpx.TimeoutException as e:
            logger.error(f"请求超时: {url}")
            raise
        except httpx.RequestError as e:
            logger.error(f"请求失败: {e}")
            raise

    def close(self) -> None:
        """关闭客户端，释放资源。"""
        self.client.close()
        logger.info("API 客户端已关闭")


# --- 工程扩展：HistoryManager 类 ---

class HistoryManager:
    """
    对话历史管理器。

    负责对话历史的保存和加载，使用 JSON 文件持久化。
    """

    def __init__(self, filepath: str = "data/chat_history.json"):
        """
        初始化历史管理器。

        Args:
            filepath: 历史记录文件的路径。
        """
        self.filepath = Path(filepath)
        # 确保父目录存在
        self.filepath.parent.mkdir(parents=True, exist_ok=True)
        logger.info(f"历史管理器初始化完成，文件路径: {self.filepath}")

    def save(self, history: list[dict[str, str]]) -> None:
        """
        保存对话历史到 JSON 文件。

        Args:
            history: 对话历史列表，每个元素是 {"role": str, "content": str}。
        """
        try:
            with open(self.filepath, "w", encoding="utf-8") as f:
                json.dump(history, f, ensure_ascii=False, indent=2)
            logger.info(f"对话历史已保存，共 {len(history)} 条记录")
        except IOError as e:
            logger.error(f"保存历史失败: {e}")
            raise

    def load(self) -> list[dict[str, str]]:
        """
        从 JSON 文件加载对话历史。

        Returns:
            对话历史列表，如果文件不存在则返回空列表。
        """
        if not self.filepath.exists():
            logger.info(f"历史文件不存在，返回空历史")
            return []

        try:
            with open(self.filepath, "r", encoding="utf-8") as f:
                history = json.load(f)
            logger.info(f"对话历史已加载，共 {len(history)} 条记录")
            return history
        except json.JSONDecodeError as e:
            logger.error(f"JSON 解析失败: {e}")
            return []
        except IOError as e:
            logger.error(f"加载历史失败: {e}")
            return []

    def add_message(self, role: str, content: str) -> None:
        """
        添加一条消息并保存。

        Args:
            role: 角色，如 "user" 或 "assistant"。
            content: 消息内容。
        """
        history = self.load()
        history.append({"role": role, "content": content})
        self.save(history)
        logger.info(f"已添加 {role} 消息: {content[:30]}...")


if __name__ == "__main__":
    # --- 演示 APIClient ---
    print("=" * 60)
    print("1. 演示 APIClient - 调用 GitHub API")
    print("=" * 60)

    # 创建客户端，调用 GitHub API
    client = APIClient(base_url="https://api.github.com", timeout=5.0)

    try:
        # GET 请求：获取 Python 仓库信息
        repos = client.get(endpoint="/search/repositories", params={"q": "python", "per_page": 3})
        print(f"\n找到 {repos['total_count']} 个 Python 仓库，显示前 3 个：")
        for repo in repos["items"]:
            print(f"  - {repo['full_name']}: {repo['description']}")

        # POST 请求：创建一个 GitHub Gist（需要 token，这里只演示结构）
        # 实际运行时会被 401 拒绝，但能演示 POST 流程
        print("\n尝试 POST 请求（会失败，因为未认证）：")
        try:
            result = client.post(
                endpoint="/gists",
                json_data={"description": "test", "public": False, "files": {"test.txt": {"content": "hello"}}},
            )
            print(f"POST 成功: {result}")
        except httpx.HTTPStatusError as e:
            print(f"POST 失败（预期行为）: {e.response.status_code} - {e.response.json()['message']}")

    finally:
        client.close()

    # --- 演示 HistoryManager ---
    print("\n" + "=" * 60)
    print("2. 演示 HistoryManager - 对话历史管理")
    print("=" * 60)

    history_mgr = HistoryManager("data/chat_history.json")

    # 添加几条消息
    history_mgr.add_message("user", "Python 中 GIL 是什么？")
    history_mgr.add_message("assistant", "GIL 是全局解释器锁，它确保同一时刻只有一个线程执行 Python 字节码。")
    history_mgr.add_message("user", "怎么绕过 GIL？")

    # 加载并打印所有历史
    all_history = history_mgr.load()
    print(f"\n当前对话历史（共 {len(all_history)} 条）：")
    for msg in all_history:
        print(f"  [{msg['role']}]: {msg['content'][:40]}...")
```

- **运行方式**：
  1. 确保虚拟环境已激活。
  2. 安装依赖：`pip install httpx`。
  3. 运行：`python src/api_client.py`。

- **你需要观察什么**：
  - GET 请求是否成功返回 GitHub 仓库列表。
  - POST 请求失败时，`httpx.HTTPStatusError` 是否被正确捕获，并打印了友好的错误信息。
  - `HistoryManager` 是否成功保存和加载了对话历史。
  - 日志输出是否清晰，包含了时间戳、日志级别和具体信息。

- **可扩展任务**：
  - 为 `APIClient` 增加 `put()` 和 `delete()` 方法。
  - 为 `HistoryManager` 增加 `clear()` 方法，清空所有历史。
  - 为 `APIClient` 增加重试机制：当请求失败时，自动重试 3 次。

---

## 🎯 每日自我验收题

1. **概念解释**：请解释 HTTP GET 和 POST 请求的区别，并分别给出一个在 AI Agent 开发中的使用场景。（提示：参考 `[[HTTP 请求结构]]`）

2. **小实现或设计题**：请为 `APIClient` 类增加一个 `get_with_retry()` 方法，当请求失败时自动重试最多 3 次，每次重试间隔 1 秒。要求使用 `time.sleep()` 实现间隔。

3. **代码改错或 Bug 排查**：以下代码试图调用一个 API 并打印结果，但运行时报错。请找出所有错误并修正。
   ```python
   import httpx

   def fetch_user(user_id: int) -> dict:
       client = httpx.Client()
       response = client.get(f"https://api.example.com/users/{user_id}")
       return response.json()

   if __name__ == "__main__":
       user = fetch_user(123)
       print(f"User name: {user['name']}")
   ```
   假设 API 返回的 JSON 格式为：`{"id": 123, "name": "Alice", "email": "alice@example.com"}`

---

## 📥 明日同步接口

请你明天把下面这段填好发给我：

【Day 3 学习反馈】
1. 今天实际学习时长：
2. 完成了哪些代码/文件：
3. 哪个概念最清楚了：
4. 哪个概念还没懂：
5. 卡住的 Bug / 报错全文：
6. 自我验收题完成情况：
7. 明天希望：正常推进 / 放慢复习 / 加难扩展
[Timestamp: 2026/06/11 10:32:03]