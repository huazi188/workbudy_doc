# 2026-10-20 AI 硬件汇总

**日期：2026 年 10 月 20 日**

**总览（中文）**：今天最重的一条是华为首台 1024 卡 Atlas 950 SuperPoD 液冷超节点在 10 月 15 日随 StepFun Step 5 Preview 开源同步点亮，1 EFLOPS FP8、全球统一内存寻址到 256TB、互连时延压到 3 微秒；国内超节点则在 CIIF 2026 上集体亮相，天数智芯、沐曦、昆仑芯、摩尔线程、平头哥、寒武纪、海光、中兴、曙光几乎全线出柜；海外一侧，NVIDIA Vera Rubin 已量产、联想在 OCP 峰会宣布全球交付 NVL72，AMD Helion 以低三成报价切入机架级。

**Overview (EN)**: The headline is Huawei's first 1024-card Atlas 950 SuperPoD liquid-cooled supernode, lit up on October 15 alongside the open-sourcing of StepFun Step 5 Preview — 1 EFLOPS FP8, 256TB globally unified memory addressing, 3-microsecond interconnect latency. At CIIF 2026 domestic vendors showed supernodes almost across the board, while NVIDIA's Vera Rubin reached volume production, Lenovo announced global delivery of Vera Rubin NVL72 at OCP, and AMD positioned Helion at roughly 30% below.

---

## 第一部分：国内 / China

### 1. 昇腾 Atlas 950 SuperPoD 1024 卡超节点首次点亮，与 Step 5 Preview 开源同步

**Huawei Atlas 950 SuperPoD, a 1024-card supernode, lights up for the first time alongside the open-sourcing of Step 5 Preview**

【超节点·华为·已点亮】

华为首台 1024 卡 Atlas 950 SuperPoD 液冷系统在 10 月 15 日完成点亮，与阶跃星辰 StepFun Step 5 Preview 的开源动作同一天披露。整机 FP8 算力 1 EFLOPS、FP4 算力 2 EFLOPS，全局统一内存寻址到 256TB，互连时延低至 3 微秒。这是昇腾从"单卡对标"转向"系统级对标"的关键一步——用一个机柜把卡的规模优势换成模型侧的可用算力。

Huawei's first 1024-card Atlas 950 SuperPoD liquid-cooled system was lit up on October 15, disclosed the same day StepFun open-sourced Step 5 Preview. It delivers 1 EFLOPS in FP8 and 2 EFLOPS in FP4, with globally unified memory addressing up to 256TB and interconnect latency as low as 3 microseconds. It marks Ascend's pivot from card-level benchmarking to system-level benchmarking.

同日，昇腾、沐曦、壁仞、摩尔线程四家宣布对 Step 5 Preview 实现 Day-0 适配。

On the same day, Ascend, MetaX, Biren and Moore Threads all announced Day-0 support for Step 5 Preview.

---

### 2. CIIF 2026 国芯展：国产超节点集体出柜，单柜 144 卡成为新基线

**CIIF 2026: domestic supernodes appear en masse, 144 GPUs per rack becomes the new baseline**

【超节点·多家厂商·已发布】

CIIF 2026 国芯展上，国产超节点几乎全线亮相，单柜 144 卡逐渐取代 64 卡成为新基线：

- **天数智芯 天垓 300 IXL144**：单柜 144 颗 GPU + 18 颗 CPU，约 10.1TB 显存，兼容 CUDA。
- **沐曦 曦景 S600**：64 卡全互连，可扩展至万卡；MXMACA 开源社区开发者超 50 万。
- **昆仑芯**：32/64 卡超节点方案，单柜可扩至 512 卡，冷板液冷，卡间通信带宽提升 8 倍。
- **摩尔线程 夸娥 MTT S5000**：搭载第四代 MUSA 架构"平湖"PH100。
- **阿里平头哥 真武 M890**：144GB 显存，片间互联 800GB/s，P2P 时延低于 150ns，配套磐久 AL128。
- **寒武纪 思元 MLU370-X8**：chiplet 设计，支持 Day-0 适配。
- **海光 深算 3.0 DCU K100**。
- **中兴 OEX**。
- **中科曙光 8000 系列**：面向全国产十万卡集群。

