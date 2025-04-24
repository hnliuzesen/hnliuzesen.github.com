---
title: Agent Development Kit (ADK) 对接阿里百炼平台
date: 2025-04-24 16:28:41
categories:
  - Computers & Technology
  - AI
tags:
  - Python
  - ADK
  - Agent
---

Claude code 源码泄露让 MCP 大火，Google 最近又推出了 [Agent Development Kit](https://google.github.io/adk-docs/) 和 
[A2A](https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/)，相当于是直接把 MCP Server 和 
MCP Client 都集成到了一起，这就一定要和 LLM 打交道了。  
[官方文档](https://google.github.io/adk-docs/agents/models/)中直接支持的平台就只有自家的 AI Studio 和 Vertex AI，其他平台都是通过 
LiteLLM 来实现的。如果想要在国内用起来 ADK 肯定还是接入国内平台用起来更方便，下面就是 ADK 接入国内平台的使用方式。

<!--more-->

这里使用[阿里百炼](https://bailian.console.aliyun.com/)平台的 API 举例，只要是 OpenAI 兼容的 API 理论上都可以接入，但是因为要涉及到函数调用，模型需要支持 
Function Calling 而且需要比较强的指令遵循才能有比较好的效果，例子中使用 qwen-max 模型。  

## 准备
首先需要按官方[快速上手](https://google.github.io/adk-docs/get-started/quickstart/)教程中的步骤把项目搭建好。  
搭建时可以不填 `API KEY`，后面用的是百炼的 `KEY`。  
搭建完成后，项目结构如下所示  
```text
├── README.md
├── main.py
├── multi_tool_agent
│         ├── __init__.py
│         └── agent.py
├── pyproject.toml
└── uv.lock
```

## 修改 Agent
要修改的内容在 `agent.py` 中，创建的 Agent 如果使用支持的模型，参数中的 model 直接填写模型名称就行，这里我们使用第三方平台的模型，参考
[Models](https://google.github.io/adk-docs/agents/models/#using-open-local-models-via-litellm) 这一节文档，model 
参数要替换为 LiteLlm。  

### 1. 安装 LiteLLM
首先，替换之前需要先安装一下 [LiteLLM](https://www.litellm.ai/)  
```shell
uv add litellm
```

### 2. 引入 LiteLlm
安装完成后在项目中引入 LiteLlm  
```python
from google.adk.models.lite_llm import LiteLlm
```

### 3. 修改 model 参数
修改 model 参数为 LiteLlm
```python
model=LiteLlm(
    model="openai/qwen-max-latest",
    api_key="sk-********************************",
    api_base="https://dashscope.aliyuncs.com/compatible-mode/v1"
)
```

### 这里有两点重要的配置细节需要注意
- 模型前面平台**必须**写 openai，否则 LiteLLM 不认会报错，或者按 LiteLLM 
[文档](https://docs.litellm.ai/docs/providers/custom_llm_server)中创建自己的 CustomLLM，如果有需求的话可以创建，能实现更多的控制。
- api_base 要用[阿里云文档](https://help.aliyun.com/zh/model-studio/use-qwen-by-calling-api)中 OpenAI 
兼容的地址，并且**必须**要带上 `/v1`。如果用了 DashScope 的地址，就需要自己创建 CustomLLM 来对结构做转换并注册到 
`litellm.custom_provider_map` 中再使用了。

最终完整的 root_agent 长下面这样  
```python
root_agent = Agent(
    name="weather_time_agent",
    model=LiteLlm(
        model="openai/qwen-max-latest",
        api_key="sk-********************************",
        api_base="https://dashscope.aliyuncs.com/compatible-mode/v1"
    ),
    description=(
        "Agent to answer questions about the time and weather in a city."
    ),
    instruction=(
        "You are a helpful agent who can answer user questions about the time and weather in a city."
    ),
    tools=[get_weather, get_current_time],
)
```

## 运行结果
使用命令 `adk web` 运行  
最终运行效果应该和官方实例几乎相同，就说明接入成功了。  

{% asset_img example.webp "Run the result" %}

成功运行后就可以复制这些 Agent 搭建自己的 Agentic 了。  
每个 Agent 都可以使用不同平台的不同模型，如果没有用到 tools，只是做一些简单的工作，比如问候之类的，使用免费的模型也是可以的。  
复杂的任务比如需要编排、调用其他 Agent 的模型就用性能好一些的模型，以实现成本优化。
