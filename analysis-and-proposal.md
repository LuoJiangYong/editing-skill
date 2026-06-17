# Editing Skill 架构分析与简化建议

> 分析对象：`ai-tech-news-editor-architecture-v1.11.md`（4677 行）
> 方法论：Hermes Skill 开发方法论（通用框架和可复用模式）
> 日期：2026-06-17

---

## 一、旧架构诊断

### 1.1 问题总览

v1.11 架构文档长达 4677 行，规划了 **8 个 Phase**、**100+ 文件**，覆盖内容生产、平台发布、插件维护三大子系统。这是一个试图在第一天就做到"完美"的设计——但违反了方法论的核心原则：**"<3 个消费者就不抽象"** 和 **"非必需不复杂化"**。

### 1.2 具体问题

| # | 问题 | 严重程度 | 说明 |
|---|------|---------|------|
| 1 | **过度抽象 data provider 层** | 🔴 高 | TrendRadar 被深度绑定为默认 provider，同时又设计了一套复杂的 provider 契约层来"解耦"它。实际上目前只有一个 provider，抽象层是净负担。 |
| 2 | **三层内容模型过重** | 🔴 高 | `content_items` → `content_blocks` → `body_markdown` 三层，加 `references`、`scoring`、`publishing_map`、`publishing.targets`。Agent 生成和维护的成本远大于收益。 |
| 3 | **平台发布子系统冗余** | 🔴 高 | 信息图（infographic）和微信公众号 HTML 是独立 Skill 的职责（已有 infographic-carousel、HyperFrames 等）。编辑 Skill 不应承接发布职能。 |
| 4 | **维护元数据系统过早** | 🟡 中 | Phase 2A + Phase 7 规划了 meta-schemas、field registry、dependency map、validation matrix、8 个 scan 脚本。一个还未开发的 Skill 不需要维护自动化。 |
| 5 | **正文结构模板过于精细** | 🟡 中 | 通用信号块定义了 15+ 种（block_time_scope、block_source_coverage、block_trend_curve...），每种内容类型又有 10-20 个特定块。Agent 难以在上下文中保持一致性。 |
| 6 | **发布映射层冗余** | 🟡 中 | `publishing_map` 作为 content_blocks → 平台素材的中间层，增加了两层确认关卡。对 Agent 执行来说，直接从确认后的 content_blocks 生成平台素材更直接。 |
| 7 | **TrendRadar 耦合过深** | 🔴 高 | 整个数据获取层、评分层、正文结构层的工具映射都以 TrendRadar 为基准。`analyze_sentiment`、`analyze_topic_trend`、`compare_periods` 等专属工具被硬编码到数据源信号块映射表中。 |
| 8 | **文件数量爆炸** | 🔴 高 | 按 8 Phase 规划需要创建 100+ 文件。即使分 session 开发，维护成本也会随文件数平方增长。方法论建议 L3 级别典型文件数是 30-40（含案例库）。 |

### 1.3 方法论对照

| 方法论原则 | 旧架构状态 | 差距 |
|-----------|----------|------|
| 高内聚 — One file, one job | ❌ 信号块定义散落在 5 个结构模板中 | 需收敛 |
| 低耦合 — 改一个不动其他 | ❌ provider 字段映射影响评分→结构→发布三层 | 需简化 |
| 可扩展 — 新能力 = 新文件 + 一行注册 | ⚠️ 形式上符合，但注册表本身是维护负担 | 可接受 |
| 易维护 — 可追溯性好过聪明 | ❌ 元数据体系本身变成维护对象 | 需删除 |
| 高效简洁 — <3 个消费者就不抽象 | ❌ provider 契约只有 1 个消费者 | 严重违反 |
| SKILL.md ≤ 3500 chars | ⚠️ 架构文档描述的内容远超 3500 chars | 需压缩 |
| 路由中枢 + 辐射引用 | ❌ 几乎所有内容都内嵌在架构文档中 | 需分离 |
| Phase 分阶段隔离 | ⚠️ 8 个 Phase 过多 | 建议 3 Phase |

