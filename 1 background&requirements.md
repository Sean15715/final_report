# ScoutAgent 选题背景 · PPT 速览版

> **从招聘行业的真实痛点，到 Agent 技术方案的反向推演**
> 每页一个核心矛盾 + 一句话结论

---

## 开场：为什么是"招聘 + Agent"

招聘是一个**信息密度极高、判断成本极重、容错率极低**的场景：

```mermaid
flowchart LR
    JD[📄 岗位 JD<br/>结构化要求] -.对齐.-> CV[📋 候选人 CV<br/>非结构化叙述]
    CV --> HR{👤 HR}
    JD --> HR
    HR --> Decide([❓ 是否进入下一轮])

    style HR fill:#fff59d
    style Decide fill:#ffccbc
```

> **一句话**：HR 每天都在用"人脑"做"大模型该做的活"——在海量异构文本里找匹配、辨真伪、做综合判断。
> 这正是 LLM Agent 最该被使用的场景。

---

# 一、行业痛点全景

## 1.1 招聘流程里的"四座大山"

```mermaid
flowchart TB
    Pool[📥 简历池<br/>每岗位 100~1000 份] --> P1[🔍 痛点 1<br/>JD-CV 匹配低效]
    P1 --> P2[🕵️ 痛点 2<br/>简历真实性存疑]
    P2 --> P3[💻 痛点 3<br/>技术能力难量化]
    P3 --> P4[📊 痛点 4<br/>综合评估靠"感觉"]
    P4 --> Out([最终录用决策])

    style P1 fill:#ffcdd2
    style P2 fill:#ffcdd2
    style P3 fill:#ffcdd2
    style P4 fill:#ffcdd2
```

| 阶段 | HR 实际做的事 | 主要痛点 |
|---|---|---|
| 初筛 | 关键词搜简历 | 同义词/技术栈迁移识别不到 |
| 深筛 | 看项目经历 | 包装、虚标、年限注水难辨 |
| 技评 | 出题、看代码 | 题目同质化、判分主观 |
| 终评 | 拍板、写理由 | 多轮信息散落、难复盘 |

---

## 1.2 痛点一：关键词匹配 ≠ 能力匹配

**HR 现状**：用 ATS 系统按关键词过滤，结果"会写 Golang 的"和"5 年 Go 后端经验"被同等对待。

```mermaid
flowchart LR
    JD["JD：要求<br/>K8s + 微服务"]
    CV1["CV-A：精通 Kubernetes<br/>设计过服务网格"]
    CV2["CV-B：用过 K8s<br/>跑过 Docker"]

    JD -- 关键词命中 --> CV1
    JD -- 关键词命中 --> CV2

    CV1 -.真实匹配.-> Hit([✅ 高匹配])
    CV2 -.真实匹配.-> Miss([⚠️ 低匹配])

    style Hit fill:#c8e6c9
    style Miss fill:#ffccbc
```

**根因**：
- 技术栈有**同义词**（K8s ≡ Kubernetes）
- 技术栈有**上下位关系**（Spring Cloud ⊂ 微服务）
- 技术栈有**迁移能力**（会 Java 后端 → 大概率能上手 Go 后端）

> 纯文本匹配天然处理不了"语义 + 结构"的双重对齐。

---

## 1.3 痛点二：简历"通用化包装"已成产业

**现状**：市面充斥"简历优化教程"，候选人会针对 JD 反向定制简历。

```mermaid
flowchart TB
    JD[岗位 JD] --> Tmpl[简历模板/优化服务]
    Tmpl --> CV[📋 高度对齐的 CV]
    CV --> HR{HR 单看 CV}
    HR -- 看不出问题 --> Pass[进入面试]
    Pass -- 面试才发现 --> Fail([❌ 名实不符])

    style Tmpl fill:#fff59d
    style Fail fill:#ffccbc
```

**典型包装手法**：
- **职责膨胀**：把团队成果写成个人主导
- **技术堆砌**：列一堆框架但说不清深度
- **时间含糊**：用"参与/负责/主导"模糊真实贡献
- **项目挪用**：拿开源项目当自研经历

> 单点信息无法证伪，必须**跨字段交叉验证**才能识破。

---

## 1.4 痛点三：技术能力评估靠"题海 + 直觉"

**现状**：刷题站答题、白板题、八股文——离真实工程能力越来越远。

```mermaid
flowchart LR
    Test[传统笔试题] --> Score[得分]
    Score --> Q{能反映工程能力?}
    Q -- ❌ --> Issue1[只考算法]
    Q -- ❌ --> Issue2[死记硬背可作弊]
    Q -- ❌ --> Issue3[评分粒度只到对错]

    style Issue1 fill:#ffccbc
    style Issue2 fill:#ffccbc
    style Issue3 fill:#ffccbc
```

**HR 真正想知道的是**：
- 候选人**怎么思考**（解题路径）
- 代码**结构和规范**（不只是 AC）
- **被追问后的反应**（真懂还是背的）

> 现有平台只给"通过率"，给不了"过程评估"。

---

## 1.5 痛点四：多轮评估信息"碎片化"

**现状**：初筛 Excel、面试官口头反馈、技术评分表、HR 邮件——决策时拼不起来。

```mermaid
flowchart TB
    Step1[初筛<br/>📑 Excel] -.散.-> Decision
    Step2[一面<br/>📝 评分表 A] -.散.-> Decision
    Step3[二面<br/>🗣 口头反馈] -.散.-> Decision
    Step4[技评<br/>📊 平台分数] -.散.-> Decision
    Step5[HR 沟通<br/>📧 邮件] -.散.-> Decision

    Decision{🤔 综合判断}
    Decision -- 全靠脑补 --> Out([决策])

    style Decision fill:#ffccbc
```

