# Next Internship Copilot — MVP 技术设计 v1.0

> **定位：** 面向中国互联网实习生的 AI 简历重构与求职助手  
> **阶段：** MVP（最小可行产品），单人开发者，6-8 周交付  
> **技术栈：** Next.js 14 (App Router) + TypeScript + PostgreSQL + Prisma + Python (DOCX导出) + Vercel  
> **最后更新：** 2026-06-21

---

## 目录

1. [MVP 范围定义](#1-mvp-范围定义)
2. [用户流程图](#2-用户流程图)
3. [数据库设计](#3-数据库设计)
4. [AI 工作流设计](#4-ai-工作流设计)
5. [关键 AI Prompt 设计](#5-关键-ai-prompt-设计)
6. [安全与合规策略](#6-安全与合规策略)
7. [DOCX 导出设计](#7-docx-导出设计)
8. [技术决策与路线图](#8-技术决策与路线图)

---

## 1. MVP 范围定义

### 1.1 What's In (MVP 必做)

| 模块 | 功能 | 优先级 |
|------|------|--------|
| **用户系统** | 手机号 + 验证码登录，Session-based Auth | P0 |
| **Mode A — Weekly Reflection** | 每周反思录入（结构化表单 + AI 追问），ERF 四阶段访谈 | P0 |
| **Mode B — Quick Export** | 已有经历直接录入，AI 量化增强 + 一键导出 DOCX | P0 |
| **AI 核心引擎** | ERF 访谈 → 素材提取 → 量化增强 → 简历生成 | P0 |
| **隐私保护** | PII 自动检测与脱敏，用户验证节点 | P0 |
| **DOCX 导出** | 中文友好排版，ATS 兼容，python-docx 生成 | P0 |
| **素材库** | 结构化存储所有实习经历片段，支持标签 + 搜索 | P1 |
| **Dashboard** | 简单的素材管理界面、导出历史 | P1 |

### 1.2 What's Out (MVP 不做)

- AI 模拟面试、多岗位简历变体、团队协作、简历评分
- 高级分析（行业对标、薪资建议）
- 文件格式多样性（仅 DOCX，不做 PDF/LaTeX）
- 移动端 App（仅 Web 响应式）

### 1.3 MVP 成功指标

- 单个用户从录入到导出 DOCX 完成时间 < 20 分钟
- AI 生成的 STAR 描述经用户确认后可直接使用的比例 > 70%
- PII 检测误报率 < 5%、漏报率 < 1%
- 导出 DOCX 在 WPS/Word 中格式无异常

---

## 2. 用户流程图

### 2.1 Mode A — Weekly Reflection 完整流程

```
用户登录 → Dashboard → 新建 Weekly Reflection
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  阶段 1: Timeline（时间线梳理）                       │
│  ───────────────────────────                        │
│  AI: "本周做了哪些主要工作？按时间线梳理"              │
│  用户: 自由文本输入（支持语音转文字预留）              │
│  AI: 识别关键节点，追问细节                           │
│  输出: 时间线节点列表 [{date, event, confidence}]     │
│                                                      │
│  [[用户验证节点 1]]: 确认时间线是否准确                │
└─────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  阶段 2: People（人物与协作）                         │
│  ───────────────────────────                        │
│  AI: "你提到和XX协作，具体怎么分工的？"                │
│  AI: "有没有跨部门协调的经历？"                        │
│  输出: 协作关系图 [{role, interaction, outcome}]      │
│                                                      │
│  [[用户验证节点 2]]: 确认协作描述是否准确              │
└─────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  阶段 3: Task（任务拆解）                             │
│  ───────────────────────────                        │
│  AI: "你主导的需求文档重构，具体做了什么？"            │
│  AI: 追问技术决策、工具选型、遇到的困难                │
│  输出: 任务详情 [{task, action, techStack, challenge}]│
│                                                      │
│  [[用户验证节点 3]]: 确认技术细节是否准确              │
└─────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  阶段 4: Impact（影响量化）                           │
│  ───────────────────────────                        │
│  AI: "优化后效果如何？有数据吗？"                      │
│  AI: 辅助推算量化指标（如从"提升了性能"到"响应时间     │
│      从 2.3s 降到 0.8s，提升 65%"）                   │
│  AI: 标注推算过程，区分「精确数据」vs「推算估计」       │
│  输出: 量化影响 [{metric, baseline, target, source}]  │
│                                                      │
│  [[用户验证节点 4]]: 确认数据准确性，标注置信度        │
└─────────────────────────────────────────────────────┘
  │
  ▼
AI 逆向生成 STAR 素材 → 素材存入素材库 → 用户选择模板 → 生成简历 → 导出 DOCX
```

### 2.2 Mode B — Quick Export 完整流程

```
用户登录 → Dashboard → 快速导出
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  步骤 1: 基础信息采集                                │
│  ───────────────────────────                        │
│  表单: 公司 | 岗位 | 时间段 | 实习内容（自由文本）    │
│  PII 检测: 实时扫描输入中的敏感信息，提醒用户脱敏     │
└─────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  步骤 2: AI 量化增强                                  │
│  ───────────────────────────                        │
│  识别用户的动作性描述 → 追问量化细节                   │
│  生成增强版 STAR 描述 → 标注 AI 修改点                │
│                                                      │
│  [[用户验证节点]]: 逐条确认或修改 AI 增强内容         │
└─────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────┐
│  步骤 3: 简历排版                                    │
│  ───────────────────────────                        │
│  选择模板 → 预览 → 调整 → 导出 DOCX                   │
└─────────────────────────────────────────────────────┘
```

### 2.3 通用状态机

```
[未登录] → [已登录] → [选择模式 A/B]
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
        [Mode A 进行中]         [Mode B 进行中]
              │                       │
              ▼                       ▼
        [ERF 四阶段完成]        [AI 增强完成]
              │                       │
              └───────────┬───────────┘
                          ▼
                    [素材已生成]
                          │
                          ▼
                    [简历已生成]
                          │
                          ▼
                    [DOCX 已导出]
```

---

## 3. 数据库设计

### 3.1 技术选型

- **数据库：** PostgreSQL 15+（生产），SQLite（本地开发可选）
- **ORM：** Prisma（TypeScript 类型安全，migration 工具成熟）
- **缓存：** Redis（仅用于验证码速率限制，可选）

### 3.2 核心表结构

#### User（用户表）

```sql
CREATE TABLE "User" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone         VARCHAR(20) UNIQUE NOT NULL,       -- 手机号（AES-256 加密存储）
  phone_hash    VARCHAR(64) UNIQUE NOT NULL,       -- phone 的 SHA-256，用于快速查找
  name          VARCHAR(100),                       -- 用户昵称
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ                         -- 软删除
);

CREATE INDEX idx_user_phone_hash ON "User"(phone_hash);
CREATE INDEX idx_user_deleted_at ON "User"(deleted_at);
```

#### VerificationCode（验证码）

```sql
CREATE TABLE "VerificationCode" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  phone_hash    VARCHAR(64) NOT NULL,
  code          VARCHAR(6) NOT NULL,
  purpose       VARCHAR(20) NOT NULL DEFAULT 'login',  -- login | export | etc.
  used          BOOLEAN NOT NULL DEFAULT FALSE,
  expires_at    TIMESTAMPTZ NOT NULL,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_vc_phone_hash ON "VerificationCode"(phone_hash, created_at DESC);
```

#### Session（会话）

```sql
CREATE TABLE "Session" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES "User"(id) ON DELETE CASCADE,
  token         VARCHAR(256) UNIQUE NOT NULL,
  expires_at    TIMESTAMPTZ NOT NULL,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_session_token ON "Session"(token);
```

#### WeeklyReflection（Mode A — 每周反思）

```sql
CREATE TABLE "WeeklyReflection" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES "User"(id) ON DELETE CASCADE,
  week_start    DATE NOT NULL,
  week_end      DATE NOT NULL,
  status        VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- draft → timeline → people → task → impact → completed
  raw_input     TEXT,                               -- 用户原始输入
  timeline_json JSONB,                              -- AI 提取的时间线 [{date, event, confidence}]
  people_json   JSONB,                              -- AI 提取的协作关系
  task_json     JSONB,                              -- AI 提取的任务详情
  impact_json   JSONB,                              -- AI 推演的量化影响
  materials     JSONB,                              -- 最终生成的 STAR 素材数组
  pii_scan_result JSONB,                           -- PII 检测结果
  confidence_score DECIMAL(3,2),                   -- 整体置信度 0.00-1.00
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ
);

CREATE INDEX idx_wr_user ON "WeeklyReflection"(user_id, created_at DESC);
```

#### QuickExport（Mode B — 快速导出）

```sql
CREATE TABLE "QuickExport" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES "User"(id) ON DELETE CASCADE,
  company       VARCHAR(200),
  role          VARCHAR(200),
  start_date    DATE,
  end_date      DATE,
  raw_input     TEXT NOT NULL,
  enhanced_json JSONB,                              -- AI 增强后的内容数组
    -- [{original, enhanced, changes[], sourceTrace[], confidence}]
  status        VARCHAR(20) NOT NULL DEFAULT 'draft',
    -- draft → enhanced → confirmed → exported
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ
);

CREATE INDEX idx_qe_user ON "QuickExport"(user_id, created_at DESC);
```

#### Material（素材库）

```sql
CREATE TABLE "Material" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES "User"(id) ON DELETE CASCADE,
  source_type   VARCHAR(20) NOT NULL,               -- 'weekly_reflection' | 'quick_export'
  source_id     UUID NOT NULL,
  content       TEXT NOT NULL,                      -- STAR 格式的素材内容
  tags          JSONB DEFAULT '[]',                 -- 自动 + 手动标签
  quantifiers   JSONB DEFAULT '[]',                 -- 量化指标 [{metric, value, confidence}]
  is_verified   BOOLEAN DEFAULT FALSE,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at    TIMESTAMPTZ
);

CREATE INDEX idx_mat_user ON "Material"(user_id, created_at DESC);
CREATE INDEX idx_mat_tags ON "Material" USING GIN (tags);
```

#### ExportHistory（导出历史）

```sql
CREATE TABLE "ExportHistory" (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID NOT NULL REFERENCES "User"(id) ON DELETE CASCADE,
  template_id   VARCHAR(50),
  material_ids  JSONB NOT NULL DEFAULT '[]',
  file_path     VARCHAR(500),
  file_size     BIGINT,
  status        VARCHAR(20) NOT NULL DEFAULT 'pending',
    -- pending → generating → completed → failed
  error_message TEXT,
  auto_delete_at TIMESTAMPTZ,                       -- 7 天后自动删除
  created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_eh_user ON "ExportHistory"(user_id, created_at DESC);
```

### 3.3 ERD

```
User 1─┬─N WeeklyReflection
      │      │
      │      └─N Material
      │
      ├─N QuickExport
      │      │
      │      └─N Material
      │
      ├─N ExportHistory
      ├─N Session
      └─N VerificationCode
```

### 3.4 设计原则

- **软删除（Soft Delete）：** 核心表均有 `deleted_at` 字段，用户删除后 30 天内可恢复
- **JSONB 灵活字段：** `timeline_json`、`people_json` 等字段使用 JSONB，适应 AI 输出结构快速迭代
- **PII 隔离：** 手机号加密存储，`phone_hash` 用于查询但不存储明文
- **无外键暴力级联：** 使用软删除 + 应用层清理，避免误删

---

## 4. AI 工作流设计

### 4.1 核心架构

```
                      ┌──────────────────────┐
                      │   AI Orchestrator    │
                      │   (API Route 层)      │
                      └──────────┬───────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  ERF Interview  │  │  Quantification │  │  Resume Builder │
│  Engine         │  │  Engine         │  │  Engine         │
│  (4-phase flow) │  │  (数据推算)      │  │  (模板渲染)      │
└─────────────────┘  └─────────────────┘  └─────────────────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                      ┌──────────┴───────────┐
                      │   Guard Layer        │
                      │  - PII Scanner       │
                      │  - Confidence Check  │
                      │  - Hallucination Det │
                      └──────────────────────┘
```

### 4.2 ERF 访谈引擎状态机

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Timeline │───▶│  People  │───▶│   Task   │───▶│  Impact  │
│ (阶段1)   │    │ (阶段2)   │    │ (阶段3)   │    │ (阶段4)   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
     │               │               │               │
     ▼               ▼               ▼               ▼
 用户确认         用户确认         用户确认         用户确认
 (验证节点1)      (验证节点2)      (验证节点3)      (验证节点4)
```

每个阶段包含以下子步骤：

1. **Analyze:** AI 分析当前阶段输入，提取关键信息
2. **Question:** AI 生成针对性追问（最多 3 个问题）
3. **UserInput:** 用户回答（文本输入）
4. **Validate:** AI 输出结构化结果 + 用户确认（可修改）
5. **Save:** 保存该阶段结果到数据库，进入下一阶段

### 4.3 量化推演引擎

```
用户输入: "我优化了接口性能"
          │
          ▼
┌────────────────────────────┐
│ Step 1: 识别可量化维度      │
│ - 时间维度: 响应时间、构建时间 │
│ - 规模维度: 用户量、数据量    │
│ - 效率维度: 人天节省、转化率  │
└────────────────────────────┘
          │
          ▼
┌────────────────────────────┐
│ Step 2: 追问基线            │
│ AI: "优化前响应时间大概多少？" │
│ 用户: "大概2秒多"            │
│ AI: "优化后呢？"             │
│ 用户: "好像不到1秒了"        │
└────────────────────────────┘
          │
          ▼
┌────────────────────────────┐
│ Step 3: 生成量化表述         │
│ "将接口响应时间从约 2.3s    │
│  优化到 0.8s，提升 65%"     │
│                             │
│ 标注:                       │
│ - source: user_estimate    │
│ - confidence: 0.7          │
│ - is_exact: false          │
└────────────────────────────┘
          │
          ▼
┌────────────────────────────┐
│ Step 4: 用户确认             │
│ ✓ 数据准确                  │
│ ✏️ 修改数据                  │
│ ⚠ 标注为估算                │
└────────────────────────────┘
```

### 4.4 反幻觉机制（Anti-Hallucination）

三层防护：

| 防护层 | 机制 | 说明 |
|--------|------|------|
| **Source Tracing** | 每个 AI 生成的素材带 `sourceTrace` | 追溯到用户的哪句话、哪次追问 |
| **Confidence Score** | 每条量化数据标注置信度 | `exact(1.0)` > `user_recall(0.85)` > `ai_estimate(0.6)` > `ai_inferred(0.4)` |
| **User Verification** | 四阶段各设验证节点 | 用户逐条确认，不确定的可标记为"待核实" |

Source Trace 数据结构示例：

```json
{
  "statement": "优化接口响应时间从2.3s降至0.8s，提升65%",
  "sourceTrace": [
    {
      "phase": "impact",
      "questionIndex": 2,
      "userInput": "优化前大概2秒多，优化后不到1秒",
      "aiInference": "根据'2秒多'估算基线为2.3s，根据'不到1秒'估算目标为0.8s"
    }
  ],
  "confidence": 0.7,
  "confidenceReason": "用户提供了估算范围，AI推算了具体数值"
}
```

---

## 5. 关键 AI Prompt 设计

### 5.1 系统级 Prompt（System Message）

```
你是 "Next Internship Copilot" 的 AI 助手。

你的角色：帮助中国互联网实习生将日常实习经历重构为专业的中文简历内容。

核心原则：
1. 绝不允许编造经历、数据、项目。一切内容必须来源于用户输入。
2. 当你需要推算数据时，必须明确标注"以下为推算，请您核实"。
3. 使用 STAR 法则 (Situation → Task → Action → Result) 组织内容。
4. 语言风格：专业但不浮夸，量化但不造数据，简洁但不省略关键动作。
5. 敏感信息（手机号、身份证号、银行卡号、家庭住址等）绝不出现在简历内容中。
6. 如果用户输入不够具体，你应该追问，而不是猜测。

你的回答格式：
- 关于事实的陈述：[用户输入来源] "根据你提到的..."
- 关于推算的数据：[推算] "根据你的描述估算为XXX，请确认"
- 关于追问的建议：[追问] "你可以补充以下信息让描述更有说服力..."
```

### 5.2 ERF 各阶段 Prompt

#### 阶段 1 — Timeline 时间线梳理

```
[System Message]

现在进入 ERF 访谈的第 1 阶段：时间线梳理。

用户的原始输入：
{raw_input}

请完成以下任务：
1. 提取用户本周（{week_start} 至 {week_end}）的主要工作节点
2. 按时间顺序排列
3. 识别出 3-5 个值得深入挖掘的关键节点
4. 对每个关键节点，生成 1 个追问

输出 JSON：
{
  "timeline": [
    {
      "date": "周一",
      "event": "参与需求评审会",
      "keyNode": false,
      "confidence": 0.9
    },
    {
      "date": "周三-周五",
      "event": "主导完成XX模块重构",
      "keyNode": true,
      "confidence": 0.95,
      "followUpQuestion": "这次重构具体改了什么？是谁发起的？"
    }
  ]
}

如果用户输入内容过少（< 100 字），先鼓励用户补充更多细节，不要强行提取。
```

#### 阶段 2 — People 人物协作

```
[System Message]

现在进入第 2 阶段：人物与协作。

基于第 1 阶段的时间线：
{timeline_json}

请完成：
1. 识别用户在工作中协作的角色（mentor、PM、其他实习生等）
2. 判断协作深度（独立执行 / 协作 / 主导推进 / 跨部门协调）
3. 生成追问，帮助用户表达协作中的"主动性"

输出 JSON：
{
  "collaborations": [
    {
      "role": "mentor",
      "interaction": "指导代码审查，提出改进建议",
      "depth": "协作",
      "followUp": "你采纳了 mentor 的哪些建议？有没有你坚持自己方案的情况？"
    }
  ]
}
```

#### 阶段 3 — Task 任务拆解

```
[System Message]

现在进入第 3 阶段：任务详情拆解。

基于前两个阶段的信息：
- 时间线：{timeline_json}
- 协作关系：{people_json}

请完成：
1. 拆解用户的关键任务为具体的动作链
2. 识别技术决策点（为什么选这个方案）
3. 挖掘困难与解决方案
4. 追问遗漏的"我做了什么"的具体细节

目标：还原用户实际动手的过程，而不是泛泛的"参与了XX项目"。

输出 JSON：
{
  "tasks": [
    {
      "title": "XX模块性能优化",
      "actions": ["发现N+1查询", "添加缓存层", "压测验证"],
      "techStack": ["Go", "Redis", "PostgreSQL"],
      "challenge": "缓存一致性需要处理",
      "solution": "使用 write-through 策略",
      "followUp": "你提到加了缓存，具体用了什么缓存策略？"
    }
  ]
}
```

#### 阶段 4 — Impact 影响量化

```
[System Message]

现在进入最后阶段：影响量化。

所有信息汇总：
- 时间线：{timeline_json}
- 协作关系：{people_json}
- 任务详情：{task_json}

请完成：
1. 对每个任务，识别可能的量化维度
2. 如果用户已提供数据 → 直接使用，标注 source=exact
3. 如果用户提供范围 → 取中值，标注 source=user_estimate
4. 如果用户未提供 → 询问用户，绝不自行编造
5. 特别注意：量化必须是 YOUR work 的结果，不能把团队/产品的整体数据当成个人贡献

输出 JSON：
{
  "impacts": [
    {
      "taskId": "xxx",
      "metric": "API 响应时间",
      "baseline": 2300,
      "baselineUnit": "ms",
      "target": 800,
      "targetUnit": "ms",
      "improvement": "65%",
      "source": "user_estimate",
      "confidence": 0.7,
      "isTeamMetric": false,
      "statement": "将XX接口响应时间从约2.3s优化至0.8s，提升65%"
    }
  ]
}
```

### 5.3 简历生成 Prompt

```
[System Message]

你是专业的中文简历撰写助手。

素材列表：
{materials_json}

模板要求：
{template_spec}

请生成简历内容。规则：
1. 每条 STAR 经历使用 [S] [T] [A] [R] 标签标注来源
2. 量化数据标注置信度：✓（用户确认）、~（AI估算）、?（待核实）
3. 每个 bullet point ≤ 2 行（在简历中展示友好）
4. 使用主动动词：主导、设计、实现、优化、推动、搭建
5. 避免空泛描述："负责XX"、"参与了XX" → 改为具体动作
6. 控制 PII 检查：移除手机号、身份证号、具体家庭地址

输出 JSON：
{
  "resume": {
    "summary": "具有XX背景的XX实习生，在X个月内...",
    "experiences": [
      {
        "company": "XX科技",
        "role": "后端开发实习生",
        "duration": "2025.06 - 2025.09",
        "bullets": [
          {
            "content": "主导XX模块重构，将接口响应时间从~2.3s优化至0.8s（提升65%）",
            "starTags": ["S", "A", "R"],
            "confidence": "~",
            "sourceMaterialId": "uuid"
          }
        ]
      }
    ],
    "piiWarnings": []
  }
}
```

### 5.4 PII 检测 Prompt

```
[System Message]

你是隐私保护检测器。扫描以下文本中的所有敏感信息：

{text}

检测规则：
- 手机号：1[3-9]\d{9}
- 身份证号：\d{17}[\dXx]
- 邮箱：[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
- 银行卡号：\d{16,19}
- 家庭地址：省市区 + 具体到门牌号
- 真实姓名 + 手机号组合
- 内部系统 URL 或 IP 地址

输出 JSON：
{
  "findings": [
    {
      "type": "phone",
      "match": "13812345678",
      "position": { "start": 42, "end": 53 },
      "suggestion": "脱敏为 138****5678 或删除"
    }
  ],
  "scanSummary": {
    "totalFindings": 2,
    "byType": { "phone": 1, "email": 1 },
    "recommendedAction": "review_and_redact"
  }
}
```

---

## 6. 安全与合规策略

### 6.1 PII 保护全景

```
用户输入
  │
  ▼
┌──────────────────────────┐
│  实时 PII 扫描            │  ← 前端：正则快速扫描，提示用户
│  (Client-side)           │
└──────────────────────────┘
  │
  ▼
┌──────────────────────────┐
│  AI PII 检测              │  ← 后端：LLM 深度检测
│  (Server-side)           │     上下文感知（如"我家住在..."）
└──────────────────────────┘
  │
  ▼
┌──────────────────────────┐
│  自动脱敏                 │  ← 生成简历时自动替换/删除
│  (Auto-redact)           │
└──────────────────────────┘
  │
  ▼
┌──────────────────────────┐
│  用户审核                 │  ← 导出前展示脱敏结果
│  (User Review)           │
└──────────────────────────┘
```

### 6.2 PIPL 合规要点

| 条款 | 实现方式 |
|------|---------|
| **最小必要** | 仅收集手机号用于登录，不要求身份证实名 |
| **知情同意** | 首次使用弹出隐私政策摘要，明确告知数据用途 |
| **数据安全** | 手机号 AES-256 加密存储，数据库连接 TLS |
| **删除权** | 用户可随时删除账号，30 天软删除后物理清除 |
| **数据本地化** | 全部数据存储在中国大陆服务器（阿里云/腾讯云 PostgreSQL） |

### 6.3 文件安全

- **验证码频率限制：** 同一手机号 60s 内只能发送 1 次，每小时最多 5 次
- **Session 管理：** JWT + HttpOnly Cookie，7 天过期
- **CSRF 防护：** SameSite=Strict Cookie
- **API 限流：** 登录/注册接口 10 req/min/IP
- **文件自动清除：** DOCX 文件生成后保留 7 天，到期自动删除

### 6.4 数据流安全

```
[浏览器] ──HTTPS──▶ [Next.js Server] ──TLS──▶ [PostgreSQL]
                             │
                             ├──▶ [AI API (国内合规服务)]
                             │      - 讯飞星火 / 阿里通义千问
                             │      - 不在境外传输用户数据
                             │
                             └──▶ [Python DOCX Service]
                                    - 本地/同 VPC 部署
                                    - 生成后即清理临时文件
```

### 6.5 AI 服务选型

由于目标用户为中国大陆实习生，优先选择国内合规的 AI 服务：

| 服务 | 优势 | 注意事项 |
|------|------|---------|
| **通义千问 (Qwen)** | 中文能力强，阿里云生态 | 需要 ICP 备案 |
| **DeepSeek** | 性价比高，开源可自部署 | 适合自建需求 |
| **讯飞星火** | 教育场景认证，合规 | API 成熟度略低 |

MVP 阶段建议使用 **DeepSeek API**，成本低、中文质量高、有开源模型可备选。

---

## 7. DOCX 导出设计

### 7.1 技术选型

- **方案：** Python 脚本 + `python-docx` 库
- **触发方式：** Next.js 调用 Python 子进程，传入 JSON 数据
- **为什么不直接用 JS 生成？** python-docx 对中文排版（字体、行距、缩进）的成熟度远超 JS 生态的 docx 库

### 7.2 数据流

```
Next.js API Route
  │
  ├─ 1. 收集素材数据
  ├─ 2. 应用模板配置
  ├─ 3. 序列化为 JSON
  │
  ▼
Python 子进程
  │
  ├─ 4. 解析 JSON
  ├─ 5. 创建 Document
  ├─ 6. 应用样式（字体、行距、边距）
  ├─ 7. 填充内容
  ├─ 8. 保存 .docx
  │
  ▼
Next.js
  │
  ├─ 9. 返回文件下载链接
  └─ 10. 7 天后自动清理
```

### 7.3 模板 JSON 描述

```json
{
  "template": "classic_cn",
  "font": {
    "name": "微软雅黑",
    "size": 10.5,
    "titleSize": 16,
    "headingSize": 12
  },
  "layout": {
    "margin": { "top": 2.54, "bottom": 2.54, "left": 3.18, "right": 3.18 },
    "lineSpacing": 1.5,
    "sectionSpacing": 12
  },
  "sections": [
    {
      "type": "header",
      "fields": ["name", "phone", "email", "targetRole"]
    },
    {
      "type": "summary",
      "label": "个人总结",
      "maxLines": 3
    },
    {
      "type": "experiences",
      "label": "实习经历",
      "itemTemplate": {
        "company": { "bold": true, "size": 11 },
        "role": { "size": 10.5 },
        "duration": { "size": 9, "color": "gray" },
        "bullets": { "size": 10.5, "bulletStyle": "•" }
      }
    },
    {
      "type": "education",
      "label": "教育背景"
    },
    {
      "type": "skills",
      "label": "专业技能",
      "layout": "tags"
    }
  ]
}
```

### 7.4 ATS 兼容性

确保生成的 DOCX 能被主流 ATS 系统（如 Greenhouse, Lever, 北森, Moka）正确解析：

- **表格不用合并单元格：** ATS 经常读错合并单元格
- **不使用文本框/艺术字：** 无法被 ATS 解析
- **关键词自然分布：** 不在简历中堆砌隐藏关键词（ATS 新算法会降权）
- **标准节标题：** "实习经历"、"教育背景"、"专业技能" 等标准中文标题
- **文本可选：** 不把关键内容放在图片中

---

## 8. 技术决策与路线图

### 8.1 关键架构决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 前端框架 | Next.js 14 App Router | SSR 对 SEO 友好（未来 Landing Page），API Routes 简化全栈开发 |
| 语言 | TypeScript | 类型安全，减少 AI 生成代码的 Bug |
| 数据库 | PostgreSQL + Prisma | 关系型数据天然适合，JSONB 提供灵活性，Prisma 类型安全 |
| AI 服务 | DeepSeek API | 中文最优，成本可控，可选开源自部署 |
| DOCX 生成 | Python (python-docx) | 中文排版最成熟 |
| 部署 | Vercel + 阿里云 RDS | Vercel 免费额度足够 MVP，数据库独立部署 |
| 认证 | 手机号 + 验证码 | 中国大陆最普遍的登录方式 |

### 8.2 MVP 开发路线图（6-8 周）

| 周 | 里程碑 | 交付物 |
|----|--------|--------|
| **W1** | 项目初始化 + 数据库 | Prisma Schema、Migration、Seed 脚本 |
| **W2** | 用户认证 | 手机号登录/注册、Session 管理 |
| **W3** | Mode B — Quick Export | 表单录入 → AI 增强 → 预览 |
| **W4** | Mode A — Weekly Reflection (Timeline + People) | ERF 前两个阶段 |
| **W5** | Mode A — Weekly Reflection (Task + Impact) | ERF 后两个阶段 + STAR 生成 |
| **W6** | PII 检测 + DOCX 导出 | 安全层 + Python 导出服务 |
| **W7** | Dashboard + 素材库 | 素材管理、导出历史 |
| **W8** | 打磨 + 上线 | Bug 修复、性能优化、文档 |

### 8.3 技术风险与缓解

| 风险 | 影响 | 缓解策略 |
|------|------|---------|
| AI 幻觉导致用户简历失真 | 高 | 三层反幻觉机制、用户验证节点、Source Trace |
| AI API 不稳定/限流 | 中 | 请求队列 + 重试 + 降级提示 |
| python-docx 中文兼容问题 | 中 | 充分测试 WPS/Word/Pages，准备 HTML→DOCX 备选 |
| PII 泄漏 | 高 | 前端 + AI 双重扫描，自动脱敏，不存储明文 |
| 单点部署风险 | 低 | MVP 阶段接受，后期做多可用区 |

### 8.4 成本估算

| 资源 | 月度估算 (RMB) | 说明 |
|------|---------------|------|
| Vercel (Hobby) | 0 | 个人项目免费额度足够 |
| 阿里云 RDS PostgreSQL | ~200-400 | 基础版 1C1G，20GB 存储 |
| DeepSeek API | ~50-200 | 按 token 计费，MVP 阶段用量低 |
| 域名 + 备案 | ~60/年 | .cn 域名 |
| **月度总计** | **~300-600** | |

---

## 附录 A：API 路由设计（草案）

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/api/auth/send-code` | 发送验证码 |
| POST | `/api/auth/login` | 验证码登录 |
| POST | `/api/auth/logout` | 登出 |
| GET | `/api/reflections` | 获取所有 Weekly Reflection |
| POST | `/api/reflections` | 创建新的 Weekly Reflection |
| PATCH | `/api/reflections/:id` | 更新 Reflection（各阶段数据） |
| GET | `/api/exports` | 获取所有 Quick Export |
| POST | `/api/exports` | 创建 Quick Export |
| PATCH | `/api/exports/:id/enhance` | 触发 AI 增强 |
| GET | `/api/materials` | 获取素材库 |
| POST | `/api/generate-resume` | 生成简历 |
| GET | `/api/download/:id` | 下载 DOCX |

## 附录 B：Prisma Schema 预览

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                String              @id @default(uuid())
  phone             String              @unique
  phoneHash         String              @unique
  name              String?
  createdAt         DateTime            @default(now())
  updatedAt         DateTime            @updatedAt
  deletedAt         DateTime?
  sessions          Session[]
  verificationCodes VerificationCode[]
  reflections       WeeklyReflection[]
  exports           QuickExport[]
  materials         Material[]
  exportHistory     ExportHistory[]
}

// ... (其余 model 定义)
```

---

> **文档状态：** 待审核  
> **下一步：** 产品负责人审批后进入 W1 开发  
> **作者：** AI + 用户协作生成
