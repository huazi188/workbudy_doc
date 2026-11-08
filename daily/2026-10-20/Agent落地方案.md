# 2026-10-20 Agent落地方案

**总览**
**中文**：国内最值得照着搭的是「售后工单 Agent 的四类工具 + 串联 RAG」——它把「模型回答得对」到「事情办得成」中间那层工具设计讲透了，还给出了切块参数与降级路径。国外最值得照着搭的是 **LangGraph 的订单风控状态机**（用 `interrupt` 做人工审核断点，`notify` 节点失败不阻塞主流程），这套骨架可以原样嫁接到任何「自动通过 + 抽人复核」的业务上。
**English**: The most directly reusable Chinese case today is a post-sales ticket agent built on four tool classes wired in series with RAG — it names the layer between "the model answers correctly" and "the task actually gets done," with concrete chunking parameters and a degradation path. The most directly reusable international case is a LangGraph order-risk state machine that uses `interrupt` for human review checkpoints and keeps a failing `notify` node from blocking the main flow — a skeleton that transplants onto any "auto-approve with sampled human review" process.

> 本期配比：国内 4 条（A类 3 条、B类 1 条），国外 3 条（A类 2 条、B类 1 条）。B类均已注明骨架来源与补全依据。

---

# 第一部分：国内 / China

### 1. 售后工单 Agent：意图识别 → 四类工具 → 串联 RAG，把"回答得对"变成"事情办得成" 【售后工单·某企业级 SaaS 团队·LLM+bge-m3 本地部署】

**中文**：

**① 场景与边界**
- 干什么：客服在工单系统里输入客户问题，Agent 判断问题类型、调对应工具取真实数据或查知识库，生成带信息来源的草稿回复，并给出工单分类与建议处理人
- 输入：客户原始问题文本、当前工单上下文（工单号、客户 ID）、会话历史
- 输出：结构化结果——回复正文、工单分类标签、建议处理人、引用来源链接
- 不干什么：**不直接发送回复**（必须由客服人工确认后点发送）、**不做退款/扣款等写操作**（只查询状态）、**不做工单流转审批**、**不在检索资料之外自由发挥**——提示词明确要求回复只能基于检索到的资料和上下文

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `order_query` | 查订单/退款真实状态 | `{order_id: str}` | `{status, refund_time, refund_channel, eta_days}` |
| `ticket_history_search` | 检索该客户历史工单 | `{customer_id, keyword?, top_k=5}` | 相似工单列表（含最终处理结论） |
| `knowledge_search` | 产品手册/标准话术/优质工单 RAG 检索 | `{query, top_k=5}` | 前 5 个 chunk + 相似度分 |
| `customer_info_query` | 查客户等级、VIP 标记、服务等级 | `{customer_id}` | `{level, is_vip, sla_hours}` |
| `intent_classify`（内部） | 意图识别（查单/知识型/投诉/其他） | 问题文本 | `{intent, confidence}` |

**③ 编排流程**

```
[1] intent_classify
     ├─ 状态型（查单/退款） ──> order_query ──┐
     ├─ 知识型（"A300连不上"） ─> knowledge_search ─┤
     └─ 混合型 ──> order_query + knowledge_search ─┤
                                                  ▼
[2] 结果回填上下文 ──> LLM 生成结构化输出
     {reply_text, ticket_category, suggested_owner, citations[]}
                                                  ▼
[3] 写回工单：草稿回复 + 分类 + 处理人 + 来源链接
                                                  ▼
[4] 人工扫一眼确认 → 发送（人工在环，不可跳过）
```

关键设计点：`order_query` 与 `knowledge_search` **不是互斥的，是串联的**——状态型问题先调 API 拿真实数据，知识型问题先做向量检索，两类结果都作为上下文再交给模型。

**④ 关键配置**

```
# 知识库切块（原文给出的实际参数）
chunk_size      = 512        # 字符
chunk_overlap   = 64         # 字符，避免检索时把关键信息截断
embedding_model = "bge-m3"   # 本地部署，不出网
retrieval_top_k = 5          # 取相似度最高的前 5 块
temperature     = 0.2        # 推导：工单场景要稳定复现，取低温度；原文只给了"别让模型自由发挥"的原则
max_steps       = 6          # 推导：意图识别 + 最多 2 次工具调用 + 生成 + 结构化 = 5~6 步
timeout         = 10         # 秒，推导：原文未给；工单系统单次响应要求
max_retries     = 2          # 推导：同一工具连续失败 2 次即换策略/转人工
context_budget  = 60000      # token，推导：留推理空间又避免过载
```

**⑤ 失败与兜底**
- 意图识别置信度低 → 不猜，直接按"其他"分类转人工，把原始问题完整带上
- 工具超时 → 返回可读错误给模型，由模型决定重试还是换方案；不静默返回空结果（空结果会被模型当成"没有退款记录"，这是最危险的失败模式）
- 检索命中为空 → 明确告知"知识库无匹配"，禁止模型用先验知识编答案
- 客户信息缺失（订单号抽取失败）→ 回问客户/客服补一句，不推断订单号
- 写操作一律不交给 Agent：原文方案里 Agent 只有查权限，退款到账状态由 `order_query` 读取，不由 Agent 触发

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 简单工单自动覆盖率 | ≥ 60%（原文实测口径） |
| 人工复核准确率 | ≈ 90%（原文实测口径） |
| 人工处理时长 | 8 分钟 → 40 秒（同系列实践中给出的对比，样本未说明） |
| 引用可追溯率 | 100%（每条回复必须带来源链接，原文的硬性要求） |
| 单任务成本 | 原方案未涉及 |
| 平均步数 | 原方案未涉及 |

