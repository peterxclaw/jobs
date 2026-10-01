# Decision Models（决策模型）产品与技术进展全景报告

> **截至日期**：2026-10-01
> **覆盖范围**：TypeSafe AI（Jev）、OpenAI（Decisions API）、SGLang / LMSYS、vLLM + Google DiffusionGemma、开源社区（Laya、Kev、AnyJev、AgentJev 等）、国内动态、第三方评测与学术研究
> **说明**：本文资料来自公开报道、官方文档、博客与开源仓库。凡是厂商**自报**、尚未经第三方复现的数据，文中都会标注；不同来源数字不一致时，会把各方说法一并列出。

---

## 目录

1. [一句话概括](#1-一句话概括)
2. [什么是 Decision Model](#2-什么是-decision-model)
3. [时间线（2026 年 9 月）](#3-时间线2026-年-9-月)
4. [主要玩家详解](#4-主要玩家详解)
   - 4.1 [TypeSafe AI — Jev](#41-typesafe-ai--jev开创者)
   - 4.2 [OpenAI — Decisions API](#42-openai--decisions-api)
   - 4.3 [SGLang / LMSYS — 原生决策 API](#43-sglang--lmsys--原生决策-api)
   - 4.4 [vLLM + Google DiffusionGemma](#44-vllm--google-diffusiongemma)
   - 4.5 [开源社区模型](#45-开源社区模型)
   - 4.6 [国内动态](#46-国内动态)
5. [技术路线对比](#5-技术路线对比)
6. [产品横向对比](#6-产品横向对比)
7. [第三方评测与学术研究](#7-第三方评测与学术研究)
8. [局限性与争议](#8-局限性与争议)
9. [应用场景与架构模式](#9-应用场景与架构模式)
10. [趋势判断](#10-趋势判断)
11. [参考资料](#11-参考资料)

---

## 1. 一句话概括

**Decision Model（决策模型，又称 "System One 模型"）不生成文本，只做"选择题"**：开发者给出一段上下文（state）和一组类型化问题（是/否、单选、打分），模型一次前向就返回**每个选项的校准概率**，延迟在几十到几百毫秒，单次成本只有通用大模型的几十分之一到几百分之一。

2026 年 9 月 15 日 TypeSafe AI 发布 **Jev**，开创了这个品类。之后两周内，**OpenAI、SGLang、vLLM/Google，以及大量开源项目**相继跟进，形成了一个新的模型赛道。

---

## 2. 什么是 Decision Model

### 2.1 核心定义

| 维度 | 传统 LLM（System 2 / 生成式） | Decision Model（System 1 / 决策式） |
|---|---|---|
| 输出 | 自由文本，逐 token 自回归生成 | 类型化结果（选项、分数、布尔值）+ 概率 |
| 输出格式保证 | 依赖提示词、JSON mode、约束解码 | **从结构上保证**符合 schema，不可能出现类型错误 |
| 置信度 | 一般没有，或需要自己解析 logprobs | 每个答案都带**校准后的概率** |
| 延迟 | 秒级 | 约 30–500 ms |
| 计费 | 输入 + 输出 token | Jev：只收输入费，输出免费 |
| 擅长 | 推理、写作、多步规划 | 分类、路由、打分、校验、Agent 下一步动作选择 |
| 不擅长 | 高频、低延迟、大批量判断 | 多步推理、数学校验、生成内容 |

> 命名由来：**"System One"** 来自 Kahneman 的快思考/慢思考理论；**"Jev"** 来自杰文斯悖论（William Stanley Jevons）：单次决策足够便宜之后，决策的总调用量会成倍增长。

### 2.2 三种标准问题类型（事实标准由 TypeSafe 定义）

| 类型 | 含义 | 返回值 |
|---|---|---|
| **Noul**（Bernoulli） | 这句话是否成立？ | 0–1 的概率 |
| **Choice** | 从给定列表中选一项 | 选中项 + 各选项概率 + 置信度（Jev 最多支持 255 个选项） |
| **Score** | 按评分细则打分 | 分数 + 分布 + 置信度 |

多个问题可以针对**同一个 state 并行求值**，额外开销很小。官方建议把复杂请求拆成多个原子问题，再由业务代码组合结果，而不是让模型自己做长链推理。

### 2.3 事实上的接口标准：`/v1/systemone`

TypeSafe 的 System One API 已经成为事实标准。**Kev、Laya Server、SGLang、vLLM 示例服务**都兼容这个接口，**TypeSafe Python SDK 不用改代码就能直接连这些后端**。这和当年 OpenAI Chat Completions 接口被各家兼容的过程很像。

---

## 3. 时间线（2026 年 9 月）

| 日期 | 事件 |
|---|---|
| **09-15** | **TypeSafe AI 发布 Jev**（隐身研发两年），同时宣布由 DCVC 领投的 **4000 万美元种子轮**，开放早期访问 |
| 09-18 | Convai Innovations 开源 **Laya**（ModernBERT-large，421M，Apache 2.0） |
| 09-19 | Laya-multilingual 发布（mmBERT-base，322M） |
| 09-21 | Simon Willison 撰文介绍 Jev，称之为 "a new shape of LLM"；国内券商研报（建投计算机）开始跟进 |
| 09-22 | **vLLM PR #57250 合并**，支持 DiffusionGemma 的"结构化读取"；OpenAI 发布 GPT-6 Luna；"JEV-as-a-Judge" 级联研究发布 |
| 09-24 | AIMultiple 发布 **AIM-Decision 浏览器自动化评测**；"Jev vs LLMs as Rubric Judges" 研究发布 |
| 09-25 | **LMSYS 博客：《Scaling JEV-like Decision Models with SGLang》**（LinkedIn、SGLang、NVIDIA、TikTok 联合作者） |
| 09-28 | Red Hat 开发者博客：在 vLLM / Red Hat AI 上用 DiffusionGemma 运行决策模型 |
| **09-29** | **OpenAI DevDay 2026 发布 Decisions API**（基于 GPT-6 Luna，有限预览） |
| 09-30 | SGLang 原生决策 API 进入 nightly 构建（正式版计划在 v0.5.21） |

---

## 4. 主要玩家详解

### 4.1 TypeSafe AI — Jev（开创者）

#### 公司概况

| 项目 | 信息 |
|---|---|
| 创始人 | **Diogo Almeida**（CEO，前 OpenAI 研究员，参与过 ChatGPT 指令遵循 / RLHF 相关工作）、Erik Gafni、Sasha Sheng |
| 融资 | 4000 万美元种子轮，**DCVC 领投**（2026-09-15 公布） |
| 产品形态 | 闭源托管 API，早期访问 + 候补名单；**不提供开源权重，不支持私有化部署** |
| 当前版本 | Jev 1.13（第三方评测中使用的版本） |

#### 技术要点

- **非自回归（Non-autoregressive）**：不逐 token 生成，单次前向完成决策。schema 中的所有字段**并行填充**。
- **结构上保证 schema 合规**：官方说法是 "type error isn't unlikely — it's unavailable"（类型错误不是概率低，而是根本不可能出现）。
- **RLCD 训练（Reinforcement Learning for Calibrated Decisions）**：优化目标是**概率诚实（校准）**，不是人类偏好（RLHF），也不是可验证奖励（RLVR）。
- **选项基数上限 255**，超出时需要两阶段打分。

#### 性能与价格

| 指标 | 数值 | 备注 |
|---|---|---|
| 输入价格 | **$0.042 / 百万 token** | 比 GPT-5 Nano（$0.05）还便宜 |
| 输出价格 | **免费**（"too cheap to meter"） | |
| 延迟 | 70–500 ms（自报）；**中位数约 0.15 s**（CMU 实测 152 ms） | |
| 吞吐 | 350+ RPS / 1 vCPU（自报） | |
| 速度 / 成本优势 | 比前沿 LLM 快 40–200 倍；官方工作流评测称"快 193.6 倍、便宜 444.6 倍" | **均为自报**，评测格式是 TypeSafe 自己的，未经独立复现 |

#### 演示案例

Minecraft 机器人（2 分钟约 $0.01）、无人机避障仿真（15 分钟约 $0.10）、自动驾驶小游戏、类 Subway Surfers 游戏。**以上全部在模拟器中进行，还没有真实硬件部署。**

#### 社区玩法

发布后几天内社区出现了 jevchat（用 Jev 拼出聊天）、jev-leftpad、jev-2048 等项目。Simon Willison 用它做搜索重排，还做了一个"湾区城市好不好"的测试（Cupertino 最高，East Palo Alto 最低）。他借这个例子提醒，决策模型可能把偏见带进打分。

---

### 4.2 OpenAI — Decisions API

#### 发布信息

- **发布时间**：2026-09-29，**OpenAI DevDay 2026**
- **底座模型**：**GPT-6 Luna**（GPT-6 家族中最小、最便宜的一档，2026-09-22 发布）
- **状态**：有限预览，面向部分 API 客户，官方称"未来几天"全面开放
- **定位**：业界普遍认为这是对 TypeSafe Jev 的直接回应（The New Stack 标题："OpenAI answers TypeSafe's Jev with a Decision API built on Luna"）

#### 功能

- 输入：**上下文（文本或图片）+ 一个问题 + 一组预定义的有限答案**
- 输出：最可能的选项 + 置信度，不输出自由文本
- 官方用例：**实时分类、路由、Agent 下一步动作选择**
- 延迟：**约 150 ms**（标准 Luna 调用约 1.6 s）

#### GPT-6 Luna 底座参数（背景信息）

| 项 | 值 |
|---|---|
| 上下文窗口 | 1,050,000 tokens；最大输出 128,000 |
| 价格 | $0.10 / $0.50 每百万 token（输入超过 272K 后为 $0.20 / $0.75） |
| Artificial Analysis 智能指数 | 37.3（max effort） |
| 知识截止 | 2026-05-18 |

#### 尚未公布

单次决策价格、单请求最大候选数、是否支持微调、SLA。

#### 评价

- Latent Space 认为它目前是 **Luna 上的一层薄封装（"a light shim over Luna"）**，没有 Jev 那种 RLCD 校准训练，但多了**视觉输入**能力。
- OrcaRouter 提醒，Luna 在真实流量中的延迟中位数是 699 ms，**p95 达到 7.6 s**，"只看 p50 来设计超时是错误的"。

#### DevDay 2026 同场发布（背景）

Dots（常驻 Agent，基于 GPT-6 Astra）、GPT-6.1 Sol（$2/$10）、Ultrafast 模式（API 提速 6 倍、价格也是 6 倍）、Agents API、ChatGPT Spaces / Pages、B2B Marketplace、$500 ChatGPT Pro 套餐等。

---

### 4.3 SGLang / LMSYS — 原生决策 API

SGLang 的思路是：**不需要专门训练的模型，任何现有 LLM 都可以当决策模型用**，办法是直接读取下一个 token 在候选标签上的概率。

#### 4.3.1 LMSYS 技术博客（2026-09-25）

**《Scaling JEV-like Decision Models with SGLang》**，作者来自 **LinkedIn、SGLang、NVIDIA、TikTok**。

**API：`/v1/score` 端点**

```python
requests.post("http://localhost:30000/v1/score", json={
    "model": "Qwen/Qwen3-0.6B",
    "query": "Context:\n...\nQuestion: Which action next?\n",
    "items": [
        "Proposed: Query order-status. Correct? Yes/No.",
        "Proposed: Search policy docs. Correct? Yes/No.",
        "Proposed: Ask for ID. Correct? Yes/No.",
    ],
    "label_token_ids": [yes_token_id, no_token_id],
    "apply_softmax": True,
})
```

它直接对指定的标签 token 打分，不依赖 top-k logprobs，因此不会出现需要的标签恰好不在 top-k 里的问题。

**两种提示范式**

| 范式 | 做法 | 特点 |
|---|---|---|
| **Pointwise（逐点）** | 每个候选单独评估："这个动作合适吗？Yes/No" | 候选之间互不影响，容易并行 |
| **Setwise（集合）** | 所有候选放在一起："A/B/C 哪个最好？" | 可以做比较式判断 |

**两种执行模式**

- **SIS（Single-Item Scoring）**：每个 query + item 作为独立序列
- **MIS（Multi-Item Scoring）**：共享 query 前缀，用**候选隔离的注意力掩码**避免重复计算，需要 FlashInfer，启动时加 `--enable-mis`

**性能（单张 H200，16 个候选，低负载）**

| 模型 | MIS（Pointwise） | 普通 Generate | Fused-Choice | Setwise |
|---|---|---|---|---|
| Qwen3-0.6B | **18.7 ms** | 39.6 ms | 25.5 ms | 31.0 ms |
| Qwen3-8B | **20.6 ms** | 54.1 ms | 36.0 ms | 32.2 ms |
| Qwen3.5-4B | **55.7 ms** | 84.8 ms | 48.9 ms | 50.5 ms |

结论：候选越多、负载越高，MIS 的优势越大。Fused-Choice 和 Setwise 没有哪个始终更好，需要在目标负载下实测尾延迟。

评测数据来自 **Open-Jev** 数据集中的选择题任务。除 CausalLM 外，也支持 `QwenForSequenceClassification`、`LlamaForClassification` 等分类头模型。

#### 4.3.2 原生决策 API（2026-09-30，nightly）

- 兼容 TypeSafe SDK 的 **`/v1/systemone` 接口**
- 支持 **Qwen3.8-27B** 等模型，**支持图片输入**（多模态决策）
- **不需要微调**
- 演示：在**《宝可梦 火红》**中实时决定攻击、换宠还是回血，**每次决策不到 100 ms**，打通了四天王和冠军
- 正式版计划在 **v0.5.21**

> 背景：SGLang 已部署在 **40 万以上张生产 GPU** 上（xAI、AMD、NVIDIA、Intel、LinkedIn、Cursor 等）。

---

### 4.4 vLLM + Google DiffusionGemma

这条路线用**扩散语言模型**做决策：把答案槽位放进模板，一步去噪就能读出所有槽位的概率分布。

#### DiffusionGemma 26B-A4B-it（Google DeepMind）

- **块扩散（block-diffusion）语言模型**，基于 Gemma 4 MoE 骨干
- **总参数 26B（另一来源为 25.2B），激活约 4B（3.85B）**
- 在固定长度的 token 画布上**并行**填充，天然适合一次回答多个结构化问题

#### vLLM "结构化读取（Structured Read）"模式：PR #57250，2026-09-22 合并

1. **种子画布**：客户端用答案模板预填画布（如 `urgent: @` / `category: @`），只让答案槽位保持噪声
2. **单步去噪**：只执行一次只读扩散步
3. **读取分布**：每个槽位返回校准后的 logprobs
4. **条件重读**：熵超过阈值时再采样几次，看结果是否一致

- 问题类型：`noul` / `choice` / `score`，答案必须是单 token
- 画布长度 64 token，max logprobs 32
- 示例端点：`/v1/systemone`（还不是 vLLM 的正式标准接口）

#### 性能

| 场景 | 延迟 | 吞吐 |
|---|---|---|
| 单请求 | 0.12 s | 8.7 req/s |
| batch 32 | 0.58 s | 54 req/s（3 题/请求，即 162 决策/s） |
| Google Cloud Run | **35–60 ms 单步** | 100–123 req/s（batch 32），**300+ 决策/s** |

Red Hat 已将其纳入 **Red Hat AI** 方案，vLLM Recipes 提供 H100/H200/DGX Spark/Xeon 6 的部署命令。社区项目 **djev-dev** 增加了图像输入、图像选项和实时摄像头采样。

---

### 4.5 开源社区模型

| 项目 | 机构/作者 | 架构 | 参数规模 | 关键数据 | 许可 |
|---|---|---|---|---|---|
| **Laya** | Convai Innovations | 双向编码器（ModernBERT-large / mmBERT-base）+ 决策头；每个选项前放一个 `[MASK]` | 421M（英文）/ 322M（多语言） | T4 上约 **33 ms**；微调后准确率 0.766（Jev 对照为 0.727，延迟低 7.8 倍）；校准误差 0.466 → 0.081（温度拟合后） | Apache 2.0 |
| **Kev** | Jared Palmer | Qwen 解码器 + rank-16 LoRA + **指针头（pointer head）**；问题共享 state 缓存，读出彼此独立 | 0.8B / 4B / 9B / 27B（Qwen3.5/3.8） | Kev-27B 新来源数据集准确率 **0.851**（Jev 为 0.857）；H100 上 Kev-4B 18–41 ms | Apache 2.0 |
| **AnyJev** | Nokia Applied Research | 在现有解码器上做多级读出：L0 受限 next-token 概率，L2 隐藏层闭式头 | 如 Qwen3-8B | BANKING77（20 类）在 5% 接受错误率下覆盖率 52.0%；**每个问题只需 100–300 条标注，不更新梯度** | 开源 |
| **AgentJev** | iTech（国内） | 去掉 Qwen3-0.6B 的 LM head，换成**置换等变（permutation-equivariant）打分头** | 0.6B | 单次前向 50–70 ms；Typed Decisions 测试集 Top-1 **79.25%**（比 Laya 高 2.25 pp）；Brier 0.0448 | Apache 2.0 |
| **NeoHorse-Jev-4B** | 社区 | — | 4B | 中文客服路由 0.912，超过 Jev 的 0.882（见 zh-decision-bench） | Apache 2.0 |
| **Kev-0.5B（衍生版）** | 社区 | Qwen2.5-0.5B + LoRA，兼容 TypeSafe API | 0.5B | 可在 MacBook Pro 上训练和运行 | 开源 |

**周边生态**

- **Laya Server**（1Panel-dev）：自托管 API 和 Web 界面，**兼容 Jev API 格式**
- `pip install "laya[serve]"` 后运行 `laya-serve`，即可提供 `POST /v1/systemone`
- Laya 0.3.21 新增**拒答（abstention）**、ONNX 和批处理接口
- Agent 框架（如 jarviscore-framework）正在把 Laya 加为与 Jev 并列的决策后端

> ⚠️ 注意：Laya 在 systemonemodels.org 的 **Decision Index 0.2.1** 上只得到 **6.04 分（Jev 为 57.91）**，零样本准确率 0.362，随机基线是 0.318。小编码器路线在零样本泛化上**明显不如** Jev。

---

### 4.6 国内动态

| 方向 | 内容 |
|---|---|
| **券商研究** | 建投计算机发布《Jev 探索通用决策模型，关注 Agent 应用效率提升》（2026-09-21）。实测中文客服样本**完整准确率 64%–65.2%**，平均每题 0.73–0.75 s，约 $0.002/题；搜索重排接近内部专用模型，主题分类仍有差距。建议关注**智能客服、企业流程自动化**的落地情况 |
| **阿里 Qwen** | Qwen 系列是开源决策模型（Kev、AgentJev、SGLang 示例）**最常用的底座**。Qwen-Audio-3.1-Realtime（2026-09-28）内置一个**全双工"决策模型"**，判断该继续听、说话、停止还是恢复。有报道称阿里云百炼（Model Studio）在 9 月 24 日上线了决策模型，**但本文没有在阿里云官方渠道找到对应发布，暂作待核实处理** |
| **TikTok（字节跳动）** | 联合署名 LMSYS 的 SGLang 决策模型博客 |
| **开源 / 个人项目** | **AgentJev**（0.6B）；**qwen3-vl-decision**（贷款平台客服的意图识别、业务路由和工具选择，基于 Qwen3-VL 做 QLoRA 微调和置信度校准）；**zh-decision-bench**（首个中文决策模型评测） |

#### zh-decision-bench：首个中文实测（v0.2，378 项 / 467 题）

| 指标 | Jev | NeoHorse-Jev-4B | Qwen3.5-2B（logit 探针） | Laya |
|---|---|---|---|---|
| 语音指令路由准确率 | **0.960** | 0.950 | 0.938 | — |
| 电商客服路由 | 0.882 | **0.912** | — | — |
| 欺诈检测 | **0.952** | — | 0.524 | — |
| 选项顺序翻转率（越低越好） | **1.9%** | — | — | 10.8% |
| 繁体中文翻转率 | **1.7%** | — | — | 12.8% |

结论：Jev 在**鲁棒性**（选项顺序、简繁体）上明显领先，开源 4B 模型在部分垂直场景能追平甚至超过 Jev。

---

## 5. 技术路线对比

目前有四条主要技术路线：

```
┌──────────────────────────────────────────────────────────────────────┐
│ ① 专用非自回归决策模型   Jev（闭源）                                    │
│    RLCD 校准训练 + 并行字段填充 + 结构上保证 schema                       │
├──────────────────────────────────────────────────────────────────────┤
│ ② 编码器 + 决策头        Laya                                          │
│    BERT 类双向编码器，[MASK] 读出；极小、极快，零样本能力弱                │
├──────────────────────────────────────────────────────────────────────┤
│ ③ 解码器 LLM + 读出头 / logit 打分                                      │
│    Kev（LoRA+指针头）、AgentJev（等变头）、AnyJev（多级读出）、            │
│    SGLang /v1/score（直接读标签 token 概率，不用训练）、                   │
│    OpenAI Decisions API（Luna 封装）                                     │
├──────────────────────────────────────────────────────────────────────┤
│ ④ 扩散语言模型单步读取   vLLM + DiffusionGemma                           │
│    模板画布 + 单步去噪，多个槽位一次并行读出                               │
└──────────────────────────────────────────────────────────────────────┘
```

| 路线 | 是否需要训练 | 校准质量 | 速度 | 零样本泛化 | 私有化部署 |
|---|---|---|---|---|---|
| ① Jev | 厂商已训练 | 高（RLCD） | 快 | 强 | ❌ |
| ② 编码器 | 需要 | 温度拟合后中等 | **最快** | 弱 | ✅ |
| ③ 解码器读出 | 可选（SGLang 不用训练；Kev 用 LoRA） | 取决于是否校准 | 中等偏快 | 取决于底座 | ✅（OpenAI 除外） |
| ④ 扩散单步 | 不用训练 | logprobs 自带，可用熵触发重读 | 快 | 强（26B 底座） | ✅ |

**关键工程技术**

- **共享前缀 + 候选隔离注意力掩码**（SGLang MIS、Kev）：多个候选或问题复用同一个 state 的 KV 缓存
- **温度拟合（temperature scaling）**：几乎每个开源方案都靠它来校准概率
- **拒答 / 级联（abstain & escalate）**：置信度低于阈值时交给大模型处理
- **熵触发重读**（vLLM）：不确定时多采样几次，测一致性

---

## 6. 产品横向对比

| 维度 | **TypeSafe Jev** | **OpenAI Decisions API** | **SGLang Decision API** | **vLLM + DiffusionGemma** | **Laya** | **Kev** |
|---|---|---|---|---|---|---|
| 类型 | 闭源托管 API | 闭源托管 API | 开源推理框架功能 | 开源推理框架 + 开放权重 | 开放权重模型 | 开放权重模型 |
| 发布 | 09-15 | 09-29 | 09-25 博客 / 09-30 nightly | 09-22 | 09-18 | 9 月 |
| 底座 | 自研 | GPT-6 Luna | 任意 LLM（Qwen 等） | DiffusionGemma 26B-A4B | ModernBERT / mmBERT | Qwen3.5/3.8 |
| 多模态 | 文本 | **文本 + 图片** | **文本 + 图片** | 社区版支持图片 | 文本 | 文本 |
| 延迟 | ~150 ms 中位 | ~150 ms（自报） | <100 ms；18–56 ms（MIS） | 35–120 ms | ~33 ms（T4） | 18–41 ms（H100，4B） |
| 价格 | $0.042/M 输入，输出免费 | 未公布 | 自付 GPU | 自付 GPU | $0 | 自付 GPU |
| 校准训练 | ✅ RLCD | ❌（薄封装） | ❌（原始概率） | logprobs + 熵 | 温度拟合 | 温度拟合 |
| 兼容 `/v1/systemone` | 原生 | ❌ | ✅ | ✅（示例服务） | ✅ | ✅ |
| 许可 | 专有 | 专有 | Apache 2.0 | Gemma 许可 / Apache | Apache 2.0 | Apache 2.0 |

**市场分层**（Flowtivity 的观点）：

> **OpenAI 占默认层，Jev 占成本优化层，Laya 等开源模型占主权 / 自托管层。**

---

## 7. 第三方评测与学术研究

### 7.1 CMU：《JEV-as-a-Judge: Accept When Confident, Escalate When Unsure》（arXiv 2609.26550）

把 Jev 和 16 个 LLM / 奖励模型评审做对比：

| 指标 | Jev | GPT-6 Astra |
|---|---|---|
| 每千次判断成本 | **$0.044** | $12.18（贵 277 倍） |
| 延迟 | **152 ms** | 1.88 s |
| 常规偏好 / 事实性 | 与 GPT-6 相差不到 3 个点 | — |
| **JudgeBench 困难正确性** | 78.6% | **93.1%** |
| **JudgeBench 推理题** | 68.4% | **95.9%** |

**级联方案**：Jev 置信度 q ≥ τ 时直接采用，否则交给 GPT-6。τ = 0.9 时有 34% 的样本被升级，准确率 91.3%，**达到 GPT-6 级联质量的 99%，成本只有 57%**。

### 7.2 其他级联研究

- **09-22 "JEV-as-a-Judge" 试点**：阈值 0.90，回退到 GPT-6 Astra，覆盖率 53.7%，成本为单用回退模型的 0.568 倍（保守估计 0.622 倍）
- **09-24 "Jev vs LLMs as Rubric Judges"**：5,003 个"提交-评分项"对、9 组评审。**LLM 回退在 96.0% 的抽样中重复了 Jev 的错误**，没有任何级联比最好的单一评审高出 1.5 个百分点以上。**级联不是万能的。**

### 7.3 AIMultiple：AIM-Decision 浏览器自动化评测（09-24）

50 个任务、9 个网站，统一使用 jev-ultrafast 浏览器运行时：

| 模型 | 完成数 | 中位耗时 | 单任务成本 |
|---|---|---|---|
| GPT-6 Astra | **47/50** | 17.4 s | $0.226 |
| Gemini 3.8 Flash | 42/50 | 12.7 s | $0.026 |
| **Jev 1.13** | 17/50 | **4.2 s** | **$0.002** |
| Kev-9B | 20/50（微调后 **32/50**） | 38.3 s | 约 $0.27（GPU 租用） |
| Laya typed-decisions | 0/50 | 15.1 s | $0 |

结论：**在长程 Agent 任务上，决策模型还明显落后于 LLM**。不过 Kev 微调后提升显著（p = 0.003），Jev 的成本只有 Astra 的约 1%。

### 7.4 Amplitude：分析场景实测

64 张合成图表、190 条时间序列，任务是判断哪些异常值得深入调查：

- Jev **召回率 100%，精确率只有 55.8%**
- 每千张图 **$0.06**，中位延迟 **0.14 s**，成本约为 DeepSeek V4.1 Flash 的 1/4，延迟约为其一半
- 关键洞察：**高置信度的错误集中出现在 schema 边界模糊的地方**。类型系统只约束输出的形状，不能保证判断的质量
- 建议：决策模型做第一层过滤，再用下游行为信号（工单重开、用户行为等）闭环评估

### 7.5 其他

- **DecisionBench**（arXiv 2605.19099）：长程 Agent 工作流中"自发委派"的基准
- **Decision Index 0.2.1**（systemonemodels.org）：Jev 57.91，Laya 6.04

---

## 8. 局限性与争议

| 问题 | 说明 |
|---|---|
| **"不会幻觉"≠"不会错"** | 输出格式一定合法，但模型可能**很自信地选错**。格式正确不等于答案正确 |
| **不擅长 System 2 任务** | 多步证明、算术校验等需要逐步推理的任务明显不如 LLM（JudgeBench 推理题 68.4% vs 95.9%） |
| **数字、日期、对抗内容** | Jev 官方文档承认这些是薄弱点 |
| **黑盒化倒退** | Simon Willison："a regression even further towards black box machine learning systems"。只给概率，没有解释，用在招聘等高风险决策中有偏见风险 |
| **厂商数据未经独立复现** | "193.6 倍更快、444.6 倍更便宜"等来自 TypeSafe 自定的评测格式；校准质量缺少外部验证 |
| **选项基数限制** | Jev 上限 255；更大的标签集需要两阶段处理 |
| **输入长度** | Laya 只有 512–1,024 token 上下文，在浏览器任务中 606 条训练记录只有 292 条装得下 |
| **长程 Agent 能力弱** | AIM-Decision 中 Jev 只完成 17/50，存在过早结束等问题 |
| **OpenAI 版本缺乏校准** | 目前是 Luna 的薄封装，没有 RLCD；尾延迟（p95）可能很高 |
| **级联的天花板** | 回退 LLM 经常和决策模型犯同样的错（96%），级联收益有限 |
| **演示停留在仿真** | Jev 的游戏、无人机演示都在模拟器中，没有真实硬件部署 |

---

## 9. 应用场景与架构模式

### 9.1 适用场景

- **分类与路由**：工单分拣、意图识别、模型路由、语音指令路由
- **Agent 下一步动作选择**：工具选择、浏览器操作、游戏和机器人控制循环
- **打分与评审**：LLM-as-a-Judge 的低成本前置层、搜索重排、内容审核
- **校验与抽取**：合规检查、欺诈检测、事实性校验
- **实时决策**：全双工语音何时开口（Qwen-Audio）、安全告警分诊（网络防御）

### 9.2 推荐架构：快慢分工 + 置信度级联

```
           ┌───────────────┐
 请求 ───► │ Decision Model│── q ≥ τ ──► 直接执行（约 60–70% 流量，毫秒级、低成本）
           │  (System 1)   │
           └──────┬────────┘
                  │ q < τ
                  ▼
           ┌───────────────┐
           │  前沿 LLM     │──► 复杂推理 / 生成（System 2）
           └──────┬────────┘
                  ▼
        下游行为信号闭环评估（工单重开、用户行为等）
```

**设计要点**

1. 把复杂问题**拆成原子问题**，并行求值，再在代码里组合
2. 用业务数据**校准阈值 τ**，同时关注**覆盖率**和**已接受样本的错误率**两个指标（例如 60% 覆盖率、5% 错误率，相当于每 100 个请求错 3 个）
3. 按**尾延迟（p95/p99）**设计超时，不要只看 p50
4. 优先使用兼容 `/v1/systemone` 的接口，**方便在 Jev、自托管 Kev/Laya 和 SGLang/vLLM 之间切换**
5. 高风险决策要做偏见审计并保留人工复核

---

## 10. 趋势判断

1. **新模型品类确立**：生产系统正从"一家公司选一个模型"转向"**每个任务选一种模型**"，决策模型成为与聊天模型、嵌入模型、重排模型并列的一类基础设施。
2. **接口先于模型标准化**：`/v1/systemone` 在两周内被开源模型、SGLang 和 vLLM 兼容，正在成为决策模型的通用接口。
3. **推理框架把能力平民化**：SGLang 和 vLLM 让**任何现有 LLM** 不经训练就能当决策模型用，专用模型的护城河会转向**校准质量**和**鲁棒性**。
4. **巨头跟进，但起步较浅**：OpenAI 两周内推出了 Decisions API，但目前只是封装。接下来值得观察 Google、Anthropic 以及国内大厂是否推出**专门训练、带校准**的决策模型。
5. **扩散语言模型找到了合适的位置**：单步去噪、多槽位并行读取的特性和决策任务很契合，DiffusionGemma 可能是扩散 LLM 第一个真正落地的场景。
6. **中文与多语言评测是空白**：公开评测几乎都是英文，zh-decision-bench 是首个中文实测，国内厂商有机会在这里切入。
7. **评测体系还在形成**：覆盖率-错误率曲线、校准误差（ECE / Brier）、选项顺序鲁棒性、级联收益等，正在取代单一的准确率，成为决策模型的核心指标。

---

## 11. 参考资料

### TypeSafe Jev

- [TypeSafe AI 官方文档](https://docs.typesafe.ai/introduction)
- [TypeSafe AI & Jev — AI Engineer](https://ai.engineer/orgs/typesafe-ai)
- [Simon Willison: Jev introduces a new shape of LLM](https://simonwillison.net/2026/Sep/21/jev/)
- [TrueFoundry: TypeSafe AI's Jev — What "System One Models" Actually Are](https://www.truefoundry.com/blog/typesafe-ai-jev)
- [MindStudio: Jev Explained](https://www.mindstudio.ai/blog/jev-system-one-model-launch)
- [Beam AI: Jev by TypeSafe](https://beam.ai/agentic-insights/jev-typesafe-ai-agents)
- [DTS Solution: Jev and Cyber Defense](https://www.dts-solution.com/jevtypesafes-new-decision-model-and-what-it-means-for-cyber-defense/)

### OpenAI Decisions API

- [The New Stack: OpenAI answers TypeSafe's Jev with a Decision API built on Luna](https://thenewstack.io/openai-decision-api-luna/)
- [Neowin: OpenAI DevDay 2026 announcements](https://www.neowin.net/news/openai-unveils-500-chatgpt-pro-plan-decisions-api-and-major-codex-upgrades-at-devday-2026/)
- [Latent Space: OpenAI DevDay 2026](https://www.latent.space/p/ainews-openai-devday-2026-dots-61)
- [OrcaRouter: OpenAI's Decisions API — GPT-6 Luna Picks One Answer](https://www.orcarouter.ai/blog/openai-decisions-api-gpt-6-luna)
- [AlphaSignal: Decisions API](https://alphasignal.ai/news/openai-s-decisions-api-gives-developers-a-constrained-gpt-6-luna-router)
- [Pasquale Pillitteri: OpenAI launches Decisions API to take on Jev](https://pasqualepillitteri.it/en/news/19372/openai-decisions-api-jev)

### SGLang / vLLM / DiffusionGemma

- [LMSYS: Scaling JEV-like Decision Models with SGLang](https://www.lmsys.org/blog/2026-09-25-sglang-decision-models)
- [KuCoin News: SGLang Adds Native Decision API](https://www.kucoin.com/news/flash/sglang-adds-native-decision-api-for-llms-without-fine-tuning)
- [Red Hat Developer: Run decision models on vLLM and Red Hat AI using DiffusionGemma](https://developers.redhat.com/articles/2026/09/28/run-decision-model-vllm-and-red-hat-ai)
- [vLLM Docs: Structured reads on DiffusionGemma](https://docs.vllm.ai/en/latest/examples/features/structured_diffusion/)
- [vLLM Recipes: diffusiongemma-26B-A4B-it](https://recipes.vllm.ai/Google/diffusiongemma-26B-A4B-it)
- [GitHub: djev-dev](https://github.com/Davipar/djev-dev)

### 开源模型

- [Trilogy AI: Jev's Open Rivals Test the Decision Model Idea](https://trilogyai.substack.com/p/jev-open-decision-models)
- [GitHub: jaredpalmer/kev](https://github.com/jaredpalmer/kev)
- [GitHub: 1Panel-dev/laya-server](https://github.com/1Panel-dev/laya-server)
- [systemonemodels.org: Laya](https://systemonemodels.org/models/laya/)
- [Flowtivity: Decisions API vs Jev vs Laya](https://flowtivity.ai/blog/decisions-api-vs-jev-vs-laya/)
- [Barestack: Laya vs Jev — Self-Hosting](https://barestack.org/blog/laya-jev-self-hosted-decision-classifier)

### 评测与研究

- [arXiv 2609.26550: JEV-as-a-Judge](https://arxiv.org/html/2609.26550v2)
- [AIMultiple: AIM-Decision — Jev vs Kev vs LLMs](https://aimultiple.com/decision-models)
- [Amplitude: An analysis of Jev](https://amplitude.com/blog/jev-analysis)
- [arXiv 2605.19099: DecisionBench](https://arxiv.org/pdf/2605.19099)

### 国内

- [腾讯新闻 / 建投计算机：Jev 探索通用决策模型](https://news.qq.com/rain/a/20260921A066GH00)
- [博客园：AgentJev — 0.6B 开源决策模型](https://www.cnblogs.com/itech/p/23126319)
- [极道：开源 Kev 决策模型](https://www.jdon.com/94944-kev-decision-model-qwen-lora-prefill.html)
- [GitHub: CodyQin/zh-decision-bench](https://github.com/CodyQin/zh-decision-bench)
- [GitHub: DrTonks/qwen3-vl-decision](https://github.com/DrTonks/qwen3-vl-decision)
- [MarkTechPost: Qwen-Audio-3.1-Realtime](https://www.marktechpost.com/2026/09/28/alibaba-qwen-releases-qwen-audio-3-1-realtime-a-full-duplex-voice-model-trained-to-think-act-and-decide-when-to-speak/)