At CIIF 2026, domestic supernodes were on display almost across the board, with 144 GPUs per rack displacing 64 as the new baseline: Iluvatar's TianGai 300 IXL144 packs 144 GPUs plus 18 CPUs with roughly 10.1TB of memory and CUDA compatibility; MetaX's Xijing S600 offers 64-card full interconnect scalable to 10,000 cards; Kunlunxin scales a single rack to 512 cards with cold-plate liquid cooling; Moore Threads' KUAE MTT S5000 uses the fourth-gen MUSA "Pinghu" PH100; Alibaba T-Head's Zhenwu M890 carries 144GB of memory with 800GB/s die-to-die links and sub-150ns P2P latency alongside the Panjiu AL128; Cambricon's MLU370-X8 uses a chiplet design; Hygon showed Shengsuan 3.0 DCU K100; ZTE showed OEX; and Sugon's 8000 series targets fully domestic 100,000-card clusters.

**缺口**：天数智芯、沐曦、昆仑芯三家的官方报价与交付节奏均未披露。

**Gap**: Pricing and delivery timelines for Iluvatar, MetaX and Kunlunxin remain undisclosed.

---

### 3. 天数智芯 天垓 300 走"兼容 CUDA"路线，迁移成本成为卖点

**Iluvatar's TianGai 300 bets on CUDA compatibility, making migration cost the selling point**

【训练卡·天数智芯·已发布】

天垓 300 IXL144 的定位很明确：不去争"纯自研生态"，而是把 CUDA 兼容性做成第一卖点。单柜 144 GPU / 18 CPU、约 10.1TB 显存的配置，配合 CUDA 兼容，实质上是在对客户说"你的代码不用改"。对已经跑在 NVIDIA 栈上的团队，这条路的迁移阻力最小。

The TianGai 300 IXL144 takes a clear position: rather than competing on a purely in-house software stack, it makes CUDA compatibility the headline feature. With 144 GPUs and 18 CPUs per rack plus about 10.1TB of memory, the pitch to customers is effectively "your code doesn't need to change" — the lowest-friction path for teams already running on NVIDIA's stack.

**落地观察**：对采购方来说，兼容路线的风险在于性能上限受制于指令映射效率，官方未给出逐模型的对标数据。

**Deployment note**: For buyers, the risk of the compatibility route is that performance ceilings are bound by instruction-mapping efficiency, and no per-model benchmark data has been released.

---

### 4. 沐曦 曦景 S600 用"开源社区 50 万开发者"做生态背书

**MetaX positions Xijing S600 with 500,000 MXMACA developers as ecosystem proof**

【超节点·沐曦·已发布】

曦景 S600 采用 64 卡全互连、可扩展至万卡的架构，硬件规格不算激进，但沐曦把 MXMACA 开源社区超 50 万开发者这个数字摆到了台前。在国产算力普遍被质疑"硬件有、软件弱"的语境下，这个数字是用来对冲质疑的。

The Xijing S600 uses a 64-card full-interconnect architecture scalable to 10,000 cards. The hardware spec is not aggressive; instead, MetaX leads with the figure of more than 500,000 developers in the open-source MXMACA community — a number deployed to counter the common critique that domestic compute has hardware but weak software.

**缺口**：50 万开发者的口径（注册数 / 活跃数 / 累计数）未明确。

**Gap**: The definition behind the 500,000-developer figure — registered, active, or cumulative — is not specified.

---

### 5. 昆仑芯押注冷板液冷与卡间带宽，单柜扩到 512 卡

**Kunlunxin bets on cold-plate liquid cooling and inter-card bandwidth, scaling a rack to 512 cards**

【超节点·昆仑芯·已发布】

昆仑芯给出 32/64 卡两档超节点方案，单柜可扩展至 512 卡，采用冷板液冷，卡间通信带宽提升 8 倍。选择冷板而非浸没，说明其目标客户仍是已有风冷机房、只能做局部改造的存量数据中心——这比"新建液冷机房"的现实得多。