**English**:
**① Scope** — A support agent inside the ticketing system: the clerk types the customer's question, the agent classifies it, calls the right tool to fetch live data or search the knowledge base, and produces a draft reply with source links plus a ticket category and suggested owner. In: raw question, ticket context (ticket ID, customer ID), session history. Out: structured result — reply body, category, suggested owner, citation links. Does NOT send replies (a human confirms and clicks send), does NOT perform writes such as refunds (read-only status checks), does NOT approve routing, and does NOT answer beyond retrieved material — the prompt forbids introducing information outside the retrieved context.
**② Tools** — `order_query` (live order/refund status), `ticket_history_search` (this customer's prior tickets, top 5), `knowledge_search` (product manuals, standard scripts, high-quality past tickets; top 5 chunks with scores), `customer_info_query` (tier, VIP flag, SLA), and an internal `intent_classify`.
**③ Orchestration** — `intent_classify` first; status-type questions go to `order_query`, knowledge-type to `knowledge_search`, mixed types to both. Results are written back into context, then the model produces structured output. The two paths are serial, not mutually exclusive: status questions fetch live data first, knowledge questions retrieve first, and both results feed the model. Finally the draft, category, owner, and citations are written back to the ticket for a human to confirm.
**④ Config** — `chunk_size=512`, `chunk_overlap=64`, `embedding_model="bge-m3"` (self-hosted), `retrieval_top_k=5`, `temperature=0.2` (derived: support needs reproducibility; the original only states the principle "don't let the model improvise"), `max_steps=6` (derived), `timeout=10s` (derived; not given in the original), `max_retries=2` (derived), `context_budget=60000` tokens (derived).
**⑤ Failure handling** — Low classification confidence → route to human with the full original question rather than guessing. Tool timeout → return a readable error to the model so it can retry or switch approach; never return empty silently, because an empty result gets read as "no refund record," the most dangerous failure mode. Empty retrieval → state explicitly that nothing matched; the model may not fall back on its own knowledge. Missing order ID → ask, never infer. All writes stay outside the agent — the original design gives the agent read-only access.
**⑥ Acceptance** — Simple-ticket auto-coverage ≥ 60% and ≈ 90% accuracy on human review (both from the original's measurements); handling time 8 minutes → 40 seconds (from a companion practice report, sample unspecified); citation traceability 100% (the original's hard requirement). Cost per task and average steps: not addressed.

来源：[大模型落地售后工单系统：从模型部署、微调到 Agent 编排的完整实践](http://www.sheratonhq.com/news/112907)

---

### 2. Agent-Native 工单闭环：五大部件 + 硬性刹车，token 降四成 【客服工单·Agent-Native 架构实践·LangGraph 风格】 📈

**中文**：

**① 场景与边界**
- 干什么：用户提交工单号后，Agent 自动查详情、按需检索知识库、生成回复草稿、必要时在 CRM 建任务，最后输出处理纪要 + 引用来源 + 任务 ID，人工扫一眼即可发送
- 输入：工单号、用户诉求文本
- 输出：`{处理纪要, 引用来源[], 创建的任务ID?}`，以及是否需人工复核的标记
- 不干什么：**不自动发送回复**、**不擅自建超过 1 个 CRM 任务**、**不处理跨工单批量操作**、**不做审批权限判断**（高风险写操作必须人工确认）

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `ticket_lookup` | 拿工单详情 | `{ticket_id}` | `{subject, body, status, customer_id, created_at}` |
| `knowledge_search` | 知识库检索 | `{query, top_k}` | chunk 列表 + 分数 |
| `create_crm_task` | 在 CRM 建跟进任务（**写操作**） | `{ticket_id, owner, due_date, note}` | `{task_id}` |
| `write_ticket_log` | 写工单日志（**写操作**，低风险） | `{ticket_id, content}` | `{ok}` |

**③ 编排流程**

```
用户提交工单号
  → ticket_lookup 拿详情
  → 判断是否需要知识库检索
       ├─ 需要 → knowledge_search → 
       └─ 不需要 →
  → 生成回复草稿
  → 判断是否必须开 CRM 任务
       ├─ 是 → create_crm_task【人工审批】→
       └─ 否 →
  → 输出结构化结果 {纪要, 引用, 任务ID}
  → 人工快速扫一眼 → 发送
```

原文的对比数字：这套闭环在 Agent 之前要人工切四个系统、耗时 5 分钟，现在人工时间被压到 20 秒左右。

**④ 关键配置**

```
max_steps                = 8      # 正常工单 4~6 步足够，超出基本是死循环
context_token_budget     = 60000  # 留推理空间，避免上下文过载导致质量下降
tool_timeout             = 10     # 秒，外部系统正常响应都在 5 秒内，超时基本是异常
same_tool_max_retries    = 2      # 超过 2 次说明方案有问题，需换策略
write_op_approval        = true   # 高风险写操作必须人工确认
irreversible_op_in_loop  = true   # 不可逆操作必须留人在回路
long_term_memory_write   = "on_task_complete"  # 每次任务完成后触发，趁热打铁
```

原文强调这些数字"不是拍脑袋拍的，是通过观测真实任务分布定出来的"，建议团队先花一周跑一版带完整日志的任务，统计"正常任务需要几步""工具耗时分布""上下文增长曲线"，再据此调参。

**⑤ 失败与兜底**
- **死循环**（最高频问题）→ 组合拳三层：① 硬性步数上限 `max_steps=8`，超了终止转人工；② 重复动作检测——连续几轮调同一工具且前次结果没带来新信息，强制结束当前策略、要求换思路；③ 进步度量——每个任务定义明确进展标记（"已获取工单详情""已生成回复草稿"），走三步进展没变化直接拉闸
- 外部系统 5 秒内无响应 → 按 `tool_timeout=10` 判超时，返回可读错误，让 Agent 决定重试或改道
- 同一工具连续失败 2 次 → 不磕第三次，转人工并附上已获取的全部上下文
- 需要写操作 → 阻断在审批节点，人工确认后才执行
- 效果：这套机制上线后"循环问题基本绝迹，顺带让 token 消耗降了四成"

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| token 消耗降幅 | -40%（原文实测口径，来源未说明观测周期） |
| 人工处理时间 | 5 分钟 → 20 秒（原文对比口径） |
| 死循环发生率 | 接近 0（"基本绝迹"，原文定性表述） |
| 任务完成率 / 首次解决率 / 单任务成本 | 原方案未涉及 |

**English**:
**① Scope** — A user submits a ticket ID; the agent fetches the ticket, retrieves from the knowledge base when needed, drafts a reply, creates a CRM task when necessary, and outputs a handling summary plus citations and task ID for a human to glance at and send. In: ticket ID, user's request text. Out: `{summary, citations[], task_id?}` plus a review-required flag. Does NOT auto-send, does NOT create more than one CRM task, does NOT batch across tickets, and does NOT judge approval authority — high-risk writes require human confirmation.
**② Tools** — `ticket_lookup` (read), `knowledge_search` (read), `create_crm_task` (write, gated), `write_ticket_log` (write, low risk).
**③ Orchestration** — Ticket lookup → decide whether knowledge retrieval is needed → draft reply → decide whether a CRM task is mandatory (gated by human approval) → emit structured result → human scans and sends. Per the original, the pre-agent process required switching across four systems over five minutes; human time now compresses to roughly twenty seconds.
**④ Config** — `max_steps=8` (4–6 is normal; beyond that it is almost certainly a loop), `context_token_budget=60000`, `tool_timeout=10s` (normal external responses land under 5s, so a timeout means something is wrong), `same_tool_max_retries=2` (a third attempt means the approach is wrong, not the parameters), `write_op_approval=true`, `irreversible_op_in_loop=true`, `long_term_memory_write="on_task_complete"`. The original insists these numbers came from observing real task distributions, and recommends a week of instrumented runs to measure steps-per-task, tool latency distribution, and context growth before tuning.
**⑤ Failure handling** — Loops are the most frequent problem, answered with three combined brakes: a hard step cap that terminates to human; repeated-action detection that forces a strategy change when consecutive calls to the same tool yield no new information; and a progress metric — each task defines explicit milestones, and three steps without a milestone change trips the breaker. Timeouts surface a readable error rather than a silent empty result. Two consecutive failures on the same tool escalate to human with all gathered context attached. Writes block at the approval node. Reported effect: loops essentially disappeared, and token consumption fell roughly 40%.
**⑥ Acceptance** — Token consumption -40% and human handling time 5 minutes → 20 seconds (both from the original, observation window unspecified); loop rate near zero (the original's qualitative claim). Task completion rate, first-contact resolution, and cost per task: not addressed.

来源：[Agent-Native 架构落地：五大核心部件与避坑指南](https://kwkr.cn/news/484346)

---

### 3. 销售异常快报：把分析师一小时的活拆成五节点固定工作流 【数据运营·Coze 工作流实践·Coze + SQL/HTTP 插件】

**中文**：

**① 场景与边界**
- 干什么：每周日晚 8 点自动触发，拉本周与上周的销售/订单/退款数据，找出波动超阈值的品类与区域，对前三异常点下钻到 SKU 与渠道，生成"结论 + 证据 + 图表"的报告并推送到钉钉/企微群
- 输入：时间范围（默认本周 vs 上周）、区域/品类维度、波动阈值
- 输出：Markdown 报告正文（异常描述、证据数据、可能业务解释、建议行动）+ 图表
- 不干什么：**不对"为什么"下最终结论**（只给"可能的业务解释"并标注为待验证）、**不自动改价/补货/调投放**、**不回答观点型问题**（如"这个季度目标能完成吗"——原文明确要求"谨慎作答"，看是否克制）、**不接管明细导出**（明细走独立工具，不塞进对话上下文）

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `metric_query` | 指标查询（已做成 API 服务的优先） | `{metric, region, period}` | 数值 + 同环比 |
| `trend_query` | 时间序列 | `{metric, region, granularity}` | 序列数组 |
| `detail_query` | 下钻到 SKU/渠道明细 | `{dimension, filters, top_n}` | 明细行 |
| `anomaly_detect`（内部节点） | 同环比超阈值判定 | 本期/上期数值 | `{is_anomaly, delta_pct}` |
| `report_render`（内部节点） | 组织成"结论-证据-图表" | 结构化发现 | Markdown + 图表配置 |
| 推送插件 | 发到钉钉/企微群 | Markdown 卡片 | `{ok}` |

**③ 编排流程（固定五节点，不允许模型自由发挥）**

```
[1] 参数抽取   ← 从触发参数/对话提取时间范围、区域、指标名
[2] 指标查询   ← 调工具拿实际数据（本周、上周、退款）
[3] 异常判断   ← 对比历史均值，判断是否显著波动
[4] 归因分析   ← 【只有存在显著波动时才调用归因工具】
[5] 结果渲染   ← 结论 / 证据 / 图表三段式，推送
```

原文的设计理由讲得很实在：编排成固定工作流有两个好处——模型不容易跑偏，且每一步可以单独测试和优化，出错时能立刻定位是参数抽取坏了还是数据查询慢了，而不是对着整段对话日志发愁。

**④ 关键配置**

```
schedule           = "周日 20:00"      # 原文
anomaly_threshold  = 0.30              # 推导：同环比波动 30% 视为异常，零售业常用经验阈值
top_n_anomalies    = 3                 # 原文：对"前三个异常点"下钻
top_n_details      = 20                # 推导：原文提到"前 20 名客户明细"，沿用该量级
temperature        = 0.3               # 推导：报告要稳定复现，但允许措辞变化
max_steps          = 5                 # 原文：五节点固定，无循环
tool_timeout       = 15                # 秒，推导：SQL 类查询比 API 慢
max_retries        = 2                 # 推导
context_budget     = 40000             # token，推导：明细必须裁剪后再进上下文（原文在 Dify 案例里强调过这点）
```

**⑤ 失败与兜底**
- **不要把所有原始数据丢给 LLM**——原文在同类实践里踩过的坑：API 返回里一堆创建时间、修改时间、自定义字段全塞进上下文，既浪费 token 又干扰判断。做法是在 HTTP/SQL 节点后加一个代码节点，只保留必要字段再转成简洁表格
- 指标 API 未就绪 → 可退而用 SQL 节点直查数据表，但原文明确不建议在生产环境这么做（会牺牲权限控制能力）
- 异常判定无显著波动 → 跳过归因节点，直接输出"本周无显著异常"，不硬凑结论
- 知识库/指标口径召回不到 → 先查分段逻辑（原文案例：长段落把关键日期切到了相邻 chunk），不要先怀疑模型
- 推送失败 → 不阻塞报告生成，记录告警，报告留档可手动补发

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 分析师人工耗时 | 1 小时 → 0（周报全自动） |
| 异常识别准确率 | 推算值：≥ 80%（人工回看上周报告，确认至少 8 成异常真实存在） |
| 单次运行成本 | 推算值：< $0.5（5 次工具调用 + 2 次大模型生成） |
| 误报率 | 推算值：≤ 30%（阈值 30% 下的合理区间） |
| 任务完成率 / 平均步数 | 原方案未涉及 |

**English**:
**① Scope** — Triggered Sundays at 20:00: pull this week's and last week's sales, order, and refund data, find categories and regions whose swing exceeds threshold, drill into the top three anomalies by SKU and channel, then generate a report of conclusions, evidence, and charts pushed to DingTalk or WeCom. In: period, region/category dimensions, threshold. Out: Markdown report with anomaly description, supporting data, candidate business explanations, and recommended actions. Does NOT conclude on causation (candidate explanations stay labelled as unverified), does NOT change prices or inventory or ad spend, does NOT answer opinion questions such as "will we hit the quarterly target" — the original explicitly asks whether the agent is restrained on these, and does NOT route detail exports through the conversation context.
**② Tools** — `metric_query`, `trend_query`, `detail_query`, plus internal `anomaly_detect` and `report_render` nodes and a push plugin.
**③ Orchestration** — Five fixed nodes: parameter extraction → metric query → anomaly detection (compare against historical mean) → attribution (invoked only when a significant swing exists) → rendering into conclusion/evidence/chart and push. The stated rationale: a fixed workflow keeps the model from drifting and lets each step be tested and optimized in isolation, so a failure points straight at parameter extraction or a slow query instead of leaving you staring at a whole conversation log.
**④ Config** — `schedule="Sunday 20:00"` and `top_n_anomalies=3` from the original; `anomaly_threshold=0.30`, `top_n_details=20`, `temperature=0.3`, `max_steps=5`, `tool_timeout=15s`, `max_retries=2`, `context_budget=40000` derived.
**⑤ Failure handling** — Do not dump raw API payloads into context: the same practitioners hit this, where created-at timestamps, modified-at values, and custom fields bloated the prompt and confused judgment, fixed by a code node that keeps only necessary fields as a compact table. If the metrics API is not ready, a SQL node against the tables is a fallback, though the original advises against it in production because it sacrifices permission control. No significant swing → skip attribution and state plainly that nothing anomalous happened. Retrieval misses → check chunking first, not the model. Push failures → alert and archive, never block report generation.
**⑥ Acceptance** — Analyst time 1 hour → 0 (fully automated weekly report) from the original; accuracy ≥ 80%, cost < $0.5 per run, and false-positive rate ≤ 30% all derived. Completion rate and average steps: not addressed.

来源：[数据智能体不是会聊天的仪表盘：从误区到落地实战](https://kwkr.cn/news/398959)

---

### 4. 订单风控状态机：条件边 + `interrupt` 人工审核 + 通知不阻塞 【订单风控·LangGraph 状态机·LangGraph】 【参考架构】

**中文**：

> **骨架来源**：LangGraph 官方 recommended pattern（状态机建模 + 条件边 + `interrupt` 断点恢复）的公开实践记录。**我补了第④段（原文只给了"3 次退避重试、30 秒超时"两条）与第⑥段（原文未给任何量化指标）；第①⑤段的边界与兜底也做了补全。**

**① 场景与边界**
- 干什么：用户提交订单后，自动判断是否需要人工审核，需要则进人工、不需要则自动通过，最后通知用户
- 输入：`{order_id, amount, user_risk_score}`
- 输出：`{need_review, review_decision, notification_sent}`，并落成一次可追溯的审核记录
- 不干什么：**不修改订单金额**（风控只判定是否放行）、**不做反欺诈模型训练**（`user_risk_score` 由外部系统给）、**不在人工拒绝时自动重试**（拒绝即终态）、**不负责通知内容的多语言与文案优化**

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `risk_eval`（节点） | 模型评估风险等级 | `{amount, user_risk_score, order_id}` | `{risk_level: low/medium/high}` |
| `auto_approve`（节点） | 自动通过 | 订单 | `{need_review: false}` |
| `human_review`（节点） | `interrupt` 暂停等人工 | 订单 + 风险等级 | `{review_decision, reject_reason?}` |
| `notify`（节点） | 通知用户 | 订单 + 决策 | `{notification_sent}` |
| 订单库 / 审核记录表 | 状态持久化 | — | — |

**③ 编排流程**

```
START → risk_eval
          ├─ low ──────────────> auto_approve ──┐
          └─ medium / high ──> human_review ────┤   （interrupt 暂停，等人工）
                                                 ▼
                                              notify → END
```

原文口径：整个工作流 15 个节点、5 个条件边；上线第 3 天，运维就能通过框架自带的事件流看出每个订单走到哪一步、卡在哪一步。

**④ 关键配置**

```
retry_max        = 3        # 原文：risk_eval 模型调用配 3 次退避重试
timeout          = 30       # 秒，原文
checkpointer     = Postgres # 推导：原文用"框架自带事件流"，生产须换持久化 checkpointer
interrupt_before = ["human_review"]              # 推导：这是 interrupt 机制的标准用法
recursion_limit  = 25                            # 推导：原文 15 节点、5 条件边，取节点数 1.5 倍余量
temperature      = 0.1                           # 推导：风控判定要可复现
context_budget   = 8000                          # token，推导：状态仅 7 个字段，无需大上下文
sla_expire_hours = 24                            # 推导：原文未给；审核超时应自动过期而非永久挂起
```

**⑤ 失败与兜底**
- 模型调用失败 → 退避重试 3 次后仍失败，降级为 `risk_level=high` 强制进人工（**绝不能在评估失败时自动放行**，这是风控系统的红线）
- 人工长时间不响应 → 超过 `sla_expire_hours` 标记 `EXPIRED` 并告警；状态机里必须有这个终态，否则订单永久挂起
- `notify` 失败 → **不阻塞整体流程**，记录告警后流程仍完成，由运维侧补偿（原文的明确设计）
- `human_review` 重放 → `interrupt` 恢复时会重跑整个节点函数，所以该函数必须无副作用或幂等；耗时/写库操作一律放在 `interrupt()` 之后
- 人工拒绝 → 附拒绝原因写回状态，不做二次自动重试
- 重复 resume → 审核记录表要独立于图状态，用 `thread_id` 唯一约束防双写

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 自动通过占比 | 推算值：40–70%（视 `user_risk_score` 分布，需按自身历史数据定） |
| 人工误放行率 | 推算值：≤ 1%（风控红线指标，抽检口径） |
| 端到端 P95 延迟（自动通过链路） | 推算值：< 5 秒 |
| 审核 SLA 内完成率 | 推算值：≥ 95% |
| 通知失败补偿率 | 推算值：100%（失败必进补偿队列） |
| 原文实测指标 | 原方案未涉及 |

**English**:
> **Skeleton source**: public practice write-ups of the LangGraph recommended pattern (state-machine modelling, conditional edges, `interrupt` with checkpoint recovery). **I filled in ④ (the original gives only "3 backoff retries, 30s timeout") and ⑥ (no metrics at all in the original), and completed ① and ⑤ as well.**

**① Scope** — When an order is submitted, decide automatically whether human review is required; if so, route to a human, otherwise auto-approve, then notify the customer. In: `{order_id, amount, user_risk_score}`. Out: `{need_review, review_decision, notification_sent}` plus an auditable review record. Does NOT modify order amounts (risk control only gates), does NOT train fraud models (`user_risk_score` arrives from an external system), does NOT retry after a human rejection (rejection is terminal), and does NOT localise notification copy.
**② Tools** — Nodes `risk_eval`, `auto_approve`, `human_review` (interrupt), `notify`, plus the order store and review-record table.
**③ Orchestration** — `START → risk_eval`; low risk → `auto_approve`; medium/high → `human_review` (pause on interrupt); both converge on `notify → END`. The original reports 15 nodes and 5 conditional edges, and that by day three in production ops could read the framework's event stream to see exactly which step any order was on.
**④ Config** — `retry_max=3` and `timeout=30s` from the original; `checkpointer=Postgres`, `interrupt_before=["human_review"]`, `recursion_limit=25`, `temperature=0.1`, `context_budget=8000`, `sla_expire_hours=24` derived.
**⑤ Failure handling** — After three backoff retries, a failing model call degrades to `risk_level=high` and forces human review; auto-approving on evaluation failure is the one thing a risk system must never do. A human who never responds hits `EXPIRED` at the SLA and raises an alert — the state machine needs that terminal state or orders hang forever. A `notify` failure does not block the flow (the original's explicit design); alert and compensate. When `interrupt` resumes, the whole node function re-runs, so it must be side-effect free or idempotent and any expensive or write operation belongs after `interrupt()`. Rejections write back with a reason and are never auto-retried. Duplicate resumes are prevented by keeping the review table independent of graph state, with a unique constraint on `thread_id`.
**⑥ Acceptance** — Auto-approve share 40–70%, human-missed-fraud rate ≤ 1%, P95 auto-approve latency < 5s, review-within-SLA ≥ 95%, notification compensation 100% — all derived, to be calibrated against your own historical distribution. The original reports no metrics.

来源：[LangGraph 智能体编排实战：状态机建模与工程化落地的完整方法论](https://mukebb.blog.csdn.net/article/details/167349190)

---

# 第二部分：国外 / International

### 1. 供应商付款 Agent：`interrupt` 断点 + 独立审批表 + 幂等打款 【财务付款·LangGraph.js + PostgreSQL + Next.js·LangGraph.js】

**中文**：

**① 场景与边界**
- 干什么：发票进入（上传或 webhook）后，Agent 抽取行项目、对 PO 做校验、跑欺诈启发式，起草一笔付款（金额、币种、收款方、通道：美国走 Stripe、印度走 RazorpayX）。低于自动审批阈值且风险低就直接付，否则暂停等财务审批人
- 输入：发票文本/文件、PO 记录、供应商主数据
- 输出：`{payoutResult}`（含支付通道流水号）或一条 `APPROVED/REJECTED/EDITED` 的审批记录
- 不干什么：**不修改发票金额本身**（审批人只能编辑待付金额）、**不跨币种做汇率换算的最终定价**（通道决定）、**不在审批前预扣款**、**不处理供应商银行账户变更**（属主数据流程，隔离在 Agent 之外）

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `draftPayoutWithLLM` | 抽取行项目、生成付款草稿 | `invoiceText` | `PayoutDraft{requestId, vendorId, amountMinor, currency, rail, riskScore, reasoning}` |
| `executePayout` | 实际打款（**幂等键必填**） | 草稿 + 最终金额 | `{payoutId, status}` |
| Stripe / RazorpayX | 支付通道 | 按币种路由 | 流水 |
| `approval_requests`（表） | 审批业务生命周期 | `thread_id, org_id, payload` | 行记录 |
| Postgres checkpointer | 图状态持久化 | `thread_id` | 快照 |

**③ 编排流程**

```
invoice（上传/webhook）
  → analyzeInvoice：抽行项目 + 对 PO 校验 + 欺诈启发式 + 起草付款
  → routeAfterAnalysis
       ├─ amount < AUTO_APPROVE_LIMIT 且 riskScore 低 → executeNode
       └─ 否则 → humanApproval（interrupt，图暂停、状态落库）
                   → 审批人 approve / reject / edit
                   → executeNode（用最终金额，带幂等键）
  → 记录结果
```

关键工程点：金额一律用**最小货币单位**（美分/派士）存 `amountMinor`，避免浮点。

**④ 关键配置**

```
AUTO_APPROVE_LIMIT_MINOR = 5_000    # 原文：$50 / ₹50 等价
checkpointer             = Postgres # 原文：生产不可用 MemorySaver
interrupt_before         = ["humanApproval"]  # 原文
idempotency_key          = true     # 原文：executor 必带
reducer                  = "last_write_wins"  # 原文：Annotation.Root 显式 reducer，避免并发写同键时意外
recursion_limit          = 20       # 推导：4 节点图，留足余量
sla_expire_hours         = 48       # 推导：原文给了 EXPIRED 终态但没给时长
```

审批状态机（原文明确列出，且要求与图状态分离）：

```
PENDING → APPROVED / REJECTED / EDITED → EXECUTED / FAILED
        └→ EXPIRED（SLA 内无人响应）
```

**⑤ 失败与兜底**
- `interrupt()` 恢复时整个节点函数会重跑 → 原文的硬性要求是：`humanApproval` 函数里 `interrupt()` 之前**不能有任何副作用**，只读状态然后返回
- 重复 resume → `approval_requests` 表用 `thread_id` 唯一约束，图状态和业务生命周期分离，这是"可查询、可审计、防双恢复"的关键
- 打款重复 → `executePayout` 必须带幂等键
- 审批超时 → `EXPIRED` 终态，不永久挂起
- 审批人改金额 → 走 `EDITED` 分支，最终金额以 `decision.amountMinor` 为准，覆盖草稿值
- 高风险或低置信 → 无条件进人工，阈值只降不放

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 自动审批占比 | 推算值：60–80%（小额低风险为主，视发票金额分布） |
| 重复打款率 | 推算值：0（幂等键 + 唯一约束双保险） |
| 审批 SLA 内完成率 | 推算值：≥ 90% |
| 单笔处理成本 | 推算值：< $0.05（1 次 LLM 起草 + 1 次打款 API） |
| 人工介入率 | 推算值：20–40% |
| 原文实测指标 | 原方案未涉及 |

**English**:
**① Scope** — An invoice arrives (upload or webhook); the agent extracts line items, validates against the PO, runs fraud heuristics, and drafts a payout with amount, currency, beneficiary, and rail (Stripe for USD, RazorpayX for INR). Under the auto-approve threshold with low risk, it pays; otherwise it pauses for a finance approver. In: invoice text, PO record, vendor master data. Out: `{payoutResult}` with the rail's transaction ID, or an `APPROVED/REJECTED/EDITED` approval record. Does NOT alter the invoice amount (approvers edit only the payout amount), does NOT do final FX pricing (the rail decides), does NOT pre-debit before approval, and does NOT handle vendor bank-account changes, which stay in a master-data process outside the agent.
**② Tools** — `draftPayoutWithLLM` producing a `PayoutDraft`, `executePayout` (idempotency key mandatory), the Stripe/RazorpayX rails, an `approval_requests` table, and a Postgres checkpointer.
**③ Orchestration** — Invoice → analyze (extract, validate against PO, fraud heuristics, draft) → route: under `AUTO_APPROVE_LIMIT` with low risk goes straight to execute; otherwise `humanApproval` pauses the graph via interrupt and persists state, the approver approves, rejects, or edits, then execute runs with the final amount and an idempotency key. Money is always stored in minor units (`amountMinor`) to avoid floats.
**④ Config** — `AUTO_APPROVE_LIMIT_MINOR=5000` ($50 / ₹50 equivalent), Postgres checkpointer instead of MemorySaver, `interrupt_before=["humanApproval"]`, idempotency key on execute, explicit `Annotation.Root` reducers so concurrent writes to the same key cannot surprise you. All from the original. `recursion_limit=20` and `sla_expire_hours=48` derived — the original defines the `EXPIRED` terminal state but gives no duration. The approval lifecycle (`PENDING → APPROVED/REJECTED/EDITED → EXECUTED/FAILED`, plus `EXPIRED`) is kept deliberately separate from graph state.
**⑤ Failure handling** — LangGraph replays the node on resume and feeds the stored value into `interrupt()`, so everything before it executes twice; the hard rule is that `humanApproval` must be pure. Duplicate resumes are blocked by a unique `thread_id` in `approval_requests`, which is what makes the system queryable, auditable, and safe from double resumes. Duplicate payouts are blocked by the idempotency key, giving two independent guards. Approval timeouts land on `EXPIRED` rather than hanging. An edited amount takes `decision.amountMinor` over the drafted value. Low confidence or high risk always routes to human — thresholds only ever tighten.
**⑥ Acceptance** — Auto-approve share 60–80%, duplicate payout rate 0, approval within SLA ≥ 90%, cost per invoice < $0.05, human involvement 20–40% — all derived. The original reports no metrics.

来源：[Engineering Durable Human-in-the-Loop (HITL) AI Workflows: State Checkpointing with LangGraph, PostgreSQL, and Next.js](https://nirajiitr.com/blog/durable-hitl-ai-workflows-langgraph-postgresql-nextjs)

---

### 2. 生产级多 Agent 客服：Supervisor 路由 + 读写工具隔离 + 置信度闸门 【客服分流·LangGraph + FastAPI·LangGraph】

**中文**：

**① 场景与边界**
- 干什么：工单进来后分类（订单状态/退款/技术/投诉），按类别调对应工具取数据或检索政策，生成回复草稿，低置信度或退款类一律转人工审批
- 输入：工单文本
- 输出：`{intent, confidence, reply}` 或人工审核请求
- 不干什么：**Agent 只能读数据**，写操作（退款）必须走**一个受控函数且审批后才执行**、**不允许 Agent 之间自由对话**（原文明确：agent 互相对话会产生循环，supervisor 把控制权收在一处）、**不做跨工单的批量退款**

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `classifier_agent`（小模型 + 结构化输出） | 分类 | 工单文本 | `Intent{intent, confidence}` |
| `data_agent` | 查订单（**只读**） | `order_id` | `{order_data}` |
| `policy_agent` | 政策库检索 | 工单文本 | 政策文本（top 3） |
| `response_agent` | 生成回复 | 上下文 | `{reply}` |
| `human_review` | 人工闸门 | 草稿 | 审批/编辑 |
| `refund`（受控写函数） | 退款执行 | 审批后的参数 | 流水 |

**③ 编排流程**

```
START → classifier
          ├─ intent ∈ {order_status, refund} → data_agent → policy_agent → response_agent
          └─ 其他 ─────────────────────────────────────────→ response_agent
                                                                  ▼
                                          needs_review?
                                            ├─ confidence < 0.75 或 intent == "refund" → human_review → END
                                            └─ 否则 → END
```

原文的设计取舍：选 supervisor 模式而不是让 agent 自由对话——"agents talking freely to each other create loops. A supervisor keeps control in one place." 另一条是**读写工具分离**：Agent 只能读，写操作像退款走一个受控函数且必须审批。

**④ 关键配置**

```
confidence_threshold = 0.75     # 原文：低于此值转人工
interrupt_before     = ["human_review"]   # 原文
checkpointer         = "Postgres-backed"  # 原文：生产替换 MemorySaver
policy_top_k         = 3        # 原文：similarity_search(k=3)
model_split          = "big for supervisor, small for classification"  # 原文：分类不需要最贵的模型
queue                = "Redis"  # 原文：FastAPI + Redis queue 扛工单峰值
tracing              = "from day one"     # 原文：LangSmith 或任意 tracing
recursion_limit      = 20       # 推导
timeout              = 10       # 秒，推导
```

**⑤ 失败与兜底**
- **Agent 互相踢皮球**：demo 里没人拦，生产上 data_agent 会不停让 classifier "再确认一下意图"，classifier 又踢回来，一张工单烧掉几十次 LLM 调用 → 解法就是 supervisor 单点控制 + 条件边，不给环形边
- 分类置信度低 → 不走猜测路径，直接 `human_review`
- 退款类 → 无论置信度多高都进人工（原文的硬规则）
- 回复错误 → "从第一天就开 tracing"，出问题打开 trace 就能看到哪个 agent 做了错误决策，这一条习惯省下的调试时间比什么都多
- 生产环境的 MemorySaver → 必须换 Postgres，否则重启丢状态

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 分类准确率 | 推算值：≥ 92%（小模型 + 结构化输出，4 类任务） |
| 人工介入率 | 推算值：25–40%（受 0.75 置信度阈值与退款全覆盖影响） |
| 单工单 LLM 调用次数 | 推算值：≤ 4（supervisor 路径 + 3 次工具/生成） |
| 首次解决率 | 推算值：≥ 55% |
| 原文实测指标 | 原方案未涉及（原文明确说明文中数字为示意值，非审计结果） |

**English**:
**① Scope** — A ticket arrives, gets classified (order status / refund / technical / complaint), routes to the tools or policy store for its category, and produces a draft reply; low confidence or any refund goes to human approval regardless. In: ticket text. Out: `{intent, confidence, reply}` or a review request. Agents are read-only; writes such as refunds run through one controlled, post-approval function; agents may not talk to each other freely, because the original notes that free agent-to-agent chat creates loops and a supervisor keeps control in one place; no batch refunds across tickets.
**② Tools** — `classifier_agent` (small model with structured output), `data_agent` (read-only order lookup), `policy_agent` (top-3 policy retrieval), `response_agent`, `human_review`, and a controlled `refund` write function.
**③ Orchestration** — `START → classifier`; order-status and refund intents go through `data_agent → policy_agent → response_agent`, everything else goes straight to `response_agent`; then a review gate checks `confidence < 0.75` or `intent == "refund"` and routes to `human_review`. The two architectural choices the original names are the supervisor pattern over free-for-all agent chat, and strict read/write separation with approval gating the write path.
**④ Config** — `confidence_threshold=0.75`, `interrupt_before=["human_review"]`, Postgres-backed checkpointer in production, `policy_top_k=3`, a large model for the supervisor and a small one for classification, FastAPI plus a Redis queue to absorb ticket bursts, and tracing switched on from day one. `recursion_limit=20` and `timeout=10s` derived.
**⑤ Failure handling** — The demo works on day one; production teaches the rest. The loop failure is concrete: with free agent-to-agent calls, a confused data agent keeps asking the classifier to "re-check" the intent and the classifier sends it back, burning dozens of LLM calls on one ticket — solved by single-point supervisor control and conditional edges with no cycles. Low confidence never guesses; it goes to human. Refunds go to human regardless of confidence. Wrong replies are diagnosed by opening the trace from day one, a habit the original says saves more debugging hours than anything else. MemorySaver must be swapped for Postgres or state vanishes on restart.
**⑥ Acceptance** — Classification accuracy ≥ 92%, human involvement 25–40%, ≤ 4 LLM calls per ticket, first-contact resolution ≥ 55% — all derived. The original explicitly labels its figures as illustrative, not audited.

来源：[How to Build a Production-Ready Multi-Agent AI System: A Practical Guide for 2026](https://logicproviders.ca/blog/production-ready-multi-agent-ai-system-guide-2026)

---

### 3. Orchestrator-Workers：给每个子任务写明"不许碰什么" 【通用多步任务·Anthropic 五种工作流模式·参考架构】 【参考架构】

**中文**：

> **骨架来源**：Anthropic「Building effective agents」中的 orchestrator-workers 模式，及 2026 年 10 月一篇工程实践对其的生产化改造。**我补了第④段的全部参数（原文代码里有 `MAX_SUBTASKS=5`，其余无值）、第⑥段（原文无指标），并补全第⑤段的兜底路径。**

**① 场景与边界**
- 干什么：面对**子任务不可预知**的任务，由 orchestrator 当场决定拆成哪几个子任务、分给哪些 worker，收齐结果后合成一个答案
- 输入：一个复合任务描述 + 一张 worker 注册表
- 输出：合成后的单一答案；若有 worker 失败，答案里必须显式说明
- 不干什么：**不用于子任务可预知的场景**（那种情况直接用 parallelization 分区，别引入动态拆解的不确定性）、**不允许 orchestrator 输出超过上限的子任务数**、**不允许 worker 之间对话**、**不允许 worker 定义里不写边界**

**② 工具清单**

| 工具 | 作用 | 输入 | 输出 |
|---|---|---|---|
| `orchestrator`（LLM 调用） | 把任务拆成 ≤5 个独立子任务 | 任务描述 | JSON 列表 `[{worker, objective, boundaries}]` |
| `workers[worker]` | 各领域 worker | `{worker, objective, boundaries}` | 结果字符串或异常 |
| `synthesize`（LLM 调用） | 合成最终答案 | 成功结果列表 + 失败计数 | 最终答案 |

**③ 编排流程**

```
task
  → orchestrator 出计划（JSON，≤5 个子任务，每个带 objective + boundaries）
  → 计划校验：过滤掉未注册的 worker，截断到 MAX_SUBTASKS
      ├─ 计划为空 → 抛错（不允许"没有子任务"就往下走）
      └─ 否则 →
  → asyncio.gather(并行执行，return_exceptions=True)
  → 分流：ok = 非异常结果
  → 若有失败，在合成提示里追加 "(NOTE: N subtask(s) failed; say so in the answer.)"
  → synthesize → 答案
```

原文提到的两个必须写进提示词的教训：**① 有实践者见过 orchestrator 三步能完的活出了 14 步计划；② Anthropic 研究团队发现模糊委派导致重复劳动，用"每个子任务写清目标 + 边界"修好了。**

**④ 关键配置**

```
MAX_SUBTASKS       = 5        # 原文
plan_format        = "JSON only, [{worker, objective, boundaries}]"  # 原文
boundaries_rule    = "必须写明这个子任务不许覆盖什么"               # 原文
return_exceptions  = True     # 原文：worker 失败不炸整批
failure_note       = "在合成时显式说明失败数量"                     # 原文
worker_timeout     = 120      # 秒，推导：单 worker 属长任务，需给足
orchestrator_temp  = 0.0      # 推导：拆解要可复现
synthesis_temp     = 0.3      # 推导：合成允许措辞灵活
max_total_steps    = MAX_SUBTASKS + 2   # 推导：拆解 1 步 + 并行 5 步 + 合成 1 步
plan_retry         = 1        # 推导：计划为空时重试 1 次，仍空则降级为单 worker 直跑
```

**⑤ 失败与兜底**
- 计划为空 / 全是未注册 worker → 重试 1 次，仍空则降级：跳过拆解，直接把原任务交给最匹配的单个 worker
- orchestrator 过度拆解（14 步干 3 步的活）→ `MAX_SUBTASKS` 硬截断 + 提示词里写死上限
- 委派模糊导致重复劳动 → `boundaries` 字段强制必填，且必须是否定式表述（"不许覆盖什么"），不是目标复述
- 某个 worker 抛异常 → `return_exceptions=True` 收住，不让整批炸；合成时显式声明"N 个子任务失败"
- worker 卡死 → `worker_timeout=120` 单独超时，超时视作异常结果处理
- 所有 worker 都失败 → 不合成，直接返回失败并附各 worker 的错误摘要

**⑥ 验收指标**

| 指标 | 目标值 |
|---|---|
| 子任务计划有效率 | 推算值：≥ 95%（校验后仍非空的比例） |
| 重复劳动率 | 推算值：≤ 10%（两个 worker 产出重叠 > 50% 计为重复） |
| 端到端 P95 延迟 | 推算值：< 150 秒（并行 5 个 worker） |
| 单任务成本 | 推算值：拆解 1 次 + 合成 1 次 + N 次 worker 调用；N=3 时约 5 次模型调用 |
| 显式失败声明率 | 推算值：100%（有 worker 失败时答案必须说明） |
| 原文实测指标 | 原方案未涉及 |

**English**:
> **Skeleton source**: the orchestrator-workers pattern from Anthropic's "Building effective agents," plus an October 2026 engineering write-up that productionised it. **I filled in all of ④ (the original code has only `MAX_SUBTASKS=5` and no other values) and ⑥ (no metrics in the original), and completed ⑤.**

**① Scope** — For tasks where the subtasks are not knowable in advance, the orchestrator decides on the spot which subtasks to create and which worker gets each, then synthesises the results into one answer. In: a composite task description plus a worker registry. Out: a single synthesised answer that must explicitly state when a worker failed. Does NOT apply where subtasks are predictable (use parallelization sectioning instead of importing dynamic-decomposition variance), does NOT let the plan exceed the subtask cap, does NOT allow worker-to-worker conversation, and does NOT accept worker definitions without boundaries.
**② Tools** — `orchestrator` (LLM call returning a JSON plan), a `workers` registry of domain workers, and `synthesize` (LLM call over successful results plus a failure count).
**③ Orchestration** — Task → orchestrator produces a plan of at most five subtasks, each with an objective and boundaries → the plan is validated: unregistered workers filtered out, capped at `MAX_SUBTASKS`, empty plan raises → parallel execution via `asyncio.gather(..., return_exceptions=True)` → successes collected → a failure note appended to the synthesis prompt → synthesised answer. Two lessons the original insists belong in the prompt: practitioners have seen an orchestrator emit a 14-step plan for a three-step job, and Anthropic's research team found vague delegation caused duplicated work, fixed by explicit objectives and explicit boundaries per subtask.
**④ Config** — `MAX_SUBTASKS=5`, JSON-only plan format, a mandatory boundary field stating what the subtask must NOT cover, `return_exceptions=True`, and a failure note on synthesis, all from the original. `worker_timeout=120s`, `orchestrator_temp=0.0`, `synthesis_temp=0.3`, `max_total_steps=MAX_SUBTASKS+2`, `plan_retry=1` derived.
**⑤ Failure handling** — Empty or fully invalid plan → one retry, then degrade by skipping decomposition and handing the task to the single best-matching worker. Over-decomposition → hard cap plus the cap written into the prompt. Vague delegation → the boundaries field is mandatory and must be phrased negatively (what it must not cover), not a restatement of the objective. A worker exception is caught by `return_exceptions=True` so the batch survives, and the synthesis must declare the failure count. A hung worker hits its own `worker_timeout` and is treated as an exception result. If every worker fails, skip synthesis and return the failure with each worker's error summary.
**⑥ Acceptance** — Plan validity ≥ 95%, duplicated-work rate ≤ 10% (overlap above 50% between two workers counts as duplicated), P95 end-to-end latency < 150s with five parallel workers, cost roughly five model calls at N=3, and 100% explicit failure disclosure — all derived. The original reports no metrics.

来源：[Design Patterns for Agentic Workloads: A 2026 Field Guide](https://orchestrator.dev/blog/2026-10-09-agentic-workload-design-patterns)

---

## 附：今天的选型判断

**中文**
1. **先判断你到底需不需要图。** 那篇 LangGraph 实战给了一条很干净的判断链：单轮问答/简单工具调用 → 原生 SDK 就够；多步但顺序固定 → 普通代码编排，图的收益不明显；**多分支 + 需人工介入 + 状态复杂 + 要断点恢复** → LangGraph 收益才显著。团队没有 Python 基础或要强约束低代码 → 考虑 Dify，但要注意深度定制受限。这条链比任何框架对比文章都实用。
2. **人工审核节点的最大坑不是逻辑，是重放。** LangGraph 的 `interrupt` 恢复时会**重跑整个节点函数**再把存下来的值喂给 `interrupt()`，所以 `interrupt()` 之前执行的任何东西都会执行两遍。这是两个方案里都被单独点出来的地方——凡是写库、发消息、扣钱，一律放到 `interrupt()` 之后。
3. **空结果比报错危险。** 工具超时如果静默返回空，模型会把它读成"没有退款记录""没有相关政策"，然后自信地给出错误答案。这几个方案里都要求把失败显式变成可读错误。
4. **读写工具必须分家。** 三个客服/财务方案都做了同一件事：Agent 只有读权限，写操作走一个受控函数，且必须过审批。这不是保守，是因为 Agent 的失败模式是"自信地做错"，而写操作没有撤销键。
5. **参数要观测出来，不要拍出来。** Agent-Native 那篇说得很直白：先花一周跑一版带完整日志的任务，统计正常任务需要几步、工具耗时分布、上下文增长曲线，再据此定 `max_steps` 和 `timeout`。把 Agent 当黑盒跑，永远养不出稳定的生产系统。

**English**
1. **Decide whether you need a graph at all.** The LangGraph write-up gives a clean decision chain: single-turn Q&A or simple tool calls → the native SDK is enough; multi-step but fixed order → plain code orchestration, the graph buys little; **multi-branch, human-in-the-loop, complex state, needs checkpoint recovery** → that is where LangGraph pays. Teams without Python depth or needing hard low-code constraints → Dify, accepting limited deep customisation. That chain is more useful than any framework comparison.
2. **The biggest trap in a human-approval node is not the logic, it is replay.** LangGraph replays the entire node function on resume and feeds the stored value into `interrupt()`, so anything before it runs twice. Both write-ups call this out separately — put every database write, message send, and money movement after `interrupt()`.
3. **An empty result is more dangerous than an error.** If a tool times out and silently returns nothing, the model reads it as "no refund record" or "no relevant policy" and answers confidently and wrongly. Every one of these designs requires failures to surface as readable errors.
4. **Read and write tools must be separated.** All three support and finance designs do the same thing: the agent is read-only, writes go through one controlled function behind approval. That is not conservatism — the agent's failure mode is being confidently wrong, and writes have no undo key.
5. **Measure parameters, don't guess them.** The Agent-Native piece is blunt about it: spend a week running instrumented tasks, measure steps per normal task, tool latency distribution, and context growth, then set `max_steps` and `timeout` from that. Treat an agent as a black box and you will never grow a stable production system.