---

## 二、简化方案

### 2.1 核心原则（从方法论裁剪）

```
1. 内容生产是核心 — 编辑 Skill 只管从信息到成稿
2. 发布交给别人 — 不内建信息图、HTML 生成
3. 数据来源弱绑定 — 用户提供材料，Skill 不主动调用 TrendRadar
4. 两层模型够用 — content_items（素材） + body_markdown（成稿），去掉 content_blocks 中间层
5. < 3 个消费者不抽象 — 当前只有 1 个 provider，不做 provider 契约层
```

### 2.2 定位变更

| 维度 | v1.11 | 简化版 |
|------|-------|--------|
| **核心产出** | content-package（三层 model + 发布计划） | 结构化的 Markdown 文章 + 来源清单 |
| **数据来源** | TrendRadar（默认）+ WebSearch（验证）+ 可替换 provider | 用户提供的搜索结果/链接/材料，Agent 用 `web_search` + `browser` 自主补充 |
| **发布对接** | 内建 infographic + WeChat HTML workflow | 产出标准 Markdown，下游 Skill 自行消费 |
| **确认关卡** | 4 层（信息获取→内容编辑→发布映射→平台发布） | 2 层（信息确认→内容确认） |
| **维护体系** | metadata/* + meta-schemas + scripts | 无——Hermes skill 工具链负责 |

### 2.3 简化后的 Skill 边界

**Editing Skill 负责：**
- 接收用户提供的原始材料（搜索结果、链接、RSS 条目、粘贴文本）
- 按 5 种内容类型组织信息：日报/周报/月报/专题/内容建议
- 应用编辑角色进行筛选、评分、排序
- 生成结构化文章（Markdown + 来源清单 + 元数据）
- 两轮确认：信息确认 → 内容确认

**Editing Skill 不负责：**
- 主动调用 TrendRadar/热点发现工具（用户自己提供数据，或由其他 Skill/cron job 采集）
- 生成信息图（交给 `infographic-carousel` skill）
- 生成微信公众号 HTML（交给 `hyperframes` 或专用发布 skill）
- 维护自身的元数据体系（交给 Hermes 的 skill 管理工具）

### 2.4 简化后的数据模型

```
content-package (简化至两层):
├── metadata        # 标题、日期、类型、关键词
├── content_items[] # 素材层（候选、入选、背景、剔除）
│   ├── title, summary, url, source_platform
│   ├── score（单层评分：热度/权威/编辑价值，各 1-5）
│   └── status: candidate | selected | background | rejected
├── body_markdown   # 完整的 Markdown 正文（含引用标注）
├── references[]    # 来源清单（名称、URL、类型、可信度）
└── editorial_notes # 编辑手记（取舍理由、风险提示）
```

**关键简化：**
- ❌ 删除 `content_blocks` 中间层——Agent 直接从 content_items 生成 Markdown
- ❌ 删除 `publishing_map`——不需要了
- ❌ 删除 `publishing.targets`——发布不是本 Skill 的事
- ❌ 删除三层评分（Provider Quant / Source Quality / Editorial Value）→ 合并为单层 1-5 分
- ❌ 删除 `package_version` / `references_data_version`——对编辑 Skill 过度设计
- ✅ 保留 `content_items` + `body_markdown`——够用

### 2.5 文件数量对比

| 类别 | v1.11 | 简化版 | 说明 |
|------|-------|--------|------|
| SKILL.md | 1 | 1 | — |
| references/ | 8 | 4 | 编辑角色、评分标准、5 类内容指南、来源规范 |
| schemas/ | 7 | 1 | 仅 content-package 结构说明 |
| templates/ | 16 | 6 | 5 类内容的结构指南 + 1 个通用 Markdown 模板 |
| workflows/ | 10 | 5 | 5 类内容 workflow（仅内容生产，无发布） |
| publishing/ | 6 | 0 | **全部删除** |
| metadata/ | 10 | 0 | **全部删除** |
| scripts/ | 8 | 0 | **全部删除** |
| examples/ | 15 | 2 | 一个日报样例 + 一个专题样例 |
| **合计** | **~100+** | **~19** | **减少 80%+** |

### 2.6 TrendRadar 弱化策略

**当前状态：** TrendRadar 被深度绑定在架构的每一层：
- `references/trendradar-tools.md` — 11 个工具的字段映射表
- 15 个通用信号块直接映射 TrendRadar 工具
- 5 个 content workflow 都硬编码了 TrendRadar 工具列表
- 评分层的 Provider Quant Score 依赖 TrendRadar 字段

**弱化方案：**

1. **不在 Skill 中硬编码任何 TrendRadar 调用**
2. **数据入口改为"用户提供材料"**：用户粘贴搜索结果、链接、RSS 摘要
3. **如果用户要求自动采集**，SKILL.md 只给出通用指引：
   ```
   如果用户要求自动获取热点：
   - 可用 web_search 检索当日 AI/科技新闻
   - 可浏览 Hacker News、ProductHunt、TechCrunch 等站点
   - 如已配置 TrendRadar MCP，可用其工具辅助，但不依赖
   ```
4. **评分简化为人工可理解的 1-5 分**，不映射 provider 字段
5. **删除 `references/trendradar-tools.md`** 和 `references/data-providers.md`，合并为一个简短的 `references/data-sources.md`，只说明通用数据源使用规范

---

## 三、实施建议

### 3.1 Phase 划分（从 8 → 3）

| Phase | 内容 | 文件数 | 独立 Session |
|-------|------|--------|-------------|
| **0** | CONSTITUTION.md + content-package schema + SKILL.md 骨架 | 3 | ✅ |
| **1** | references/（编辑角色、评分、内容类型指南、数据源规范）+ templates/（5 类结构指南 + Markdown 模板） | 10 | ✅ |
| **2** | workflows/（5 个内容 workflow）+ examples/（日报 + 专题样例） | 7 | ✅ |

### 3.2 Phase 0 — 宪法 + Schema + 路由骨架

**文件清单：**
```
editing-skill/
├── CONSTITUTION.md              # 5 条设计原则 + 目录布局 + 禁止模式
├── SKILL.md                     # 路由中枢（≤3500 chars 正文）
└── schemas/
    └── content-package.md       # 简化后的两层数据模型
```

**CONSTITUTION.md 核心内容：**
- 5 条原则（用 Editing Skill 领域词替换）
- 目录布局
- 禁止事项：
  - ❌ 不主动调用 TrendRadar 或其 MCP 工具
  - ❌ 不生成信息图/HTML（交给其他 Skill）
  - ❌ 不杜撰新闻、链接、数据
  - ❌ 不跳过确认关卡
  - ❌ 不用三层模型，只用两层（items + markdown）

**SKILL.md 路由决策树（简版）：**
```
用户需求
  ├─ "日报/今天有什么新闻" → workflows/daily-news.md
  ├─ "周报/本周热点"       → workflows/weekly-hotspots.md
  ├─ "月报/本月回顾"       → workflows/monthly-hotspots.md
  ├─ "分析/深挖 XX"       → workflows/topic-deep-dive.md
  └─ "推荐选题/有什么可写" → workflows/content-advice.md
  
所有路径 → 加载 references/editorial-roles.md（按需匹配）
         → 加载 references/scoring.md
         → 信息确认关卡 → 内容编辑 → 内容确认关卡 → 输出 content-package
```

### 3.3 Phase 1 — 引用文档 + 模板

**文件清单：**
```
references/
├── editorial-roles.md           # 3 个编辑角色
├── scoring.md                   # 简化的 1-5 分评分体系
├── content-types-guide.md       # 5 类内容的特征、结构、篇幅
└── data-sources.md              # 数据源使用规范（弱化 TrendRadar）

templates/
├── daily-news-structure.md      # 日报结构指南
├── weekly-hotspots-structure.md # 周报结构指南
├── monthly-hotspots-structure.md# 月报结构指南
├── topic-deep-dive-structure.md # 专题结构指南
├── content-advice-structure.md  # 内容建议结构指南
└── markdown-body.md             # 通用 Markdown 渲染规范
```

**scoring.md 简化评分：**
```markdown
## 评分维度（每项 1-5 分）
1. 热度/关注度 — 是否被广泛讨论（5=全网刷屏，1=极少数人关注）
2. 来源权威性 — 来源是否可靠（5=官方/顶会/顶刊，1=匿名/不可考）
3. 编辑价值 — 对目标读者的价值（5=必读，1=可忽略）

## 总分计算
总分 = 热度×0.4 + 权威×0.3 + 编辑价值×0.3

## 缺失数据处理
- 热度无法判断时标 `unknown`，总分降置信度
- 来源缺失时最多给 2 分
- 编辑价值始终由 Agent 判断
```

### 3.4 Phase 2 — 工作流 + 样例

**文件清单：**
```
workflows/
├── daily-news.md
├── weekly-hotspots.md
├── monthly-hotspots.md
├── topic-deep-dive.md
└── content-advice.md

examples/
├── daily-news-example.md
└── topic-deep-dive-example.md
```

### 3.5 开发优先级

```
Week 1: Phase 0（宪法 + Schema + SKILL.md）
  → 验证：SKILL.md 能正确路由到 5 个 workflow（即使 workflow 还是空壳）

Week 2: Phase 1（references + templates）
  → 验证：编辑角色能匹配不同内容类型，评分能给出合理解释

Week 3: Phase 2（workflows + examples）
  → 验证：完整跑通"用户提供材料 → 日报生成 → 确认 → 输出"
```

### 3.6 旧架构中值得保留的设计

并非全盘否定 v1.11。以下设计是好的，保留：

| 设计 | v1.11 做法 | 简化版做法 |
|------|----------|----------|
| 确认关卡机制 | 4 层确认 | 精简到 2 层（信息→内容），保留影响范围判定 |
| 编辑角色匹配 | `use_for` + `applies_to` | 保留匹配机制，精简角色到 3 个 |
| 内容类型判断 | 5 种类型路由 | 保留，路由表放 SKILL.md |
| 事实不可杜撰 | 全文多处强调 | 保留为核心禁令 |
| 中文输出优先 | 全文强调 | 保留 |
| 来源展示格式 | `source_platform.display` | 保留在 content_items 中 |
| content_items.type | 15 种类型枚举 | 保留，Agent 按需选用 |

---

## 四、总结

### 旧架构的根因

v1.11 是一个 **"先设计宇宙再写一行代码"** 的产物。它试图：
1. 用一套 provider 契约屏蔽未来可能的数据源变更（但当前只有 1 个 provider）
2. 用三层模型覆盖从信息到发布的全链路（但 Agent 实际只需要两层）
3. 用全套维护元数据体系防止未来字段失控（但文件还没创建）

这违反了方法论的 **"高效简洁 LEAN — <3 个消费者就不抽象"** 和 **"非必需不复杂化"**。

### 简化版的核心思路

**"先跑通核心链路，再按需扩展"**

1. 两层模型：`content_items` + `body_markdown`，Agent 能在一个上下文中管理
2. 数据来源弱绑定：不主动调用 TrendRadar，用户提供材料
3. 不做发布：Markdown 输出是最终产物，其他 Skill 自行消费
4. 19 个文件，3 个 Phase，3 周可完成
5. 如果未来需要三层模型、provider 契约、发布管道，那是 v2.0 的事——在真实需求驱动下扩展

---

*整理日期：2026-06-17*
