# 刘泽森

**高级后端 / AI 工程师 | Java · Python · Agentic AI · 分布式系统 · 技术领导力**

<a href="tel:+8615515595029">15515595029</a> · [hnliuzesen@gmail.com](mailto:hnliuzesen@gmail.com) · [liuzesen.com](https://liuzesen.com) · [GitHub](https://github.com/hnliuzesen)

## 个人简介

11 年软件研发经验，长期从事后端、分布式系统与基础设施研发，近 4–5 年持续将大语言模型与 Agentic AI 落地到生产环境。具备 Java/JVM、Python、PostgreSQL、Kafka、Kubernetes 等生产系统经验，也有 RAG、Agent Loop、Tool Use、Memory、Context Engineering、MCP、AI 输出验证与评测等 AI 工程实践。

在黑湖科技参与面向 30,000+ 企业工厂的多租户工业 SaaS 建设，完成核心服务性能优化、数据库与中间件治理、P0/P1 事故处置及多项 AI 功能落地；在 TELOS 担任技术负责人，负责 AI 工程规范、RAG Pipeline、异步 LLM 数据链路与 CI/CD。擅长在复杂业务和不确定需求中做工程权衡，把模型能力、业务规则、验证机制和用户反馈组合成可上线、可维护的产品。

代表性成果：PostgreSQL 慢查询 **7.8s → 140ms**；核心用户服务 **10 Pods → 2 Pods** 且稳定承载峰值 QPS 1,000；Kafka Connector 基础设施成本降低 **97%**；主导/参与 **70+** 候选人面试；生产落地 ASR+LLM 工单、容错 AI 导入、RAG Pipeline，并完成 memory-driven Multi-Agent 发货助手架构设计。

## 技术能力

**编程语言：** Java · Python · Kotlin · JavaScript / TypeScript · SQL

**后端与分布式系统：** Spring Boot · Spring Framework · Kafka · Redis · Node.js · Netty · EasyExcel · 多租户架构 · 异步任务 · 服务拆分 · 数据库分片

**数据库与数据系统：** PostgreSQL（高级）· pgvector · MySQL · Oracle · Hive · ClickHouse · InfluxDB

**AI / Agentic 系统：** LLM API 集成 · Agent Loop · Tool Use · Memory / State · Context Engineering · MCP · Skills 机制 · RAG Pipeline · Rerank · Prompt Engineering · AI 输出验证与 Eval · ASR+LLM · OpenCV · Google ADK

**工程与可观测性：** GitLab CI/CD · GitHub Actions · Docker · Kubernetes · Linux · Grafana · Kibana / ELK · OpenTelemetry · JUnit · Mockito · JMH · Code Review · TDD

**云与平台：** 阿里云 · Azure · AWS · Cloudflare Workers

## 工作经历

### TELOS（涌启爱） · 技术负责人
**2026.01 – 2026.04**

- 直接负责 3 人工程团队的技术方向、交付节奏与跨职能协作，建立 PR、Code Review 与 CI/CD 工作流。
- 独立设计并实现生产级 **RAG Pipeline**：使用 pgvector 作为向量存储，按章节与 chunk 两阶段检索，低置信度自动降级，Top 10 结果经 rerank 取 Top 1，兼顾召回率和精度。
- 构建 **Python 异步 LLM 数据处理链路**：Java 主程序通过 PostgreSQL 提交任务，Python 服务异步消费，调用 LLM 提取结构化信息并生成 Embedding 写入 pgvector，打通数据采集、模型调用、向量化与入库全流程。
- 设计数据库驱动的 **Skills 加载机制**，将产品说明、角色配置等上下文模块化，降低 Prompt 硬编码和上下文散落问题。
- 建立团队 **AI 工程治理规范**，定义 AI 约束、渐进式上下文披露、代码评审规则和 AI 代码坏味道，减少跨功能回归与协作摩擦。
- 在 Redis + MQ 与 PostgreSQL 方案之间做架构评估，推动使用更简单的 PostgreSQL 无日志表方案，移除两套冗余中间件，降低系统复杂度与运维成本。
- 从零搭建 GitLab CI/CD，实现分支到对应环境的自动部署，并规范 PR 与 Code Review 流程。

### 黑湖科技（Blacklake） · 高级后端工程师
**2021.05 – 2025.12**  
Series C 工业 SaaS，服务 30,000+ 企业工厂

#### AI / Agentic 系统

- **语音工单 ASR+LLM：** 面向工厂一线员工设计自然语音报工功能，通过“固定字段预填 → LLM 语义校正 → 人工复核”三层防御机制，降低方言 ASR 偏差并保证生产数据准确性；48 小时黑客松完成后端开发并上线，获一等奖与最具创新奖。
- **容错 AI 导入 Pipeline：** 将 ChatGPT 接入 Excel 导入流程，把不同工厂、多来源、多格式表单字段映射到系统 Schema；将 AI 提取结果与系统已有数据交叉验证，不匹配字段高亮供用户确认后导入。
- **发货助手 Agent：** 基于 Google ADK 设计 memory-driven Multi-Agent 工作流，采用 Sequential + Loop Agents，将“获取数据 → 确认分配 → 确认发货单”分阶段执行；利用 state/memory 沉淀不同工厂个性化发货策略，预计单次节省 30–60 分钟 Excel 人工核对时间。
- 多次前往客户现场进行需求调研和问题诊断，将真实工厂工作流反馈转化为 AI 产品设计和迭代，而非仅基于离线 Demo 做功能判断。

#### 后端、性能与稳定性

- **PostgreSQL 慢查询优化：** 通过分析执行计划、重设索引并消除冗余 JOIN，将高频报表查询从 **7.8 秒降至 140 毫秒**，性能提升约 98%，并将排查方法沉淀为团队文档。
- **核心用户服务重构：** 梳理跨系统调用链路，引入缓存并裁剪冗余功能，结合 JMH 性能基准测试完成服务重构；最终由 **10 Pods 降至 2 Pods**，稳定承载峰值 QPS 1,000，上线后零重大故障。
- **Kafka Connector 成本优化：** 使用 ECS 自建 Kafka Connect 替换阿里云托管 Connector，保持功能等效，基础设施成本降低 **97%**。
- **PostgreSQL ltree 优化 BOM：** 使用 ltree 表达制造业 BOM 层级路径，改善父子层级查询性能并降低复杂查询成本。
- 参与数据库分片设计、实施与上线，提升高并发多租户场景下的数据层扩展能力。
- 使用 DDD 重写小工单系统，将 Kotlin/MySQL 迁移至 Java/PostgreSQL，通过领域建模统一工程师与制造业务专家的语言。

#### 可观测性与事故响应

- 独立搭建 P0/P1/P2 分级告警体系，结合飞书、Grafana、Kibana/ELK 完成线上异常发现、租户级异常流量检测与事故定位。
- 处理数据库连接池耗尽导致的 P0 事故，完成停流、服务与数据库恢复，并推动数据库参数和 SQL 治理后续改进，MTTR 约 20 分钟。
- 处理 Kafka 消费背压导致主服务受压问题，识别“扩容反而加剧数据库压力”的根因，通过反向降容消费服务、Feature Flag 下线功能恢复主服务可用性。
- 处理共享用户服务级联故障，完成调用路径重构、缓存层引入和专属用户服务拆分，彻底消除级联依赖。

#### 工程质量与技术影响力

- 推动团队建立单元测试与集成测试边界，提供 JUnit + Mockito 参考实现，并推动 TDD 在新功能中落地。
- 引入 Alibaba P3C 与 Spotless，参与制定后端研发规范并持续推动 Code Review。
- 主导/参与 **70+** 候选人面试，终面通过率超过 60%，参与绩效评估与人才识别。
- 独立开发 Jira + GitLab 数据驱动的飞书测试效能机器人，并引入 RAG 做 QA 频道 AI 问答，被跨业务线团队采用。

### 独立开发者 · 全栈工程师
**2019.01 – 2021.04**

- 开发医疗工具箱小程序，支持医生在手机端计算体表面积相关用药量并查询检验指标，用户增长主要来自医生间分享传播。
- 为政府相关单位开发招投标软件，将评标逻辑程序化以保证结果中立和投标信息隔离。
- 开发医院绩效工资分配小程序，将科室每月人工计算过程自动化，并在云端保存配置和历史记录。
- 开发 Angular 医疗检验报告页面，参与临床数据展示与业务交互设计。

### 千寻位置 · 高级 Java 工程师
**2018.08 – 2018.12**

- 基于 XXL-Job 搭建可执行代码片段的任务层，支持线上卫星数据的临时计算和按需执行。
- 使用 Netty 处理大量基站服务器产生的实时卫星数据，并整合上传至阿里云 OSS 供下游使用。

### 携程旅行 · 高级 Java 工程师
**2017.03 – 2018.08**

- 作为团队中少数同时熟悉 Java 与 ASP.NET 的工程师，参与并推动微软技术栈向开源技术栈迁移，选择 Kotlin 缩短迁移成本并最终被团队采用。
- 将下单流程中的同步操作改为异步消息队列处理，减少高流量场景用户等待时间。
- 使用 Vue + Spring Boot 开发双数据库数据核对工具，帮助测试快速验证 MSSQL/MySQL 双写一致性。
- 编写 Hive ETL 任务，将订单与系统数据汇总到业务看板和异常告警系统，提升数据可观测性。

### 沪江英语 · 基础架构工程师
**2015.11 – 2017.02**

- 将多个业务线独立维护的 Solr 服务整合升级为统一 SolrCloud 集群，实现跨产品线搜索与统一管理。
- 负责文档权限与状态管理模块设计，使用位运算高效实现权限及状态流转。
- 基于 React 开发 CAT 日志系统权限管理模块。

### 普元信息 · Java 工程师
**2014.09 – 2015.10**

- 在中国移动大数据平台中优化 Oracle 查询，使用函数索引与 RowID 等方案，将超过 30 秒的查询降至秒级。
- 参与服务器虚拟化平台建设并部署 Huawei FusionCompute，帮助团队获得华为官方认证。
- 在公司内部分享 Docker 等容器化技术，并持续跟进 OpenStack 与云计算基础设施方向。

## 开源贡献

- **LiteLLM：** 为 DeepSeek 增加 `thinking={"type":"disabled"}` 支持，完成 `reasoning_effort="none"` 参数映射、API Base 更新及回归测试；涉及 6 个文件，CI 全部通过，Greptile Review 5/5。
- 参与多个开源项目的文档修正、模型列表更新、异常场景修复与工具能力补充，包括 Google ADK 文档、Magisk、Cherry Studio 文档、Kindle 下载工具、小宇宙播客下载工具等。

## 教育背景

**郑州大学（211）**  
计算机科学与技术 · 工学学士  
2010.09 – 2014.06

## 证书与荣誉

- 上海市重点产业领域人才专项奖励
- [软件设计师（中级）](https://r2.liuzesen.com/%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1%E5%B8%88_%E4%B8%AD%E7%BA%A7.jpg)  
- [阿里巴巴 P3C 认证开发者](https://r2.liuzesen.com/CLDT02180300010859.jpg) 
- [Google Flutter Clock 挑战赛证书](https://r2.liuzesen.com/Flutter_Clock.png)
- 工业视觉相关发明专利（实审中）
