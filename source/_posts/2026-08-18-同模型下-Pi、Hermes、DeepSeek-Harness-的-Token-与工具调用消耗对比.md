---
title: 同模型下 Pi、Hermes、DeepSeek Harness 的 Token 与工具调用消耗对比
date: 2026-08-18 17:01:13
categories:
  - AI
tags:
  - harness
---

最近，DeepSeek 推出了官方的 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)，
设计理念是一切皆插件，市面上又多了一个可以自己部署的 AI Agent，如果只是想给它安排一些简单的任务，相比功能全面的
[hermes](https://github.com/nousresearch/hermes-agent) 和极简的
[pi](https://github.com/earendil-works/pi)，哪个更合适？  

我对比了一下这三个 Agent 在不同任务上的 Token 消耗，先放结果：

|                  |  pi  | Hermes | DSH  |
|:-----------------|:----:|:------:|:----:|
| say hi           | 4.7K | 19.4K  | 9.5K |
| 执行任务         | 129K |  364K  | 270K |
| 任务结果         | 正确 |  正确  | 正确 |
| 工具调用请求次数 |  5   |   31   |  12  |
| 工具调用成功次数 |  5   |   31   |  7   |

在测试过程中，还发现了很多意想不到的差异

<!--more-->

## 测试配置

Pi Coding Agent (文中简称 pi)：[v0.84.2](https://www.npmjs.com/package/@earendil-works/pi-coding-agent/v/0.84.2)  
Hermes Agent (文中简称
Hermes)：[v0.20.3 (2026.8.16.2)](https://github.com/NousResearch/hermes-agent/releases#release-v2026.8.16.2)  
DeepSeek Harness (文中简称 DSH)：[0.1.0-rc.5](https://github.com/deepseek-ai/deepseek-harness)  
模型：[agnes-2.5-flash](https://www.agnes-ai.com/en/docs/agnes-25-flash)  
技能：[巴菲特价值投资买入前 Checklist](https://github.com/xbtlin/ai-berkshire/blob/main/skills/investment-checklist.md)

## 空载 Say Hi 测试

这一轮测试在三个 Agent 配置好相同的 skills 后，只发送一个单词 `hi`，记录模型的 Token 消耗，然后在同一 Session 中再次发送
`hi`，观察上下文增长情况。

### Pi

#### Token 消耗

使用 [`/session`](https://pi.dev/docs/latest/sessions#session-commands) 命令查看当前会话的用量

##### 第一次发送

```text
 File: /home/user/.pi/agent/sessions/--home-user--/2026-08-18T03-49-09-788Z_01a012fc-c21c-7488-aa3a-81e0db37bfcc.jsonl
 ID: 01a012fc-c21c-7488-aa3a-81e0db37bfcc

 Messages
 Total: 2
 User: 1
 Assistant: 1
 Tools: 0 calls, 0 results

 Tokens
 Input: 4,743
   Cached: 4,608 (97.2%)
   Uncached: 135
 Output: 24
 Total: 4,767
```

##### 第二次发送

```text
Session Info

 File: /home/user/.pi/agent/sessions/--home-user--/2026-08-18T03-49-09-788Z_01a012fc-c21c-7488-aa3a-81e0db37bfcc.jsonl
 ID: 01a012fc-c21c-7488-aa3a-81e0db37bfcc

 Messages
 Total: 4
 User: 2
 Assistant: 2
 Tools: 0 calls, 0 results

 Tokens
 Input: 9,506
   Cached: 4,608 (48.5%)
   Uncached: 4,898
 Output: 50
 Total: 9,556

 Cost
 Total: $0.000
 Cache Re-billed: 4,743 tokens, 1 miss
```

### Hermes

#### Token 消耗

使用 [`/usage`](https://hermes-agent.nousresearch.com/docs/reference/slash-commands#info)
命令查看当前会话的用量

##### 第一次发送

```text
📊 Session Token Usage
────────────────────────────────────────
Model:                     agnes-2.5-flash
Input tokens:                  11,182
Output tokens:                     27
↳ Reasoning (subset):              15
Prompt tokens (total):         19,374
Completion tokens:                 27
Total tokens:                  19,401
API calls:                          1
Session duration:                 27s
────────────────────────────────────────
Current context:  19,374 / 256,000 (8%)
Messages:         2
Compressions:     0
```

##### 第二次发送

```text
📊 Session Token Usage
────────────────────────────────────────
Model:                     agnes-2.5-flash
Input tokens:                  30,577
Output tokens:                     51
↳ Reasoning (subset):              30
Prompt tokens (total):         38,769
Completion tokens:                 51
Total tokens:                  38,820
API calls:                          2
Session duration:                 58s
────────────────────────────────────────
Current context:  19,395 / 256,000 (8%)
Messages:         4
Compressions:     0
```

### DSH

#### Token 消耗

通过页面右上角 `Session log` 按钮下载 session 的 jsonl 文件，从文件内获取会话的用量数据  
DSH 中输入和缓存是分开列出的，所以最终对比时用 `inputTokens + cacheReadTokens`

##### 第一次发送

```json
{
  "inputTokens": 2889,
  "outputTokens": 42,
  "cacheReadTokens": 6656
}
```

##### 第二次发送

```json
{
  "inputTokens": 2918,
  "outputTokens": 26,
  "cacheReadTokens": 6656
}
```

测试 DSH 时，第一次忘记安装相同的 skills，所以用量为 8874 tokens，安装了 skills 后用量增加了 671 tokens (7.6%)
，说明 DSH 采用了渐进式披露机制

### 对比

|                    |         Pi |        Hermes |        DSH |
|--------------------|-----------:|--------------:|-----------:|
| 第一次 `hi` prompt | 4,743 (1x) |   19,374 (4x) |  9545 (2x) |
| 第二次 `hi` prompt | 4,763 (1x) |   19,395 (4x) |  9574 (2x) |
| 固定上下文大致规模 | ~4.7k (1x) | ~19.3k (4.1x) | ~9.5k (2x) |

空载测量结果 **pi 最轻，DSH 居中，Hermes 最重**

```text
Pi       █████                     4.7K  
DSH      ██████████                9.5K  
Hermes   ████████████████████     19.4K
```

## 执行任务测试

提示词: 使用 investment-checklist 检查`XX精密`，需要搜索时使用
[tavily-search](https://docs.tavily.com/documentation/agent-skills)
> 为了方便对比结果，这里要求 Agent 都用相同的搜索引擎获取结果

### pi

#### Token 消耗

```text
Session Info

 File: /home/user/.pi/agent/sessions/--home-user--/2026-08-18T06-10-35-439Z_01a0137e-3d2e-7918-8248-07260b4e4931.jsonl
 ID: 01a0137e-3d2e-7918-8248-07260b4e4931

 Messages
 Total: 13
 User: 1
 Assistant: 4
 Tools: 8 calls, 8 results

 Tokens
 Input: 129,492
   Cached: 64,512 (49.8%)
   Uncached: 64,980
 Output: 4,852
 Total: 134,344

 Cost
 Total: $0.000
 Cache Re-billed: 3,792 tokens, 1 miss
```

#### 搜索关键词

pi 的搜索关键词在终端中会展示出来，查看 session 对应的 jsonl 文件也能找到

- XX精密 0023** 基本面 财务数据 ROE 毛利率 2024 2025
- XX精密 估值 PE PB 股价 市值 2025
- XX精密 护城河 竞争优势 竞争对手 市场份额
- XX精密 管理层 董事长 实控人 资本配置 回购分红
- XX精密 最新动态 2025 业绩 收购 风险

#### 执行结果

```markdown
分析报告已写入 `~/巴菲特Checklist-XX精密.md`。

---

## XX精密（0023**）Checklist 结论： **❌ 未通过**

[其余内容省略]
```

### Hermes

#### Token 消耗

```text
📊 Session Token Usage
  ────────────────────────────────────────
  Model:                     agnes-2.5-flash
  Input tokens:                 119,807
  Output tokens:                  9,844
  ↳ Reasoning (subset):           1,952
  Prompt tokens (total):        363,775
  Completion tokens:              9,844
  Total tokens:                 373,619
  API calls:                         10
  Session duration:                  9m
  ────────────────────────────────────────
  Current context:  54,235 / 512,000 (11%)
  Messages:         27
  Compressions:     0
```

#### 搜索关键词

Hermes 的工具调用存储在 `state.db` 中的 `messages` 表的 `tool_calls` 字段内

- XX精密 股票代码 2026
- XX精密 2024 2025 年报 ROE 毛利率 净利润
- XX精密 竞争护城河 PCB FPC 苹果产业链
- XX精密 管理层 董事长 股权激励 资本配置
- XX精密 负债 现金流 自由现金流 2024 2025
- XX精密 股价 市值 PE 估值 2026
- XX精密 0023** 2024年报 营收 净利润 ROE 毛利率 每股收益 EPS
- XX精密 0023** 2025年报 年度报告 财务数据
- XX精密 2026年一季报 营收 净利润
- XX精密 董事长 缪X根 持股 资本配置 并购 M&A
- XX精密 PCB FPC 柔性电路板 全球市场份额 竞争对手 鹏X 欣X
- XX精密 2025年报 营收 净利润 ROE 毛利率 每股收益 EPS 每股净资产
- XX精密 0023** 最新股价 市值 PE 2026年8月
- XX精密 缪X根 实际控制人 持股 创始人
- XX精密 自由现金流 FCF 资本开支 负债率 2024 2025
- XX精密 鹏X控股 深南电路 PCB FPC 竞争对手 市场份额
- XX精密 2024年年报 净利润 归母净利润 营收 同比
- XX精密 2025年业绩预告 快报 净利润增长
- XX精密 苹果 供应商 iPhone FPC 业务收入占比
- XX精密 员工持股计划 回购 分红 2024 2025
- XX精密 0023** 股价 2026年8月 最新
- XX精密 袁X刚 实际控制人 持股 家族 控制权
- XX精密 索X思 光模块 AI算力 收购 并购
- XX精密 2025年年报 净利润率 毛利率 ROE 每股收益 EPS
- XX精密 0023** 总股本 每股收益 每股净资产 2025年报
- XX精密 自由现金流 经营活动现金流 资本开支 2024 2025
- XX精密 毛利率 净利率 行业对比 PCB FPC
- XX精密 风险 客户集中度 苹果依赖 2025 2026
- XX精密 分红 股息率 2024 2025
- XX精密 员工持股计划 2026 行权价
- XX精密 并购 商誉 减值 2025

#### 执行结果

```markdown
报告已完成，写入 /root/巴菲特Checklist-XX精密.md。

XX精密 Checklist 结论：❌ 未通过

[其余内容省略]
```

### DSH

#### Token 消耗

DSH 的 JSON Lines 文件只记录了 6 次 LLM 调用各自的用量，因此需要做汇总

| 调用 | Uncached input | Cache read | 完整 Prompt | Output |
|------|---------------:|-----------:|------------:|-------:|
| 1    |          3,017 |      6,656 |       9,673 |     83 |
| 2    |          6,686 |      6,656 |      13,342 |    661 |
| 3    |          8,439 |     13,824 |      22,263 |    350 |
| 4    |         39,488 |     22,528 |      62,016 |    226 |
| 5    |         68,623 |      9,728 |      78,351 |  6,398 |
| 6    |          6,463 |     78,336 |      84,799 |    527 |

```text
Uncached input:   132,716
Cache read:       137,728
Prompt total:     270,444
Output:             8,245
Total:            278,689
API calls:              6
```

#### 搜索关键词

DSH 的搜索关键词可以通过页面上方的`轨迹`页面查看  
DSH 首次并发调用 tavily-search，6 条命令中 5 条参数错误，只有 1 条执行成功，随后又发起了 4 
次调用，全部成功，并且第二次调用时还对关键词进行了微调

##### 失败

- XX精密 0023** 公司概况 主营业务 2025 2026
- XX精密 0023** 财务数据 ROE 毛利率 净利润 2024 2025
- XX精密 0023** 股价 市值 PE PB 估值 2025 2026
- XX精密 护城河 竞争优势 竞争对手 PCB FPC
- XX精密 0023** 管理层 董事长 持股 资本配置

##### 成功

- XX精密 0023** 最新新闻 重大事件 2025 2026
- XX精密 0023** 财务数据 ROE 毛利率 净利润 2024 2025
- XX精密 0023** 股价 市值 PE 2026年7月
- XX精密 0023** 负债率 现金 商誉 资本开支
- XX精密 创业板 主板 定增 低价 实控人 减持
- XX精密 PCB竞争对手 鹏X控股 深X电路 生X电子 市场份额
- XX精密 0023** 光模块 索X光电 收购 2025 2026 业务结构

#### 执行结果

```markdown
报告已生成并保存至 `巴菲特Checklist-XX精密.md`。

---

## XX精密 (0023**.SZ) — Checklist 快速结论

[此处内容省略]

### 最终结论： **❌ 未通过 Checklist**

[其余内容省略]
```

### 对比

| Agent  | Prompt/Input 总量 |       Output |       总 token | LLM calls |
|--------|------------------:|-------------:|---------------:|----------:|
| Pi     |    129,492 (1.0x) | 4,852 (1.0x) | 134,344 (1.0x) |         4 |
| DSH    |    270,444 (2.1x) | 8,245 (1.7x) | 278,689 (2.1x) |         6 |
| Hermes |    363,775 (2.8x) | 9,844 (2.0x) | 373,619 (2.8x) |        10 |

三个 Agent 都得出了一致的结论：不通过

为了比较真实的结果质量，我将三份报告去除 Agent 名称后发给一位金融从业者朋友进行评价，结果为 **DSH > Pi >> Hermes**。
这一评价仅代表单一样本下的主观体感判断，不是为了给 Agent 能力做排名。

统计搜索关键词时，我发现了一个很有意思的差异：Pi 和 DSH 都在阅读 Skill 后直接使用 tvly 命令搜索；Hermes 则调用 
execute_code 工具编写 Python 代码，将关键词数组传给 subprocess，循环执行 tvly 
指令。这种做法与任务指导中的搜索次数限制存在偏差，最终产生了 31 次 Tavily 调用

## 结果

在任务执行中，pi 仍然保持优势，token 消耗只有 DSH 的一半，而 Hermes 在复杂任务里固定上下文占比下降，所以从空载时的 4 倍缩小为约
2.8 倍。

缓存命中率方面，三者差距不大，命中率都很高。

三者不仅在 Harness 自身的上下文开销上有所不同，执行效率也存在明显差异。虽然最终结论一致，理由也基本重合，但 pi
使用了更短的轨迹达成了相同的目标。  
DSH 首次并发调用工具出错。虽然参数校验失败本身不会产生 Tavily API 用量，但失败信息会进入后续上下文。日志中该阶段之后单轮
Prompt 从约 22K 增长到约 62K；这部分增长同时包含工具错误信息和后续搜索结果，因此不能全部归因于失败调用，但失败轨迹确实会成为后续每轮携带的额外上下文。
Hermes 存在搜索过度和工具调用成本失控的风险。Tavily 搜索超出免费额度后会产生费用，pi 调用了 5 次，DSH 调用了 7 次，Hermes
一共调用了 31 次。其中有很多关键词是重复的，比如 `财务数据、ROE、毛利率、EPS`:

```text
XX精密 2024 2025 年报 ROE 毛利率 净利润
XX精密 0023** 2024年报 营收 净利润 ROE 毛利率 每股收益 EPS
XX精密 2025年报 营收 净利润 ROE 毛利率 每股收益 EPS 每股净资产
XX精密 2025年年报 净利润率 毛利率 ROE 每股收益 EPS
XX精密 0023** 总股本 每股收益 每股净资产 2025年报
```

`现金流`:

```text
XX精密 负债 现金流 自由现金流 2024 2025
XX精密 自由现金流 FCF 资本开支 负债率 2024 2025
XX精密 自由现金流 经营活动现金流 资本开支 2024 2025
```

`估值`:

```text
XX精密 股价 市值 PE 估值 2026
XX精密 0023** 最新股价 市值 PE 2026年8月
XX精密 0023** 股价 2026年8月 最新
```

`竞争`:

```text
XX精密 PCB FPC 柔性电路板 全球市场份额 竞争对手 鹏X 欣X
XX精密 鹏X控股 深X电路 PCB FPC 竞争对手 市场份额
XX精密 毛利率 净利率 行业对比 PCB FPC
```

个人猜测，Hermes 可能采用了更偏迭代式研究的策略，每轮先通过 execute_code 生成一组关键词，在 Python 中使用 subprocess 
循环调用 tvly，工具结果返回后，模型进入下一轮推理，并根据已有结果继续补充搜索。 这种方式可能能获得更充分的信息，但在盲测中没有占优，反而带来了明显的调用放大。 从 
Hermes 顶层轨迹看，只能看到若干次 execute_code 调用，而一次 execute_code 内部可以循环产生多次付费 API 
请求。因此，只观察 Agent 的 tool-call 数量，并不能直接反映真实的外部服务调用次数。在长期无人值守场景中，这部分成本很容易忽略。

DSH 的搜索分为三批。第一轮一次性并发生成了 6 个 Tavily 调用，其中 5 个因参数错误失败，只有 1 个成功。整批结果返回后，因为第一轮有失败，所以又并发生成了
4 个搜索调用进行了第二轮搜索并全部成功。随后模型判断“数据收集充分，现在进行全面的巴菲特价值投资 Checklist 
分析”，但分析过程中又补充了 2 次搜索。最终共规划了 12 次搜索调用，其中 7 次成功、5 次参数校验失败，最终只产生了 7 次 Tavily API 调用。

```text
最新事件
财务
估值
资产负债、商誉、CAPEX
治理、定增
竞争格局
光模块、索X思
```

> 因为把 Tavily skills 安装到了公共 Skills 目录，我的 Codex 在需要搜索时没有用内置搜索，也去用了 `tvly`，而且犯了和 DSH
> 相同的参数错误

而 pi 的 5 条搜索更有条理，先按 checklist 的几个维度切分了问题，然后每个维度做宽泛查询:

```text
1. 基本面、财务
2. 估值
3. 护城河、竞争
4. 管理层、资本配置
5. 最新动态、风险
```

我的需求是让 Agent 调用 Skill 完成明确任务，因此 Pi 更合适：固定上下文最小，Token 消耗最低，工具调用也最克制。

这次测试也说明即使模型、Skill、搜索工具和 Prompt 都相同，不同的 Harness 也会显著影响模型的上下文规模、工具调用偏好和最终成本。

> 又及：DSH 官方目前提供的使用方式以 Web 界面为主。在帮助信息中，除了 `dsh --profile web`，我还看到了 
> `dsh --profile tui`，不过后者目前还无法成功运行
