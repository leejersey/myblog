---
title: 给项目增加持久化记忆 + 联网搜索 Agent
date: 2025-12-15 14:44:26
tags: python
---

这是一个非常棒的进阶需求！这意味着我们不再只是写一个“聊天机器人”，而是在构建一个真正的 **Agent（智能体）**。

* **Redis**: 实现了生产级别的**持久化记忆**（Server 重启后记忆还在）。
* **Tools (Web Search)**: 赋予了 AI **获取实时信息**的能力（突破模型训练数据的截止日期限制）。

###🛠️ 准备工作####1. 安装新增依赖你需要安装 Redis 客户端、DuckDuckGo 搜索工具（它是免费的，不需要 API Key，适合演示）。

```bash
pip install redis duckduckgo-search

```

####2. 启动 Redis作为一个开发者，我假设你电脑上有 Docker。如果没有，请先安装 Redis。
最快启动方式：

```bash
docker run -d -p 6379:6379 --name my-redis redis

```

---

###💻 最终版代码：持久化记忆 + 联网搜索 Agent这个 `main.py` 将是一个功能完备的后端服务。

```python
import os
from dotenv import load_dotenv
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field

# --- LangChain 核心组件 ---
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_core.runnables.history import RunnableWithMessageHistory

# --- 新增：工具与代理 ---
from langchain_community.tools import DuckDuckGoSearchRun
from langchain.agents import AgentExecutor, create_tool_calling_agent

# --- 新增：Redis 历史记录 ---
from langchain_community.chat_message_histories import RedisChatMessageHistory

load_dotenv()

app = FastAPI(title="Super AI Agent")

# 1. 定义工具集 (Tools)
# 类似于给 AI 安装 "插件"。这里我们使用 DuckDuckGo 搜索。
search = DuckDuckGoSearchRun()
tools = [search]

# 2. 初始化 LLM
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0) # 设为 0 让使用工具更精准

# 3. 定义 Agent 的 Prompt
# 这里的 placeholder 很有讲究：
# {history}: 放入聊天记录
# {agent_scratchpad}: 这里的 "scratchpad" (草稿纸) 是 Agent 的思考过程。
# 当它决定搜索时，它会把 "我要搜索xxx" 的中间步骤写在这里。
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个全能助手。如果用户问的问题你不知道，请使用搜索工具查找最新信息。"),
    MessagesPlaceholder(variable_name="history"),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

# 4. 创建 Agent (核心逻辑)
# create_tool_calling_agent 是 LangChain 专门为支持 Function Calling 的模型(如 GPT)设计的
agent = create_tool_calling_agent(llm, tools, prompt)

# 5. 创建 AgentExecutor (运行时)
# Agent 只是逻辑，Executor 才是真正去循环执行 "思考 -> 调用工具 -> 拿结果 -> 再思考" 的人
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 6. Redis 记忆管理
# 替换掉了之前的 InMemory 字典
def get_session_history(session_id: str):
    return RedisChatMessageHistory(
        session_id=session_id,
        url="redis://localhost:6379/0",  # Redis 连接串
        ttl=3600 # 记忆有效期 1 小时 (可选)
    )

# 7. 包装 Agent，注入记忆能力
agent_with_chat_history = RunnableWithMessageHistory(
    agent_executor,
    get_session_history,
    input_messages_key="input",
    history_messages_key="history"
)

# --- API 定义 ---
class AgentRequest(BaseModel):
    session_id: str = Field(..., description="用户会话ID")
    query: str = Field(..., description="用户的问题")

@app.post("/agent/chat")
async def chat_endpoint(req: AgentRequest):
    try:
        # invoke 调用
        result = await agent_with_chat_history.ainvoke(
            {"input": req.query},
            config={"configurable": {"session_id": req.session_id}}
        )
        # AgentExecutor 的返回结果通常在 "output" 字段里
        return {"response": result["output"]}
    
    except Exception as e:
        print(f"Error: {e}")
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)

```

---

###🧪 验证“超级能力”启动服务 `python main.py`，然后在 Swagger UI 或 Postman 测试。

####场景 1：测试联网能力 (Tool Use)询问一个 GPT-3.5 训练数据截止之后的事情。
**Request:**

```json
{
  "session_id": "dev_001",
  "query": "现在 OpenAI 的最新模型是 GPT-4o 吗？请搜索确认。"
}

```

**后台日志 (Verbose=True)**：
你会看到精彩的日志输出：

1. **Thought**: 用户问了最新模型 -> 我需要搜索。
2. **Action**: `duckduckgo_search` with input `"OpenAI latest model GPT-4o"`.
3. **Observation**: (DuckDuckGo 返回的搜索结果文本)。
4. **Final Answer**: 基于搜索结果生成回答。

**Response:**

> "是的，根据搜索结果，OpenAI 发布了 GPT-4o..."

####场景 2：测试 Redis 持久化记忆1. 先告诉它：`{"session_id": "dev_001", "query": "我叫 Alex，是个前端开发。"}`
2. **重启你的 Python 服务** (Ctrl+C 然后重新运行)。这一步模拟服务器宕机。
3. 再问它：`{"session_id": "dev_001", "query": "我是谁？"}`

**Response:**

> "你是 Alex，一个前端开发。"
> *(因为它从 Redis 里重新读取了历史记录！)*

---

###💡 JS 开发者视角解读 (Deep Dive)1. **`AgentExecutor` vs Promise Chain**
* 之前的代码是 `prompt | llm`，这是一个线性的 Promise 链（Pipeline）。
* `AgentExecutor` 是一个 **While Loop**。它相当于一个状态机：
```javascript
while (true) {
    const nextAction = await model.predict(context);
    if (nextAction.isFinalAnswer) return nextAction.text;
    const toolResult = await executeTool(nextAction.toolName);
    context.push(toolResult); // 把工具结果塞回上下文，再次循环
}

```


* 这就是为什么 Agent 能够处理复杂任务，因为它有“再思考”的机会。


2. **Redis 连接**
* Python 的 `redis://localhost:6379/0` 这种 URL 风格配置与 Node.js 的 `ioredis` 或 `node-redis` 是一致的。
* `RedisChatMessageHistory` 封装了所有 `rpush` (写入 List) 和 `lrange` (读取 List) 的底层操作。


3. **依赖注入 (Dependency Injection)**
* 注意 `tools` 列表。我们把它同时传给了 `llm` (bind_tools，为了让模型知道有哪些工具可用) 和 `AgentExecutor` (为了让代码真正能执行这些工具)。这保持了**定义**与**实现**的分离。



###⚠️ 潜在坑点 (Gotchas)1. **Token 消耗爆炸**：
* Agent 每进行一次搜索，都会把搜索结果（可能很长）塞回 Prompt 再请求一次 LLM。
* 如果历史记录（Redis）里存了太多对话，再加上搜索结果，很容易超出 GPT-3.5 的 Context Window (4k/16k)。
* **解决思路**：生产环境中需要使用 `summary_memory`（只记住摘要）或者 `trim_messages`（只保留最近 N 条）。


2. **Docker 网络**：
* 如果你的 Python 代码也是在 Docker 容器里跑，写 `localhost` 是连不上 Redis 容器的。需要用 Redis 容器的名字（如 `redis`）或者 `host.docker.internal`。



这一步完成后，你基本上已经具备了开发复杂 AI 后端的能力了！感觉如何？需要再深入讲讲如何控制 Token 消耗吗？