Kunlunxin offers 32- and 64-card supernode configurations, scalable to 512 cards per rack, using cold-plate liquid cooling with 8x the inter-card communication bandwidth. The choice of cold plate over immersion signals that the target customer is still an existing air-cooled data center capable of only partial retrofit — a far more realistic scenario than building a new liquid-cooled facility.

---

### 6. 摩尔线程 夸娥 MTT S5000 上第四代 MUSA "平湖" PH100

**Moore Threads' KUAE MTT S5000 carries the fourth-gen MUSA "Pinghu" PH100**

【超节点·摩尔线程·已发布】

夸娥 MTT S5000 搭载第四代 MUSA 架构"平湖"PH100。摩尔线程的路线与天数智芯相反——坚持自研架构迭代而非兼容，代价是生态迁移成本由客户承担，收益是长期不受制于人。

The KUAE MTT S5000 runs the fourth-generation MUSA "Pinghu" PH100. Moore Threads takes the opposite path from Iluvatar — iterating its own architecture rather than pursuing compatibility. The cost is that ecosystem migration falls on the customer; the benefit is long-term independence.

**缺口**：PH100 的制程、显存规格、单卡算力均未在展会上披露。

**Gap**: Process node, memory spec and per-card compute for the PH100 were not disclosed at the show.

---

### 7. 平头哥真武 M890 把"低时延互连"做成量化指标

**T-Head's Zhenwu M890 turns low-latency interconnect into hard numbers**

【超节点·阿里平头哥·已发布】

真武 M890 给出 144GB 显存、片间互联 800GB/s、P2P 时延低于 150ns，配套磐久 AL128 整机。这是国内厂商里少见的把互连指标写到纳秒级的做法——对推理场景，P2P 时延直接决定多卡并行时的有效吞吐，比峰值算力更能说明问题。

The Zhenwu M890 specifies 144GB of memory, 800GB/s die-to-die interconnect, and sub-150ns P2P latency, paired with the Panjiu AL128 system. It is a rare case of a domestic vendor quantifying interconnect at the nanosecond level — for inference, P2P latency directly determines effective throughput under multi-card parallelism, and says more than peak FLOPS.

---

### 8. 寒武纪思元 MLU370-X8 用 chiplet，主打 Day-0 适配

**Cambricon's MLU370-X8 uses chiplets and leans on Day-0 adaptation**

【训练卡·寒武纪·已发布】

思元 MLU370-X8 采用 chiplet 设计，并强调对新模型 Day-0 适配。chiplet 路线在国产卡里主要用于绕开先进制程限制，用封装换面积；Day-0 适配则是把"新模型出来当天就能跑"变成可承诺的服务指标。

The MLU370-X8 uses a chiplet design and emphasizes Day-0 adaptation for new models. Among domestic cards, chiplets are mainly a way to work around advanced-node constraints — trading packaging for die area — while Day-0 adaptation turns "runs the day a model ships" into a service-level promise.

---

### 9. 海光深算 3.0 DCU K100 与中兴 OEX 同台，国产算力进入"整机比拼"阶段

**Hygon's Shengsuan 3.0 DCU K100 and ZTE's OEX share the stage as domestic compute enters a systems contest**

【训练卡·海光 / 中兴·已发布】

海光深算 3.0 DCU K100 与中兴 OEX 同台亮相。两家都不是纯粹的芯片公司叙事——海光的优势在 x86 生态衔接，中兴的优势在整机与网络集成。这也说明国产算力的竞争焦点已经从"单卡跑分"移到"整机能不能稳定交付"。

Hygon's Shengsuan 3.0 DCU K100 and ZTE's OEX appeared side by side. Neither tells a pure chip-company story — Hygon's edge is continuity with the x86 ecosystem, ZTE's is system-level and network integration. The competitive focus in domestic compute has shifted from single-card benchmarks to whether a full system can be delivered reliably.

**缺口**：两款产品的具体算力、功耗、量产时间均未披露。

**Gap**: Compute figures, power draw and volume-production timing for both were not disclosed.

---

### 10. 中科曙光 8000 系列指向"全国产十万卡集群"

