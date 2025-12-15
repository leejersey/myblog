---
title: 最小可行性（MVP）的 AI Agent 后端
date: 2025-12-15 13:04:09
tags: python
---

太棒了！我们直接进入实战。

我们将构建一个**最小可行性（MVP）的 AI Agent 后端**。

目标：不仅是写 Python，而是体验 Python 在 AI 领域的“乐高积木”式开发体验（LangChain）。

类比：

- **FastAPI** ≈ Express + TypeScript + 自动生成 Swagger 文档
- **LangChain** ≈ AI 编排层的中间件
- **Uvicorn** ≈ Node.js 的运行时（HTTP Server）

------

### 第一步：环境初始化 (The "npm install" Phase)

在你的项目文件夹下打开终端：

```Bash
# 1. 创建虚拟环境 (养成肌肉记忆)
python -m venv .venv

# 2. 激活环境
# Windows:
.venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate

# 3. 安装依赖 (类似 npm install)
# fastapi: web框架
# uvicorn: 服务器
# langchain-openai: LangChain 的 OpenAI 适配器
# python-dotenv: 读取 .env 文件
pip install fastapi uvicorn langchain-openai python-dotenv
```

------

### 第二步：配置文件 (.env)

在根目录新建 `.env` 文件（和 JS 一样）：

代码段

```
OPENAI_API_KEY=sk-your-api-key-here
# 如果你需要配置 base_url (比如用转发)
# OPENAI_API_BASE=https://api.openai.com/v1
```

------

### 第三步：核心代码 (main.py)

新建 `main.py`。我会通过注释，逐行对比 JS 的概念。

Python

```Python
import os
from dotenv import load_dotenv
from fastapi import FastAPI
from pydantic import BaseModel
# LangChain 的现代化语法 LCEL (LangChain Expression Language)
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 1. 加载环境变量
load_dotenv()

# 2. 初始化 FastAPI (类似 const app = express())
app = FastAPI(
    title="My AI Agent API",
    description="专门为 JS 开发者构建的 Python Demo"
)

# 3. 定义数据模型 (类似 TypeScript Interface + Zod 校验)
# Pydantic 会自动处理 JSON 解析和类型错误
class ChatRequest(BaseModel):
    topic: str
    style: str = "幽默" # 默认值

# 4. 初始化 AI 模型
# temperature=0.7 代表创造性
llm = ChatOpenAI(
    model="gpt-3.5-turbo",
    temperature=0.7
)

# 5. 定义 Prompt 模板
# {topic} 和 {style} 是我们要替换的变量
prompt = ChatPromptTemplate.from_template(
    "你是一个抖音爆款文案写手。请用{style}的风格，写一段关于{topic}的短视频脚本，包含分镜描述。"
)

# 6. 定义 API 路由
# @app.post 这是一个装饰器 (Decorator)，类似 NestJS 的 @Post()
# async def 表明这是异步函数
@app.post("/generate-script")
async def generate_script(request: ChatRequest):
    
    # --- 核心魔法区 (LangChain Expression Language) ---
    # 这里的 '|' 管道符被重载了。
    # 数据流向：prompt模板 -> 注入变量 -> 发给LLM -> 解析输出为字符串
    # 这就像 JS 里的 .pipe() 或者 Promise 链
    chain = prompt | llm | StrOutputParser()
    
    # invoke 调用链条
    result = await chain.ainvoke({
        "topic": request.topic,
        "style": request.style
    })
    
    return {
        "success": True,
        "data": result
    }

# 7. 启动提示
# 通常我们在命令行启动，但也可以写在这里便于调试
if __name__ == "__main__":
    import uvicorn
    # 相当于 app.listen(8000)
    # reload=True 相当于 nodemon，代码变动自动重启
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)
```

------

### 第四步：运行与测试 (Run & Test)

1. 运行服务：

   在终端输入：

   ```bash
   python main.py
   ```

   *(或者直接用 `uvicorn main:app --reload`)*

2. 体验“魔法” (Swagger UI)：

   FastAPI 最爽的一点来了。你不需要写 Postman collection。

   - 浏览器打开：`http://127.0.0.1:8000/docs`

   - 你会看到自动生成的 API 文档。

   - 点击 `/generate-script` -> `Try it out`。

   - 输入 JSON：

     JSON

     ```
     {
       "topic": "程序员加班",
       "style": "凡尔赛"
     }
     ```

   - 点击 `Execute`。

------

### 第五步：关键点复盘 (JS Dev 视角)

1. 装饰器 (@app.post)：

   在 JS 中，你可能习惯 app.post('/path', handler)。Python 使用装饰器模式，直接把路由逻辑“贴”在函数上面。这让代码结构更扁平。

2. 类型提示 (request: ChatRequest)：

   FastAPI 读取了这个类型提示。如果你传的 JSON 缺了 topic 字段，或者 style 不是字符串，FastAPI 会在进入函数之前就报错拦截（422 Validation Error）。这省去了你在函数里写 if (!req.body.topic) ... 的繁琐工作。

3. 链式调用 (|)：

   LangChain 的 prompt | llm | parser 语法非常像 Unix 管道或者函数式编程。对于 JS 开发者，这很容易理解为数据的 Stream 流转。

------

### 你的下一步 (Next Step)

你现在的代码是一个“无记忆”的单次请求。

作为“运营助理”项目的开发者，你肯定需要**“记忆 (Memory)”**功能，让 AI 记得上下文（比如你之前跟它说过你是 JS 开发者）。

**你想试着把“记忆功能”加进去吗？** 这会涉及到 Python 的类（Class）和简单的数据库概念（或者内存存储）。