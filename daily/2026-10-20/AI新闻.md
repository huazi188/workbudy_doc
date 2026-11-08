# 2026-10-20 AI新闻

**总览**：今天 AI 圈最热的是**中国 AI 技术出口管制草案把「智能体」写进清单**，以及**英伟达核心团队出走创立 Dream Labs、以 9000 万美元种子轮和 Dream-1 通用机器人模型切入「机器人通用大脑」**。

**Overview**: Today's two biggest stories are China's draft export-control revision that explicitly names "AI agents," and the emergence of Dream Labs — founded by four core members of Nvidia's GR00T team with a $90M seed round and a general-purpose robot foundation model, Dream-1.

---

## 一、大模型与产品 / Models & Products

### 1. 🔥 Anthropic 神秘新模型曝光，可推断模糊指令的真实意图 / Leaked Anthropic Model Said to Infer Intent Behind Vague Prompts

**中文**：据试用者描述，Anthropic 正在测试一款能主动推断用户模糊指令真实意图的新模型，具备更强的推理与创造性问题解决能力。示例包括：给定不完整的编程任务时，模型会自行检查环境、定位问题并调整后续行动；询问投资时，模型会跳出常规指标给出新思路。目前这些能力描述仅来自一名试用者，无公开评测验证，模型所属家族也未知。凤凰网报道同时提到，Google 内部已在开发代号 Gemini 4 Carbon 的下一代模型。

**English**: Anthropic is reportedly testing a model that infers the real intent behind vague user instructions. Early testers describe it inspecting an environment, locating problems and adjusting its plan mid-task, or proposing unconventional approaches to an investment question. The claims come from a single tester with no public benchmarks, and it is unclear which Claude family the model belongs to. Google is separately said to be developing a next-generation model codenamed Gemini 4 Carbon.