**Sugon's 8000 series aims at a fully domestic 100,000-card cluster**

【集群·中科曙光·已发布】

曙光 8000 系列直接把目标写成"全国产十万卡集群"。十万卡是当下超大规模训练的门槛规模，把"全国产"和"十万卡"放在同一个句子里，是这一轮展会里口径最激进的一条。

The Sugon 8000 series states its target plainly: a fully domestic 100,000-card cluster. A hundred thousand cards is the entry scale for today's ultra-large training runs, and pairing "fully domestic" with "100,000 cards" in one sentence is the most aggressive claim at this year's show.

**缺口**：十万卡集群的落地时间表、所在地、客户均未披露。

**Gap**: The timeline, location and customer for the 100,000-card cluster are all undisclosed.

---

## 第二部分：国外 / International

### 1. NVIDIA Vera Rubin 已进入量产，单机架估算 780 万美元 📈

**NVIDIA Vera Rubin enters volume production, with rack pricing estimated at $7.8 million 📈**

【平台·NVIDIA·量产中】

10 月 14 日的 SEC 等效文件确认 Vera Rubin 已开始出货。相较 Blackwell，其智能体（agentic）工作负载吞吐约为 10 倍。Morgan Stanley 估算单机架价格约 780 万美元，并预计 2027 年新订单价格再涨 15% 以上。

An SEC-equivalent filing dated October 14 confirmed Vera Rubin has begun shipping. Agentic workload throughput is roughly 10x that of Blackwell. Morgan Stanley estimates a rack at about $7.8 million and expects new orders in 2027 to price a further 15%-plus higher.

**观察**：单机架 780 万美元意味着采购门槛进一步提高，超大规模客户之外的买家会被挤向租赁与云上推理。

**Note**: At $7.8 million per rack, the purchasing bar rises further, pushing buyers outside the hyperscaler tier toward leasing and cloud inference.

---

### 2. 联想 OCP 峰会宣布全球交付 Vera Rubin NVL72 📈

**Lenovo announces global delivery of Vera Rubin NVL72 at OCP Summit 📈**

【整机·联想·已发货】

10 月 16 日，联想在 OCP 峰会上宣布全球交付 Vera Rubin NVL72：72 颗 Rubin GPU + 36 颗 Vera CPU，采用 NVLink 6。官方口径是每兆瓦吞吐提升 30 倍、每百万 Token 推理成本降至 1/45，且完成同等任务仅需 1/4 的 GPU 数量。

On October 16 at the OCP Summit, Lenovo announced global delivery of Vera Rubin NVL72: 72 Rubin GPUs plus 36 Vera CPUs on NVLink 6. The company claims 30x throughput per megawatt, inference cost per million tokens down to 1/45, and one quarter the GPUs needed for the same workload.

**落地观察**："每百万 Token 成本降至 1/45"是本轮最有冲击力的单一数字，但换算基准（对比哪一代、什么模型、什么 batch）未公开。

**Deployment note**: The 1/45 cost-per-million-tokens figure is the most striking single number this cycle, but the comparison baseline — which generation, which model, what batch size — is not public.

---

### 3. AMD Helion 以"便宜三成"切入机架级智能体 AI

**AMD Helion enters rack-scale agentic AI at "30% cheaper"**

【平台·AMD·已发布】

AMD 于 10 月 12 日发布 Helion 机架级智能体 AI 方案，明确以比 Vera Rubin 便宜 30% 作为卖点。在 NVIDIA 把机架价格推到 780 万美元量级后，"便宜三成"是唯一能直接动摇采购决策的差异化话术。

AMD released Helion, a rack-scale agentic AI offering, on October 12, explicitly positioning it at 30% below Vera Rubin. With NVIDIA pushing rack prices toward the $7.8 million mark, "30% cheaper" is the one differentiator that can directly move a purchasing decision.

**缺口**：Helion 的交付时间、客户名单、软件栈成熟度均未披露。

**Gap**: Delivery timing, customer list and software-stack maturity for Helion are undisclosed.

---

