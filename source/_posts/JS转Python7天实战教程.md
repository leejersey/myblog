---
title: JS转Python7天实战教程
date: 2025-12-15 12:58:34
tags: python
---

作为一个 JS 开发者，你已经掌握了 80% 的编程核心逻辑。这份教程的设计思路是：**“跳过你已知的，深挖你未知的，纠正你误解的”**。

这是一份为期 **7 天** 的速成路径，旨在帮你从“会写 Python 语法的 JS 开发者”转变为“写出 Pythonic 代码的工程师”。

------

# 🚀 JS 转 Python 7 天实战教程

## 第一阶段：语法重塑与环境习惯 (Day 1-2)

### Day 1: 忘记大括号，习惯缩进与环境

**目标**：配置环境，理解 Python 的包管理（痛点所在）。

1. **虚拟环境 (Virtual Environments)**

   - **JS 思维**：`npm install` 把包放在项目下的 `node_modules`，自动隔离。

   - **Python 现实**：`pip install` 默认安装到全局。**必须**养成习惯：

     ```bash
     # 1. 创建虚拟环境 (类似生成一个空的 node_modules)
     python3 -m venv .venv
     
     # 2. 激活环境 (必须激活！)
     # Mac/Linux:
     source .venv/bin/activate
     # Windows:
     .venv\Scripts\activate
     
     # 3. 安装包
     pip install requests
     ```

2. **基本数据结构差异**

   - **List**: 学习切片 `list[1:3]`，这是 Python 最强的语法糖。
   - **Tuple (元组)**: JS 没有这个。它是“不可变的数组”。`t = (1, 2)`。主要用于函数返回多值。
   - **Set**: 去重神器。`list(set([1, 2, 2, 3]))`。

3. **练习任务**：

   - 写一个脚本，读取一个文本文件，统计每个单词出现的次数（使用 `dict`），并按出现频率排序输出。

### Day 2: 函数与高级控制流

**目标**：理解参数传递和推导式。

1. **参数魔法**
   - `*args` 和 `**kwargs`：对应 JS 的 `...args`。
   - **坑**：默认参数不要用可变对象（昨天提到的 `def func(a=[])` 是禁忌）。
2. **列表推导式 (List Comprehensions)**
   - 替代 `map` 和 `filter`。
   - `[user['name'] for user in users if user['age'] > 18]`
   - **进阶**：字典推导式 `{u['id']: u for u in users}` (把数组转为 Map)。
3. **练习任务**：
   - 用 Python 复写一个你常用的 JS 工具函数（比如处理日期的格式化），并尝试使用 Type Hints（类型提示）。

------

## 第二阶段：面向对象与魔术方法 (Day 3-4)

### Day 3: 类 (Class) 与 Self

**目标**：理解 Python 是真 OOP，不是原型链。

1. **Self 的本质**
   - JS 的 `this` 是动态绑定的（谁调用指向谁）。
   - Python 的 `self` 是显式传递的（定义方法时第一个参数必须写）。
2. **魔术方法 (Magic Methods)**
   - Python 的核心。一切皆对象，一切操作符皆方法调用。
   - `__init__`: 构造函数。
   - `__str__`: 对应 JS `toString()`。
   - `__len__`: 定义 `len(obj)` 的行为。
   - `__getitem__`: 让对象可以像数组一样 `obj[key]` 访问。
3. **练习任务**：
   - 创建一个 `ShoppingCart` 类，实现 `len(cart)` 返回商品数量，`print(cart)` 打印清单。

### Day 4: 模块化与异常处理

**目标**：组织代码结构。

1. **Import 系统**

   - `__init__.py`：让文件夹变成包。
   - 理解绝对导入 vs 相对导入。

2. **异常处理 (EAFP 风格)**

   - **JS 思维**：LBYL (Look Before You Leap) -> `if (obj && obj.key) ...`

   - **Python 思维**：EAFP (Easier to Ask for Forgiveness than Permission)

     ```Python
     try:
         val = data["key"]
     except KeyError:
         val = None
     ```

   - Python 鼓励直接做，出错了再捕获，而不是层层 `if` 检查。

------

## 第三阶段：现代工程化与 Web 开发 (Day 5-6)

### Day 5: 现代化工具与 Type Hints

**目标**：写出 TypeScript 感觉的 Python。

1. **Type Hints**

   - Python 3.6+ 支持。
   - `def fetch(url: str) -> dict[str, any]:`
   - 配合 VS Code 的 Pylance 插件，体验极佳。

2. **Pydantic (必学)**

   - 这是 Python 界的 Zod / Joi，但更强大。
   - 它是 FastAPI 的基石，用于数据校验和序列化。

   ```Python
   from pydantic import BaseModel
   
   class User(BaseModel):
       id: int
       name: str
       is_active: bool = True
   ```

### Day 6: Web 框架 (FastAPI)

**目标**：因为你是 JS 开发者，FastAPI 是你最亲切的选择。

1. **FastAPI vs Express**
   - FastAPI 自带 Swagger UI (`/docs`)。
   - 基于 Type Hints 自动生成接口文档和参数校验。
2. **异步编程 (Async/Await)**
   - Python 的 `async def` 和 `await` 与 JS 几乎一样。
   - 注意：Python 的标准库（如 `time.sleep`）是阻塞的，异步代码里要用 `asyncio.sleep`。

------

## 第四阶段：实战与生态 (Day 7)

### Day 7: 实战项目 - AI Agent 接口

**结合你的背景（开发者、想做 AI Agent 账号）**，我们做一个有实际价值的项目。

**项目目标**：用 FastAPI 写一个后端，接收用户输入，调用 OpenAI API，返回结果。

**技术栈**：

- **Web**: FastAPI
- **Env**: `python-dotenv` (读取 .env)
- **Http**: `httpx` (异步的 axios)
- **LLM**: `langchain` 或 `openai` 官方库

**代码骨架预览**：

```Python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel
import openai
import os
from dotenv import load_dotenv

load_dotenv() # 加载 .env 里的 key

app = FastAPI()
client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

class PromptRequest(BaseModel):
    content: str

@app.post("/chat")
def chat_with_ai(req: PromptRequest):
    # 这里演示同步调用，生产环境建议用 async
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": req.content}]
    )
    return {"reply": response.choices[0].message.content}

# 运行: uvicorn main:app --reload
```

------

## 📚 推荐资源 (JS 开发者特供版)

1. **书/文档**：
   - **《Fluent Python》 (流畅的 Python)**：**必读神书**。不要看入门书，直接看这本。它讲的是如何利用 Python 的特性（魔法方法、生成器等）写出优雅代码。
   - **FastAPI 官方文档**：写得非常好，甚至可以当作 Python 异步教程来看。
2. **避坑指南 (JS -> Python 常见错误)**：
   - **变量作用域**：Python 没有块级作用域（`if` 里的变量外面能访问），只有函数、类、模块作用域。
   - **闭包**：在内部函数修改外部函数的变量，需要用 `nonlocal` 关键字。
   - **JSON**：Python 的 `True/False` 是大写，JSON 是小写。一定要用 `json.dumps()` 和 `json.loads()`，不要手动拼字符串。

下一步建议：

不用从头开始看视频。直接用 Python 重构 一个你以前写的简单的 Node.js 脚本（比如爬虫、文件处理脚本）。这是最快的学习方式。

下一节课，我可以带你写第一个 **FastAPI + LangChain** 的 Demo。