**带来的代价**：
- 复盘困难，**为什么淘汰**说不清
- 经验**留不下**，HR 离职即失忆
- 同公司**不同岗位**的偏好无法沉淀

> 招聘是一个长链路决策过程，需要**结构化、可追溯、可复用**的信息载体。

---

# 二、从痛点到选题：反向推演

## 2.1 痛点 ↔ 技术映射

```mermaid
flowchart LR
    P1[🔍 关键词失效] --> T1[GraphRAG +<br/>技术知识图谱]
    P2[🕵️ 简历包装] --> T2[多源交叉验证<br/>真实性 Agent]
    P3[💻 能力难量化] --> T3[代码执行 +<br/>过程评估]
    P4[📊 信息碎片化] --> T4[Planning +<br/>结构化报告]
    P5[🧠 经验难沉淀] --> T5[长期偏好<br/>记忆]

    style T1 fill:#bbdefb
    style T2 fill:#bbdefb
    style T3 fill:#bbdefb
    style T4 fill:#bbdefb
    style T5 fill:#bbdefb
```

| 行业痛点 | 选型方案 | 解决思路 |
|---|---|---|
| 同义词 / 上下位概念 | **Neo4j 知识图谱 + GraphRAG** | 把"技术栈"建成图，用结构推理代替字符串匹配 |
| 包装和虚标 | **真实性检测 Agent** | 让 LLM 跨"JD / CV / 项目 / 测试"四源比对 |
| 能力评估粗糙 | **代码执行工具 + 追问能力** | Agent 调工具拿运行结果，结合解题过程评分 |
| 决策碎片化 | **Plan-and-Execute + 报告生成** | 拆解评估流程，最终输出结构化 Markdown |
| 经验流失 | **企业级 / 岗位级偏好库** | LLM 自动识别硬性要求并 upsert 到 PG |

---

## 2.2 为什么必须是"Agent"，而不是单次 LLM 调用

```mermaid
flowchart LR
    Naive[单次 Prompt<br/>JD + CV → 评分] --> Bad{够用吗?}
    Bad -- ❌ --> R1[不能调外部知识图谱]
    Bad -- ❌ --> R2[不能跑代码验证]
    Bad -- ❌ --> R3[不能记住历史偏好]
    Bad -- ❌ --> R4[不能做多步推理]

    R1 & R2 & R3 & R4 --> Need[需要 Agent]
    Need --> Sol[Function Calling +<br/>Memory + Planning]

    style Sol fill:#c8e6c9
```

**招聘任务的特征决定了它必须是 Agent**：
- 需要**外部能力**（图谱、代码沙箱、第三方系统）→ Function Calling
- 需要**跨会话记忆**（这家客户的偏好、这个岗位的硬性要求）→ Memory
- 需要**多步骤拆解**（先匹配 → 再验真 → 再技评 → 再综合）→ Planning

> 单次问答只能解决"问题"，Agent 才能解决"流程"。

---

## 2.3 选题立意：不做"AI 简历筛选器"，做"AI HR 助手"

```mermaid
flowchart LR
    subgraph Old[市面常见方案]
        O1[关键词过滤]
        O2[一次性打分]
        O3[黑盒输出]
    end
    subgraph New[ScoutAgent 定位]
        N1[语义+结构匹配]
        N2[多阶段交叉验证]
        N3[全过程可追溯]
    end

    Old -.替代 → 不是.-> HR1[替代 HR]
    New -.增强 → 才是.-> HR2[增强 HR]

    style Old fill:#ffccbc
    style New fill:#c8e6c9
```

| 维度 | 传统筛选工具 | ScoutAgent |
|---|---|---|
| **目标** | 提升处理速度 | 提升判断质量 |
| **输出** | 一个分数 | 一份带证据链的报告 |
| **对 HR** | 被取代焦虑 | 协作式增强 |
| **可解释** | 黑盒 | 计划 + 工具调用全程可见 |
| **可沉淀** | 无 | 偏好持续学习 |

---

# 三、选题价值

## 3.1 三层价值闭环

```mermaid
flowchart TB
    Tech[🔧 技术价值<br/>Agent 工程范式落地] --> Biz
    Biz[💼 业务价值<br/>降本提效 + 提高匹配质量] --> Soc
    Soc[🌐 社会价值<br/>降低信息不对称<br/>保护求职者公平]

    style Tech fill:#bbdefb
    style Biz fill:#c8e6c9
    style Soc fill:#fff59d
```

| 层面 | 价值点 |
|---|---|
| **技术** | 在真实业务里同时验证 Function Calling / Memory / Planning 三大模块的协同 |
| **业务** | HR 每岗位评估时间从"几小时/份"压缩到"几分钟/份"，且产出结构化报告 |
| **社会** | 减少"靠包装上岗"，让真实能力被看见；减少"靠学历过滤"，给非典型背景候选人机会 |

---

## 3.2 一句话收束

```mermaid
flowchart LR
    Pain[招聘行业的<br/>四类核心痛点] --> Why[需要一个<br/>会调工具 / 有记忆 / 会规划<br/>的智能体]
    Why --> What[ScoutAgent<br/>= 招聘领域的 LLM Agent 落地]
    What --> Value[让 HR 把时间<br/>花在判断上，而不是搜索上]

    style What fill:#c8e6c9
    style Value fill:#fff59d
```

> **选题意图一句话**：
> 用 Agent 把招聘流程里"最重复、最易错、最难沉淀"的部分自动化，让 HR 回到"做决策"的位置。