### 4. 机架级竞争从"算力密度"转向"每兆瓦每 Token 成本"

**Rack-scale competition shifts from compute density to cost per token per megawatt**

【趋势·全行业·—】

今天三条海外新闻的共同点：联想讲"每兆瓦吞吐 ×30、每百万 Token 成本 ÷45"，AMD 讲"便宜 30%"，NVIDIA 讲"智能体吞吐 ×10"。没有一家在讲峰值 FLOPS。这说明机架级产品的评价体系已经换成"单位电力产出多少可用 Token"。

The three international stories today share a common thread: Lenovo talks throughput per megawatt (30x) and cost per million tokens (1/45), AMD talks 30% cheaper, NVIDIA talks 10x agentic throughput. None leads with peak FLOPS. The evaluation standard for rack-scale products has become how many usable tokens come out of a unit of power.

**观察**：这个口径变化对国产厂商是压力也是机会——压力在于需要给出同等口径的实测数据，机会在于国产方案在电力成本上有结构性优势。

**Note**: This shift is both pressure and opportunity for domestic vendors — pressure to publish measured data on the same basis, opportunity because domestic designs hold a structural advantage on power cost.

---

### 5. 液冷从"可选项"变成"默认项"

**Liquid cooling moves from optional to default**

【配套·全行业·—】

国内昆仑芯选冷板液冷、华为 Atlas 950 是液冷系统；海外 Vera Rubin NVL72 与 Helion 同样是整机架液冷形态。机架功率密度上到 100kW 量级后，风冷已经没有余量，液冷不再是"要不要做"的问题，而是"冷板还是浸没"的选择题。

Domestically, Kunlunxin chose cold-plate liquid cooling and Huawei's Atlas 950 is a liquid-cooled system; abroad, Vera Rubin NVL72 and Helion are likewise full-rack liquid-cooled. Once rack power density reaches the 100kW class, air cooling has no headroom left — liquid cooling is no longer a yes-or-no question, only a cold-plate-versus-immersion choice.

**落地观察**：对存量机房，冷板是唯一可行路径；浸没仍受制于运维习惯与改造成本。

**Deployment note**: For existing facilities, cold plate is the only viable path; immersion remains constrained by operational habits and retrofit cost.

---

### 6. 生态适配的竞争焦点从"支持"变成"Day-0"

**Ecosystem adaptation shifts from "supported" to "Day-0"**

【生态·全行业·—】

国内昇腾、沐曦、壁仞、摩尔线程四家同日宣布对 Step 5 Preview 实现 Day-0 适配；寒武纪也在强调 Day-0。这意味着"新模型发布后多久能跑起来"已经成为可公开比较的指标，而不是事后补丁。

Domestically, Ascend, MetaX, Biren and Moore Threads all announced Day-0 support for Step 5 Preview on the same day, and Cambricon emphasizes Day-0 as well. Time-to-run after a new model ships has become a publicly comparable metric rather than an after-the-fact patch.

**缺口**：各家的"Day-0 适配"是否包含性能调优、是否覆盖训练与推理两侧，均未说明。

**Gap**: Whether each vendor's Day-0 support includes performance tuning, and whether it covers both training and inference, is not stated.

---

### 7. 训练与推理的分野在硬件层面固化

**The training/inference split hardens at the hardware level**

【趋势·全行业·—】

天垓 300 强调 CUDA 兼容（照顾已有训练代码），真武 M890 强调 P2P 时延（照顾推理并行），寒武纪强调 Day-0（照顾新模型上线速度）。三家的卖点分别对应训练迁移、推理效率、上线速度三种不同诉求——硬件正在按场景分化，而不是追求一颗卡打全场。

TianGai 300 emphasizes CUDA compatibility (serving existing training code), Zhenwu M890 emphasizes P2P latency (serving inference parallelism), and Cambricon emphasizes Day-0 (serving time-to-launch for new models). The three pitches map to three distinct needs — training migration, inference efficiency, launch speed. Hardware is differentiating by scenario rather than chasing one card for everything.

---

### 8. 十万卡与 1024 卡：规模叙事的两条线