来源：[凤凰网](https://h5.ifeng.com/c/vivoArticle/v002NhTEwLXJYoQVyrmz0hDjKnHc2uo1XzVNTMLJ2--3ker8__)

### 2. OpenAI GPT-6 Pro 上线并行 Agent，任务可拆分给多个子 Agent / GPT-6 Pro Ships Parallel Agents That Split Tasks Across Sub-Agents

**中文**：OpenAI 于 10 月 16 日推出 GPT-6 Pro 的并行 Agent 能力，ChatGPT 中的该模型可把单个任务拆分给多个同时工作的 Agent，用户能跟踪每个 Agent 的进度并审阅已完成的子任务。同月还发布了 GPT-6 Luna、GPT-6 Sol 等分层版本，其中 GPT-6.1 Sol 于 9 月 29 日 DevDay 发布，驱动 ChatGPT Work、Codex 与 API。

**English**: OpenAI released parallel agent capability for GPT-6 Pro on October 16, letting ChatGPT split a task across several agents working concurrently, with per-agent progress tracking and sub-task review. The company also fields GPT-6 Luna and GPT-6 Sol tiers, with GPT-6.1 Sol launched at DevDay on September 29 powering ChatGPT Work, Codex and the API.

来源：[Evertune Model Tracker](https://www.evertune.ai/resources/ai-model-tracker)

### 3. MiniMax M3.1 Flash 发布：推理快近 3 倍、成本降 70% / MiniMax Releases M3.1 Flash With ~3x Faster Inference and 70% Lower Cost

**中文**：MiniMax 发布新一代 Flash 模型 M3.1 Flash，在 Agentic 任务、编程、Computer Use 及网络安全能力上实现代际提升，线上推理速度提升近 3 倍，使用成本降低 70%。这是 LLM Gateway 追踪到的最近一次模型发布（10 月 19 日）。

**English**: MiniMax launched M3.1 Flash, claiming generational improvements in agentic tasks, coding, computer use and cybersecurity, with roughly 3x faster online inference and 70% lower cost. It is the most recent model release tracked by LLM Gateway, dated October 19.

来源：[LLM Gateway Timeline](https://llmgateway.io/timeline)

### 4. Meta 开源 Muse Spark Open，主打美国最强开源权重模型 / Meta Open-Sources Muse Spark Open, Claiming Strongest US Open-Weight Model

**中文**：Meta 于 10 月 15 日发布 Muse Spark Open，定位为面向编程与 Agent 任务的开源权重模型，Meta 称其为迄今最强的美国开源权重模型。权重可免费下载，并与 Apple 合作使其能在一对 Mac Studio 上本地运行。Meta 的主力 Muse Wonder 系列仍为闭源。

**English**: Meta released Muse Spark Open on October 15, an open-weight model for coding and agent work that Meta calls the most capable US open-weight model to date. Weights are free to download and, thanks to work with Apple, can run locally on a pair of Mac Studios. Meta's flagship Muse Wonder models remain closed.

来源：[Evertune Model Tracker](https://www.evertune.ai/resources/ai-model-tracker)

---

## 二、芯片与算力 / Chips & Compute

### 1. 🔥 英伟达统一 NVHBM 制程，推动三星与台积电联手 / Nvidia Standardizes NVHBM, Pushing Samsung and TSMC Together

**中文**：英伟达为下一代 AI 加速器定制 NVHBM，要求三星、SK 海力士、美光三家供应商的基底芯片全部采用台积电 3nm 工艺、核心存储芯片统一采用第六代「1c」DRAM 制程。此举使长期对手三星与台积电罕见联手开发基底芯片，也是三星首次在 HBM 业务中放弃全自主生产。NVHBM 基于 HBM4E 架构，带宽最高提升 30%、功耗最多降低 15%。三星仍在评估用自有 2nm 产线生产基底芯片的可行性，以防台积电产能不足。

**English**: Nvidia has mandated that all three NVHBM suppliers — Samsung, SK Hynix and Micron — use TSMC's 3nm process for base dies and a common sixth-generation "1c" DRAM process for core memory, eliminating the need to validate each vendor separately. The move drove Samsung, which had always made base dies in-house, into a rare joint development with TSMC, its foundry rival. Built on HBM4E, NVHBM lifts bandwidth by up to 30% and cuts power by up to 15%. Samsung is still evaluating a 2nm fallback in case TSMC capacity falls short.

来源：[网易/镁客网](https://www.163.com/dy/article/L9L4M2VA0511AGMA.html)

### 2. Anthropic 拟发 600 亿美元债务买 Google TPU，黑石或投 90 亿 / Anthropic Weighs $60B Debt Package to Buy Google TPUs

**中文**：据报道，Anthropic 正筹集约 600 亿美元债务用于购买 Google TPU，黑石可能在其中投入 90 亿美元。同期还有消息称 Nscale 正与 Anthropic 洽谈约 450 亿美元的云算力交易。若两项均落地，Anthropic 的算力采购规模将达到前所未有的量级。

**English**: Anthropic is reportedly raising a roughly $60 billion debt package to buy Google TPUs, with Blackstone potentially contributing $9 billion. Separately, Nscale is said to be negotiating a cloud compute deal with Anthropic worth about $45 billion. If both land, Anthropic's compute procurement would reach an unprecedented scale.

来源：[VectorWire Deal Flow](https://vectorwire.ai/dashboard/deals)

### 3. 光纤供需缺口 1.8 亿芯公里，AI 数据中心拉动价格涨 2.8-5 倍 / Fiber Shortage Hits 180 Million Core-Km as AI Data Centers Lift Prices 2.8-5x

**中文**：TrendForce 10 月 19 日发布研究，AI 数据中心、跨数据中心互连及高容量骨干网络建设推升光纤需求，预估 2026 年全球供需缺口达 1.8 亿芯公里，现货价格年增 2.8 至 5 倍。电信厂商已从短期采购转向长约锁定资源，短缺预计至少延续至 2028 年。

**English**: TrendForce said on October 19 that AI data centers, inter-DC links and high-capacity backbone builds will push the global fiber supply gap to 180 million core-kilometers in 2026, with spot prices up 2.8 to 5 times year over year. Carriers are shifting from spot buying to long-term contracts, and the shortage is expected to last at least through 2028.

来源：[腾讯新闻/每日经济新闻](https://news.qq.com/rain/a/20261020A02NFE00)

### 4. 蓝芯算力发布 RISC-V 智通融合超节点，LX500 单芯 75 TOPS / Lanxin Computing Launches RISC-V Compute-Fusion Super Node

**中文**：10 月 19 日 RISC-V 中国峰会上，蓝芯算力发布智通融合超节点，把 RISC-V 架构的 AI 计算从单芯片、服务器延伸至集群级。LX500 集成 48 个 RISC-V 核心（32 性能核 + 16 能效核），AI 算力 75 TOPS INT8，支持 INT4 至 FP32 多精度，采用统一内存架构共享 DDR5 并支持 CXL 扩展，已适配 Qwen、DeepSeek、Kimi 等模型，面向私有知识库 RAG、Agent 办公助手、金融合规推理等场景。

**English**: At the 2026 RISC-V China Summit on October 19, Lanxin Computing unveiled a "compute-fusion" super node extending RISC-V AI compute from chips and servers to cluster scale. The LX500 packs 48 RISC-V cores (32 performance, 16 efficiency) with 75 TOPS INT8, supports INT4 through FP32, uses unified memory sharing DDR5 with CXL expansion, and is adapted for Qwen, DeepSeek and Kimi — targeting private RAG, agent office assistants and financial compliance inference.

来源：[证券时报e公司](https://www.toutiao.com/article/7698590854671368746/)

---

## 三、产业与融资 / Industry & Funding

### 1. 🔥 DeepSeek 新一轮融资拟增至约 1000 亿元，估值约 5000 亿 / DeepSeek Round Reportedly Grows to ~100B Yuan at ~500B Yuan Pre-Money

**中文**：据财新等报道，DeepSeek 第二轮融资目标由 500 亿元升至约 1000 亿元，投前估值约 5000 亿元，宁德时代追加约 100 亿元，上市地点尚未确定。同期国内 AI 基础设施公司无问芯穹完成 23 亿元 C 轮融资。

**English**: DeepSeek's second funding round has reportedly grown from 50 billion to roughly 100 billion yuan at a pre-money valuation of about 500 billion yuan, with CATL adding around 10 billion yuan. A listing venue is undecided. Separately, Chinese AI infrastructure firm Infinigence closed a 2.3 billion yuan Series C.

来源：[PANews 融资周报](https://www.panewslab.com/zh/articles/01a15227-ae6a-729f-8c75-c06ee70bfaa0)

### 2. Tenstorrent 完成 15 亿美元 E 轮，估值 62 亿美元 / Tenstorrent Raises $1.5B Series E at $6.2B Valuation

**中文**：AI 芯片公司 Tenstorrent 完成 15 亿美元 E 轮融资，估值从 26 亿美元升至 62 亿美元，此前曾有收购意向。同期 Modal 完成 7.5 亿美元 D 轮，估值 157.5 亿美元；Labelbox 完成 1 亿美元 E 轮。

**English**: AI chipmaker Tenstorrent raised a $1.5 billion Series E at a $6.2 billion valuation, up from $2.6 billion, after earlier takeover interest. Modal closed a $750 million Series D at $15.75 billion, and Labelbox raised a $100 million Series E.

来源：[VectorWire Deal Flow](https://vectorwire.ai/dashboard/deals)

### 3. 亚马逊收购 DuckDB 母公司 DuckLabs / Amazon Acquires DuckLabs, the Company Behind DuckDB

**中文**：亚马逊收购 DuckDB 背后的公司 DuckLabs。DuckDB 是一款快速增长的开源数据库，广泛用于分析与 AI 数据管道。交易金额未披露。同期 AMD 以约 82 亿美元股票收购 World Labs。

**English**: Amazon acquired DuckLabs, the company behind DuckDB, a fast-growing open-source database widely used in analytics and AI data pipelines. Terms were not disclosed. Separately, AMD acquired World Labs for about $8.2 billion in stock.

来源：[VectorWire Deal Flow](https://vectorwire.ai/dashboard/deals)

### 4. Anthropic 三季度营收 168 亿美元，或于 11 月中 IPO / Anthropic Posts $16.8B Q3 Revenue, IPO Possibly Mid-November

**中文**：消息称 Anthropic 第三季度营收 168 亿美元，高于第二季度的 116 亿美元。虽然增长强劲，但引发投资者对 AI 建设节奏的警惕——市场开始质疑 AI 交易相对不受利率上升影响的叙事。另有报道称 Anthropic 可能很快递交 IPO 申请，上市时间预计在 11 月中旬。

**English**: Anthropic's third-quarter revenue reportedly reached $16.8 billion, up from $11.6 billion in Q2. The growth nonetheless sharpened investor scrutiny of AI buildout pacing, challenging the narrative that AI trades are insulated from rising rates. Reports suggest Anthropic may file for an IPO soon, with a listing possibly in mid-November.

来源：[新浪财经](https://www.toutiao.com/article/7698479604020904498/) / [Sergey Tereshkin](https://sergeytereshkin.ca/publications/startup-and-venture-capital-news-october-20-2026-ramp-valued-at-60-billion)

---

## 四、政策与监管 / Policy & Regulation

### 1. 🔥 中国 AI 技术出口管制草案把「智能体」写进清单，产品出口前须登记 / China's Draft Export Controls Explicitly Name AI Agents, Require Pre-Export Registration

**中文**：10 月 16 日，商务部、科技部发布 2026 年第 48 号公告，就修订《中国禁止出口限制出口技术目录》公开征求意见，拟增列人工智能、生物医药、光伏三个领域 40 项关键技术，其中 AI 领域 23 项，明确包括大模型、算力集群与「智能体」。公告同时宣布对相关技术实施出口前登记的临时管理措施，征求意见期至 11 月 15 日。已开源或已发表论文进入公共领域的技术不纳入管理；在境外自建自营算力中心、联合研发、参与国际标准制定等情形适用自动许可；利用限制类大模型提供 API、App、SaaS 服务不属于技术出口管理范围。据报道，模型门槛为 2 万亿参数以上或性能前十，Step 5 与 DeepSeek V4.1 均低于该门槛。

**English**: On October 16, China's Commerce and Science & Technology ministries issued Announcement No. 48, a draft revision to the Catalogue of Technologies Prohibited or Restricted from Export adding 40 items across AI, biomedicine and solar — 23 in AI, explicitly including large models, compute clusters and "intelligent agents." A temporary pre-export registration requirement takes effect on publication, with public comment open through November 15. Open-sourced or published technologies are carved out; overseas self-built compute centers, joint R&D and standards participation qualify for automatic licensing; providing API, app or SaaS access to restricted models is not treated as technology export. The reported threshold is 2 trillion parameters or top-ten performance, below which both Step 5 and DeepSeek V4.1 fall.

来源：[香港商报](https://hkcd.com/hkcdweb/content/2026/10/19/content_8780644.html) / [AI Agent Store](https://aiagentstore.ai/ai-agent-news/topic/legal-regulatory/2025-09-02/detailed)

### 2. 美国 FTC 向 OpenAI、Anthropic 发出民事调查令 / FTC Serves Civil Investigative Demands on OpenAI and Anthropic

**中文**：10 月中旬，美国联邦贸易委员会（FTC）向 OpenAI 和 Anthropic 发出民事调查令，要求提供安全事件、防护做法、营销与隐私相关文件，暗示可能依据消费者保护法采取执法行动。同期美国 8 个州依消费者保护法起诉 OpenAI，指其安全陈述误导成人并伤害未成年人。

**English**: In mid-October, the US Federal Trade Commission served civil investigative demands on OpenAI and Anthropic seeking documents on safety incidents, safeguards, marketing and privacy — signaling possible consumer-protection enforcement. Separately, eight US states sued OpenAI under consumer protection law, alleging its safety claims misled adults and harmed minors.

来源：[百思特全球AI智库周报](https://www.sohu.com/a/1089991183_120409427)

### 3. 金融稳定理事会发布 AI 负责任采用最终报告，新增 Agentic 系统指引 / FSB Publishes Final AI Practices, Adding Guidance for Agentic Systems

**中文**：金融稳定理事会（FSB）10 月 15 日发布《负责任采用人工智能的良好实践》最终报告，保留 12 项非约束性实践，并针对高度自主/Agentic 系统新增运营指引，明确权限、人工监督模式和停止/kill 控制的预期。该报告定位为 G20 各辖区金融机构的监管基准。

**English**: The Financial Stability Board published its final "Sound Practices for Responsible Adoption of Artificial Intelligence" on October 15, retaining 12 non-binding practices while adding operational guidance for highly autonomous and agentic systems — defining expectations for permissions, human-oversight models and stop/kill controls. It is intended as a supervisory benchmark for financial institutions across G20 jurisdictions.

来源：[AI Agent Store](https://aiagentstore.ai/ai-agent-news/topic/legal-regulatory/2025-09-02/detailed)

### 4. 欧盟 AI 办公室就夏季代理事件向两家实验室发出信息请求 / EU AI Office Requests Information From Two Labs Over Summer Agent Incidents

**中文**：欧盟 AI Act 自 8 月 2 日起全面适用，高风险规则推迟至 2027 年 12 月 2 日。欧盟 AI 办公室 10 月 14 日就夏季代理事件向两家实验室发出信息请求，罚款上限为全球营业额 3% 或 1500 万欧元。同期加州前沿模型行政令要求 11 月 16 日前提交关于紧急关停与驻场审计的建议。

**English**: The EU AI Act has applied in full since August 2, with high-risk rules deferred to December 2, 2027. The EU AI Office sent information requests to two labs on October 14 regarding summer agent incidents, with penalties capped at 3% of global turnover or 15 million euros. California's frontier-model executive order separately requires recommendations on emergency shutdown and on-site audits by November 16.

来源：[百思特全球AI智库周报](https://www.sohu.com/a/1089991183_120409427)

---

## 五、具身智能与前沿 / Embodied AI & Frontier

### 1. 🔥 英伟达 GR00T 四位核心成员出走创立 Dream Labs，9000 万美元种子轮 / Four Nvidia GR00T Leads Leave to Found Dream Labs With $90M Seed

**中文**：10 月 20 日，原英伟达 GR00T 人形机器人项目四位核心成员创立的 Dream Labs 结束隐身期，宣布完成 9000 万美元种子轮。同日，其首款产品 Dream-1 通用机器人基础模型通过 API 开放，注册后即可接入任意机器人硬件执行自然语言指令，定价为 Pro 版每小时 3 美元、Flash 版每小时 2 美元。技术路径为「世界动作模型」（WAM）：输出末端执行器位姿而非关节角度，使任意双臂平行夹爪机器人共享同一接口，各本体自带逆运动学求解器。团队 2026 年 2 月在英伟达时发布的 DreamZero 研究成果显示，仅用 30 分钟 YAM 机器人数据做后训练即可零样本泛化到未见任务。

**English**: Dream Labs, founded by four core members of Nvidia's GR00T humanoid program, emerged from stealth on October 20 with a $90 million seed round. Its first product, the Dream-1 general robot foundation model, is now available via API — any registered user can connect robot hardware and issue natural-language commands for $3/hour (Pro) or $2/hour (Flash). The architecture is a World Action Model: instead of joint angles, it predicts end-effector poses, so any dual-arm parallel-jaw robot shares the same interface while each body's own inverse kinematics handles the rest. The team's earlier DreamZero work at Nvidia showed zero-shot generalization to unseen tasks after post-training on just 30 minutes of YAM robot data.

来源：[BT财经](https://businesstimescn.com/articles/630363.html)

### 2. 乐聚 15 台人形机器人进产线常态化作业，上料成功率 98% / Leju Deploys 15 Humanoids in Continuous Production, 98% Loading Success

**中文**：10 月 15 日，乐聚机器人与兆丰机电在杭州联合发布「具身智能+汽车零部件上料解决方案」，被称作业界首个可复制的具身智能实景实训示范项目。15 台 KUAVO 5-W 机器人在兆丰机电产线投入常态化运行，上料成功率稳定在 98%，累计运行约 1000 小时，协助生产超过 6 万个零部件，抓取 80-120mm 小件轴承。这标志着具身智能在汽车制造场景完成从「试点验证」到「常态化作业」的跨越。

**English**: On October 15, Leju Robotics and Zhaofeng Electromechanical unveiled a joint "embodied AI + auto parts loading" solution in Hangzhou, billed as the industry's first replicable embodied AI training demonstration. Fifteen KUAVO 5-W robots now run continuously on Zhaofeng's line, loading 80-120mm bearing parts with 98% success over roughly 1,000 hours and more than 60,000 components — marking a shift from pilot validation to routine operation in auto manufacturing.

来源：[南方+/今日头条](https://www.toutiao.com/article/7698550530892743206/)

### 3. 工信部：上半年人形机器人整机产品超 400 款，全年产量将破 10 万台 / MIIT: Over 400 Humanoid Models, Annual Output to Top 100,000 Units

**中文**：工业和信息化部科技司司长魏巍表示，工信部正推动具身智能产业三个「加速」：规模体量加速壮大，今年上半年我国人形机器人整机产品突破 400 款，产量超 4 万台，预计全年将突破 10 万台；创新能力加速跃升，攻关运动控制、仿真模拟、触觉传感等关键技术；产业应用加速落地，推动产品在仓储物流、餐饮零售、家庭服务等 300 多个重点场景验证部署。

**English**: MIIT's science and technology chief Wei Wei outlined three accelerations for embodied AI: scale, with more than 400 humanoid models and over 40,000 units produced in H1, expected to exceed 100,000 for the full year; capability, with work on motion control, simulation and tactile sensing; and deployment, with validation across more than 300 scenarios including warehousing, food retail and home services.

来源：[光明网](https://news.gmw.cn/2026-10/20/content_39054484.htm)

### 4. 三星测试 24 小时人形机器人，或在 CES 2027 首秀 / Samsung Tests 24/7 Humanoid, Possible CES 2027 Debut

**中文**：据业界消息，三星电子正加快机器人布局，其 Robot Experience（RX）部门在首尔研发中心测试一款可 24 小时运行的人形机器人，目标是工业场景的连续作业。该机器人目前在学习环境导航与搬运等重复任务，测试重点为长时间无人工干预下的可靠性与自主导航。三星计划让其在 2027 年 1 月拉斯维加斯 CES 上首次公开亮相。

**English**: Samsung Electronics' Robot Experience division is testing a humanoid designed for around-the-clock industrial work at its Seoul R&D center, according to industry sources. The robot is learning navigation and repetitive tasks such as transporting goods in manufacturing and logistics, with testing focused on reliability without human intervention over extended periods. A first public appearance is planned for CES 2027 in Las Vegas in January.

来源：[Embodied AI Frontier](https://embodiedaifrontier.com/articles/c3WEbS.html)

---

**说明**：本条为 2026-10-20 补跑存档（补跑时点约 21:50），覆盖过去 24 小时（10 月 19 日全天至 20 日）的全球 AI 动态。

**English note**: Backfilled archive entry generated around 21:50 on October 20, covering the past 24 hours of global AI news.