**100,000 cards versus 1,024 cards: two tracks in the scale narrative**

【趋势·华为 / 曙光·—】

华为给的是"单柜 1024 卡、256TB 统一寻址、3 微秒时延"，曙光给的是"全国产十万卡集群"。前者解决"一个机柜内不要跨网络"，后者解决"集群规模上限"。两条线并不冲突——超节点是把通信压进机柜，集群是把机柜连成一片，缺一条都拼不出可用的超大规模训练系统。

Huawei offers 1,024 cards per rack, 256TB of unified addressing and 3-microsecond latency; Sugon offers a fully domestic 100,000-card cluster. The former solves "don't cross the network inside a rack," the latter solves the ceiling on cluster scale. The two tracks don't conflict — supernodes push communication into the rack, clusters stitch racks together, and without both there is no usable ultra-large training system.

---

### 9. 单机架 780 万美元背后的采购结构变化

**What a $7.8 million rack does to purchasing structures**

【市场·全行业·—】

Vera Rubin 单机架 780 万美元、2027 年新单再涨 15%，意味着自建集群的资本门槛继续抬升。可预期的结果是：中小团队进一步转向云上推理，大客户则更倾向于长约锁价。硬件厂商的收入结构会随之从"卖设备"向"卖容量"倾斜。

At $7.8 million per Vera Rubin rack, with 2027 orders expected 15% higher, the capital bar for self-built clusters keeps rising. The likely result: smaller teams move further toward cloud inference, while large customers favor long-term contracts to lock in pricing. Vendor revenue structures tilt from selling equipment toward selling capacity.

**缺口**：780 万美元为卖方研究估算，非 NVIDIA 官方定价。

**Gap**: The $7.8 million figure is a sell-side estimate, not official NVIDIA pricing.

---

### 10. 国产算力的真实瓶颈仍在软件与交付

**The real bottleneck for domestic compute remains software and delivery**

【判断·国产阵营·—】

把今天国内 10 条放在一起看：硬件规格已经不落后（144 卡单柜、256TB 寻址、150ns P2P 时延都拿得出手），但几乎每一家都在用"生态""兼容""适配"作为主要话术，且报价、交付节奏、逐模型实测数据普遍缺失。这说明竞争已经从"能不能做出来"进入"能不能稳定交付、能不能让客户少改代码"的阶段。

Taken together, today's ten domestic items show hardware specs that no longer lag — 144 cards per rack, 256TB addressing, 150ns P2P latency are all credible — yet nearly every vendor leads with ecosystem, compatibility or adaptation as its main pitch, while pricing, delivery cadence and per-model measured data are broadly missing. Competition has moved from "can it be built" to "can it be delivered reliably, and can the customer avoid rewriting code."

**观察**：下一次真正的分水岭，会出现在第一家愿意公开逐模型实测数据的厂商身上。

**Note**: The next real dividing line will be the first vendor willing to publish per-model measured benchmarks.

---

## 缺口汇总 / Gap Summary

| 项目 | 缺失内容 | 影响 |
|---|---|---|
| 国产超节点报价 | 天数智芯、沐曦、昆仑芯均未披露价格 | 无法判断性价比与采购意愿 |
| 国产芯片量产节奏 | 海光 K100、中兴 OEX 未披露算力/功耗/量产时间 | 难以评估实际可用规模 |
| Day-0 适配口径 | 是否含性能调优、是否覆盖训练+推理 | 指标可比性存疑 |
| Vera Rubin 价格 | 780 万美元为卖方估算，非官方定价 | 采购预算测算存在偏差 |
| 成本对比基准 | "每百万 Token 成本 1/45"未公开对比基准 | 数字冲击力大于可验证性 |
| AMD Helion | 交付时间、客户、软件栈成熟度未披露 | 难以判断是否真能形成替代 |

**未覆盖方向**：AI 眼镜、AI 玩具、具身智能机器人等消费级硬件本日无重大新增，故未单列；如需补，建议下一期单独扫一遍。

---

*本汇总由定时任务「AI硬件汇总」生成，归档路径 `/workspace/daily/2026-10-20_AI硬件汇总.md`。*
