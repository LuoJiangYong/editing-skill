# Editing Skill — 架构基准 v1.0

> 分析源：`ai-tech-news-editor-architecture-v1.11.md`（4677 行）
> 方法框架：Hermes Skill 开发方法论
> 讨论轮次：3 轮（v1.0 初稿 + 2 轮深度校准）
> **状态：✅ 全部开放问题已确认，可进入 Phase 0**

---

## 一、旧架构（v1.11）诊断

### 1.1 核心问题：设计失控

v1.11 规划了 **8 个 Phase、100+ 文件**，覆盖内容生产、平台发布、插件维护三大子系统。根因是 **"先设计宇宙再写一行代码"**——在只有 1 个 data provider 时设计了完整的 provider 契约层，在代码尚未存在时设计了完整的维护元数据体系。

### 1.2 8 个具体问题

| # | 问题 | 严重 | 说明 |
|---|------|------|------|
| 1 | **过度抽象 data provider 层** | 🔴 | TrendRadar 被深度绑定为默认 provider，同时又设计 provider 契约来"解耦"。1 个消费者却有完整契约层。 |
| 2 | **三层内容模型（items→blocks→markdown）** | 🔴 | Agent 在单个上下文无法维护三层结构一致性。两层（items + markdown）够用。 |
| 3 | **平台发布子系统内建** | 🔴 | 信息图（infographic）内嵌在编辑 Skill 中，但这是 `infographic-carousel` Skill 的职责。 |
| 4 | **维护元数据系统过早** | 🟡 | 8 个 metadata yaml + 5 个 meta-schema + 8 个 scan 脚本——文件还没创建就开始设计维护框架。 |
| 5 | **正文结构模板过于精细** | 🟡 | 15+ 通用信号块，每种内容类型 10-20 个特定块。Agent 上下文承载不了。 |
| 6 | **发布映射（publishing_map）冗余** | 🟡 | content_blocks → publishing_map → 平台素材，多了一层映射确认。直接从确认后的内容资产包生成更直接。 |
| 7 | **TrendRadar 耦合过深** | 🔴 | 11 个工具的字段映射表硬编码在正文信号块、评分层、workflow 工具列表中。 |
| 8 | **8 个 Phase** | 🔴 | 方法论建议 L3 级别 5 个 Phase，8 个太多——维护成本随文件数平方增长。 |

### 1.3 方法论对照

| 方法论原则 | v1.11 | 差距 |
|-----------|-------|------|
| 高效简洁 — <3 个消费者不抽象 | ❌ provider 契约只有 1 个消费者 | 严重违反 |
| 高内聚 — One file, one job | ❌ 信号块散落在 5 个结构模板 | 需收敛 |
| 低耦合 — 改一个不动其他 | ❌ provider 字段映射影响评分→结构→发布三层 | 需简化 |
| 易维护 — 可追溯性好过聪明 | ❌ 元数据体系本身变成维护对象 | 需删除 |
| SKILL.md ≤ 3500 chars | ⚠️ 内容远超 | 需压缩 |
| Phase 分阶段隔离 | ⚠️ 8 个 Phase | 需精简为 3-4 |

---

## 二、用户需求校准（第 2 轮讨论）

### 2.1 用户明确的需求边界

> 用户原话："技能主工作流不涉及评分，如果有需要 trendradar 的能力，单独调用或者内聚在对应的选题场景（日报、周/月热点分析等），另外我需要在这个 skill 完成从内容采编-信息获取-内容资产包确认-格式消费（用于微信公众号等平台的 html 长文排版，预设格式，格式可拓展更换）-发布建议。"

| 需求点 | 含义 |
|--------|------|
| **评分不在主工作流** | 评分逻辑下沉到各 workflow，按内容类型决定是否需要评分。需要热点信号时（如日报/周报），对应的 workflow 内聚评分逻辑。 |
| **TrendRadar 按需调用** | 不硬编码在主工作流中。日报/周报/月报需要热点发现时，在对应 workflow 中"单独调用"TrendRadar 工具。 |
| **格式消费是核心能力** | 微信公众号 HTML 长文排版是 Skill 的正式产出（不是交给别人的）。预设格式，格式可拓展更换。 |
| **发布建议是最后一环** | 基于内容类型 + 格式 + 时效给出发布时间/平台/标题变体建议。 |
| **内容资产包是中间产物** | 采编完成后 → 确认 → 生成 content-package → 格式消费的输入。 |

### 2.2 Skill vs Plugin 决策

| 维度 | Skill（推荐） | Plugin |
|------|-------------|--------|
| 核心价值 | 流程编排 + 编辑判断 + 模板引导 | 确定性逻辑 + 自定义工具 |
| 匹配度 | ✅ 工作流、角色、结构模板、HTML 生成——AI 擅长 | ❌ 不需要 Python/Node 支撑 |
| 格式消费 | Agent 按模板生成 HTML——灵活，改模板即生效 | 脚本生成 HTML——迭代需改代码 |
| 维护成本 | 编辑 .md → 立即生效 | 编辑代码 → 测试 → 部署 |
| 扩展性 | 新增格式 = 新增模板文件 + SKILL.md 一行路由 | 新增格式 = 新脚本 + 测试 |

**结论：推荐 Skill**。

---

## 三、内容类型体系（7 种）

> 这是从 v1.11 的 5 种类型 + 专题变体中重新梳理的结果。
> 专题分析的 4 个变体（通用专题、产品推荐、GitHub 推荐、社会洞察）升级为独立的一级类型，
> 因为它们的信息获取策略、结构逻辑、评分需求各不相同——不是一个"变体"标签能覆盖的。

| # | 类型 | 英文标识 | 特征 | 数据获取模式 |
|---|------|---------|------|------------|
| 1 | **日报** | `daily-news` | 当日 AI/科技要闻速览，10 条以内，标准化生产 | 热点驱动（web_search / TrendRadar 可选） |
| 2 | **周/月热点分析** | `hotspot-analysis` | 周期性热点复盘，趋势+排行+代表事件+预测 | 热点驱动 + 周期对比（TrendRadar 按需） |
| 3 | **通用专题分析** | `topic-deep-dive` | 公司/技术/事件/产业议题的深度分析 | 编辑驱动 + web_search 深度检索 |
| 4 | **产品/工具推荐** | `product-recommendation` | AI 工具、科技产品、模型服务、开发者工具推荐 | 编辑驱动 + 官方页面 + 评测来源 |
| 5 | **GitHub 项目推荐** | `github-project` | 开源项目、AI 工具库、开发框架推荐 | 编辑驱动 + GitHub API/页面 + 社区信号 |
| 6 | **AI 与科技社会洞察** | `social-insight` | 技术变化对社会/职业/教育/文化/监管的影响 | 编辑驱动 + 研究/政策/新闻多源交叉 |
| 7 | **推荐内容建议** | `content-advice` | 选题推荐、标题方向、传播角度、后续流转建议 | 轻量信息扫描 + 编辑判断 |

**两种生产模式：**

| 模式 | 类型 | 特征 |
|------|------|------|
| **标准化生产** | 日报、周/月热点 | 数据驱动 → 评分筛选 → 结构模板 → 相对固定流程 |
| **编辑判断驱动** | 专题、产品、GitHub、社会洞察 | 主题先行 → 深度检索 → 编辑组织 → 灵活结构 |

---

## 四、修正后的架构：4 层 Pipeline

```
用户需求
    │
    ▼
┌──────────────────────────────────────────┐
│              SKILL.md 路由中枢             │
│  日报 / 周月热点 / 专题 / 产品 / GitHub /   │
│  社会洞察 / 内容建议                          │
└──────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────┐
│  L1: 内容采编 + 信息获取                   │
│  ┌────────────────────────────────────┐  │
│  │ 用户提供材料 / web_search 补充       │  │
│  │ 日报/周报/月报：可选 TrendRadar 热点   │  │
│  │ → 候选池 → 筛选 → 来源归集           │  │
│  └────────────────────────────────────┘  │
│  ➤ 信息确认关卡                          │
└──────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────┐
│  L2: 内容资产包 (content-package)         │
│  ┌────────────────────────────────────┐  │
│  │ content_items（素材层）              │  │
│  │ body_markdown（Markdown 成稿）       │  │
│  │ references（来源清单）               │  │
│  │ editorial_notes（编辑手记）          │  │
│  └────────────────────────────────────┘  │
│  ➤ 内容确认关卡                          │
└──────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────┐
│  L3: 格式消费                             │
│  ┌────────────────────────────────────┐  │
│  │ 读取 confirmed content-package      │  │
│  │ → 匹配预设格式模板                   │  │
│  │ → 生成平台排版成品（HTML/其他）       │  │
│  │ 预设：微信公众号长文 HTML             │  │
│  │ 可扩展：新增格式 = 新增模板文件        │  │
│  └────────────────────────────────────┘  │
│  ➤ 格式确认关卡                          │
└──────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────┐
│  L4: 发布建议                             │
│  ┌────────────────────────────────────┐  │
│  │ 基于内容类型 + 格式 + 时效            │  │
│  │ → 建议发布时间/平台/标题变体/封面方向   │  │
│  └────────────────────────────────────┘  │
│  ➤ 最终交付：content-package + HTML + 发布建议卡 │
└──────────────────────────────────────────┘
```

**3 层确认关卡**（从 v1.11 的 4 层精简）：

| 关卡 | 位置 | 确认内容 | 可回流至 |
|------|------|---------|---------|
| ① 信息确认 | L1→L2 之间 | 选题范围、候选新闻、来源覆盖 | L1 |
| ② 内容确认 | L2→L3 之间 | 正文结构、事实准确性、引用完整性 | L1 或 L2 |
| ③ 格式确认 | L3→L4 之间 | 排版样式、组件选择、输出规格 | L3 |

---

## 五、目录结构（~35 文件）

```
editing-skill/
├── CONSTITUTION.md                   # 5 条设计原则 + 禁止模式
├── SKILL.md                          # 路由中枢（≤3500 chars 正文）
│
├── references/                       # 领域知识
│   ├── content-types-guide.md        # 7 类内容定义 + 适用场景
│   ├── data-sources.md               # 数据源清单（能力→工具映射）
│   ├── source-and-verification.md    # 来源分类与验证规则
│   ├── frameworks/                   # 分析框架知识库（按需扩展）
│   │   ├── _README.md                # 挂载点说明
│   │   └── ...                       # 麦肯锡/PEST/Porter/技术成熟度等
│   └── confirmation/                 # 确认协议
│       ├── information-confirmation.md
│       ├── editorial-confirmation.md
│       └── format-confirmation.md
│
├── schemas/
│   └── content-package.md            # 内容资产包结构（两层 + assets/）
│
├── templates/                        # 模板层
│   ├── content-structure/            # 正文结构指南（7 类）
│   │   ├── daily-news.md
│   │   ├── hotspot-analysis.md       # 周/月热点
│   │   ├── topic-deep-dive.md        # 通用专题
│   │   ├── product-recommendation.md
│   │   ├── github-project.md
│   │   ├── social-insight.md
│   │   └── content-advice.md
│   ├── formats/                      # 格式消费模板（每个格式=独立目录）
│   │   ├── tech-magazine/            # 第一阶段：用户提供文章链接+design.md 固化
│   │   │   ├── design.md             # 格式基线（配色/字号/组件/思想）
│   │   │   ├── REFERENCE.md          # 参考来源
│   │   │   └── example.html          # 完整样例供审阅选择
│   │   └── _TEMPLATE/                # 新增格式模板（复制此目录起手）
│   ├── panels/                       # 编辑面板模板（Phase 2）
│   │   └── review-panel.html         # 通用面板，格式无关
│   └── publishing-advice.md          # 发布建议卡模板
│
├── workflows/                        # 7 个工作流 + 格式消费入口
│   ├── daily-news.md
│   ├── hotspot-analysis.md           # 周/月热点
│   ├── topic-deep-dive.md
│   ├── product-recommendation.md
│   ├── github-project.md
│   ├── social-insight.md
│   ├── content-advice.md
│   └── format-consume.md             # 格式消费总入口（L3）
│
└── examples/
    ├── github-project-example/       # 第一阶段样例
    │   ├── manifest.json
    │   └── body.md
    └── hotspot-analysis-example/     # 第一阶段样例
        ├── manifest.json
        └── body.md
```

---

## 五、关键设计决策

### 5.1 内容资产包：两层模型

```
content-package:
├── metadata            # 标题、日期、类型（daily_news|weekly|monthly|topic|advice）
├── content_items[]     # 素材层
│   ├── id, type（news|event|trend|product|paper|github...15 种）
│   ├── title, summary, url
│   ├── source_platform { category, owner, channel, medium, display }
│   ├── status: candidate | selected | background | rejected
│   └── editorial_note（为什么入选/剔除）
├── body_markdown       # 完整 Markdown 正文（含引用标注）
├── references[]        # 来源清单（id, title, url, type, reliability）
└── editorial_notes     # 编辑手记（全局取舍理由、风险提示、数据局限说明）
```

**不做 `content_blocks` 的原因：**
- Agent 从 items 直接写 markdown 更自然，质量可控
- 格式消费层用 markdown 标题/段落结构解析即可，不需要额外 block type 系统
- 减少 1/3 的结构维护成本

### 6.2 评分：不放在主工作流，各 workflow 内聚

```
# workflows/hotspot-analysis.md（热点分析需要快速筛选 → 有评分）
## 评分（热点分析专用）
- 热度: 提及次数/平台覆盖/趋势方向 → 1-5
- 来源权威: 官方/媒体/社区 → 1-5
- 编辑判断: 行业重要性 → 1-5
- 入选线: 总分 ≥ 10 或编辑强制入选

# workflows/github-project.md（GitHub 推荐不需要评分 → 无评分）
## 项目筛选标准
- 不评分，编辑全量判断
- 按 stars / 最近提交 / 文档质量 / 社区活跃 → 筛选
- 按适用场景/技术栈分组
```

### 6.3 工具调用解耦：能力声明 + 数据源清单

**核心思路：workflow 声明"我需要什么数据"，不声明"用哪个工具拿"。**
Agent 运行时查 `references/data-sources.md` 决定用什么工具。

#### workflow 中的能力声明（以热点分析为例）

```markdown
# workflows/hotspot-analysis.md

## 需要的数据能力

### 日期解析
- 需求: 把"本周""最近7天"转为标准日期范围
- 来源: data-sources.md → date_resolution

### 热点发现
- 需求: AI/科技领域本周热度最高的新闻标题、排名、平台覆盖
- 来源: data-sources.md → trending_discovery
- 门槛: 至少覆盖 3 个平台 / 20 条候选

### 趋势信号
- 需求: 核心话题的热度变化曲线
- 来源: data-sources.md → trend_signal
- 可选: 数据不足时不输出该信号块

### 周期对比
- 需求: 本周 vs 上周话题消长
- 来源: data-sources.md → period_comparison
- 可选
```

#### data-sources.md 结构

```markdown
# 数据源清单

## 能力 → 可用工具

| 能力 | 工具（按优先级） | 说明 |
|------|-----------------|------|
| `date_resolution` | ① LLM 直接计算 ② TrendRadar `resolve_date_range` | LLM 够用 95% |
| `trending_discovery` | ① web_search ② browser HN/ProductHunt ③ TrendRadar `analyze_sentiment` | TrendRadar 是加速器 |
| `trend_signal` | ① LLM 从热点结果归纳 ② TrendRadar `analyze_topic_trend` | 趋势判断本质是编辑判断 |
| `period_comparison` | ① 两次 web_search 结果对比 ② TrendRadar `compare_periods` | |
| `github_discovery` | ① gh CLI ② browser github.com ③ web_search | 不需要 TrendRadar |
| `web_verification` | ① browser ② web_search 交叉验证 | 通用能力 |

## 运行时决策链

Agent 读 data-sources.md → 按优先级尝试工具：
  ① 通用工具可用 → 用通用工具
  ② 不可用 → 降级下一优先级
  ③ 全部不可用 → 写 data_limitations，能力标记 unavailable
  用户说"用 TrendRadar" → 跳过降级链，直达专属工具

## 新增规则

- 新增工具：对应能力下加一行
- 新增能力：表中加一行 + workflow 中加一条声明
- 不新增任何中间层（adapter / field_mapping / provider 契约）
```

#### 高内聚保证

```
每个 workflow 文件包含：
  ├── 我需要什么数据（能力声明）
  ├── 怎么筛选/评分（该类型专属逻辑）
  ├── 怎么组织正文（引用结构模板）
  └── 输出什么（content-package）

改评分 → 只改 workflow
换工具 → 只改 data-sources.md
新增类型 → 新 workflow + 结构模板，不动其他文件
```

### 6.4 来源验证：真实、可访问、权威优先

`references/source-and-verification.md` 定义三项硬检查：

```markdown
# 来源分类与验证规则

## 验证目标

每条引用必须通过三项检查：

### 1. 来源真实
- 有明确的发布主体（公司/机构/个人）
- 有明确的发布时间
- 不是二手转述链（"据外媒报道""网友称"）
- 无法确认主体或时间 → 标记"来源待核实"

### 2. 链接可访问
- Agent 用 browser 打开 URL → 确认页面存在且内容匹配
- 链接失效/404/需登录 → 标记"链接不可访问"
- 无链接 → 标记"无原始链接"

### 3. 优先权威源
- 同一事件有多个来源时，选官方/一手：
  官方公告 > 主流科技媒体 > 社区讨论
- 有官方来源时不引用二手报道
- 仅有社区来源 → 标注"未经独立验证"

## 标记方式

每条 content_item 标注:
  source_verified: true | false
  link_accessible: true | false | no_link
  source_tier: official | media | community | unknown
```

### 6.5 分析框架知识库

`references/frameworks/` 是分析框架知识库。当前为空目录（含 `_README.md`），未来按需添加。

```
references/frameworks/
├── _README.md                  # "放什么、怎么用、怎么引用"
├── mckinsey-methods.md         # MECE / 金字塔原理 / 假设驱动（待添加）
├── industry-analysis.md        # Porter 五力 / PEST / 技术成熟度曲线
└── tech-strategy.md            # 创新者窘境 / 跨越鸿沟 / 平台战略
```

**加载方式：** workflow 中按需声明，沿用已有的"加载 references/xxx.md"模式：

```markdown
# workflows/topic-deep-dive.md

## 知识库加载
- 默认: references/frameworks/industry-analysis.md
- 公司分析: 额外加载 references/frameworks/tech-strategy.md
- 不强制: 框架是辅助工具，不替代编辑判断
```

**为什么不需要架构级设计：**
- 不需要新加载协议——workflow 已有"加载 references/"模式
- 不需要新文件格式——就是 Markdown
- 不影响 content-package schema
- 不需要专属 Phase——随用随加
- 新框架 = 新增一个 `.md` 文件，零摩擦

### 6.6 格式消费：与内容资产包完全解耦

**核心原则：** content-package 不存任何格式信息（颜色、字号、组件），格式不存任何内容信息（标题、来源、评分）。两者仅在 `format-consume.md` 运行时交汇。

#### 开发阶段 vs 应用阶段

```
开发阶段（迭代固化）：
  ① 你提供: 公众号文章链接 + design.md
  ② Agent: 分析文章 + 解析 design.md → 生成格式草案
  ③ Agent: 用草案渲染一个样例 HTML → 你审阅
  ④ 你: 提出调整（"标题字号太小""卡片间距太紧""配色偏冷"）
  ⑤ Agent: 根据反馈修改 design.md → 重新渲染样例 → 你再审阅
  ⑥ 多轮迭代直到满意 → 固化为命名格式目录
  → 成果: templates/formats/tech-magazine/

应用阶段：
  用户: "用科技杂志风生成 HTML"
  Agent: 读取 content-package + tech-magazine/ → 合并渲染 → HTML
```

#### 目录结构

```
templates/formats/
├── tech-magazine/                    # 格式 = 一个独立目录
│   ├── design.md                     # 设计规范（配色/字号/组件/思想）
│   ├── REFERENCE.md                  # 参考来源（文章链接 + 提取的视觉特征）
│   ├── overrides.md                  # 可选：特殊组件的格式级覆盖
│   └── example.html                  # 完整样例（用户审阅和选择用）
│
├── minimal-light/                    # 未来扩展
│   ├── design.md
│   ├── REFERENCE.md
│   └── example.html
│
└── _TEMPLATE/                        # 新建格式模板
    ├── design.md
    ├── REFERENCE.md
    └── example.html
```

#### 解耦关系

```
content-package（格式无关）           格式（内容无关）
┌────────────────────────┐           ┌────────────────────────┐
│ manifest.json          │           │ tech-magazine/          │
│   content_items[]     │           │   design.md            │
│   references[]        │           │   (颜色/字号/间距)      │
│                        │           │   REFERENCE.md         │
│ body.md               │           │   overrides.md         │
│   # 标题               │           │                        │
│   ## 章节              │           │                        │
│   ![](assets/x.png)   │           │                        │
│                        │           │                        │
│ assets/               │           │                        │
└────────────────────────┘           └────────────────────────┘
         │                                      │
         └──────────────┬───────────────────────┘
                        ▼
              format-consume.md
              运行时读取两边，合并渲染 → HTML
```

**扩展格式不改上下游：**
- 新增格式 = 复制 `_TEMPLATE/` + 填入 design.md → 完成
- 不影响 content-package schema、不影响 workflow、不影响已有格式
- 换格式只需在请求时指定不同格式名

### 6.7 本地可视化编辑面板（Agent 交互层）

HTML 分为两种形态：

| | 本地预览版（含面板） | 最终交付版（干净） |
|---|---|---|
| **用途** | 用户审阅 + 编辑 + 与 Agent 交互 | 复制到微信后台 |
| **JS** | ✅ 内联（编辑/批注/提交逻辑） | ❌ 无 JS |
| **面板** | ✅ 工具栏 + 编辑区 + 批注层 | ❌ 无 |
| **文件名** | `output-review.html` | `output.html` |

#### 面板布局

```
output-review.html（本地浏览器打开）
┌─────────────────────────────────────────────────┐
│ [编辑内容] [调整排版] [添加批注] [提交修改] [导出]  │ ← 工具栏
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─ 📝 可编辑 ──────────────────────────────┐  │
│  │ # 今日 AI 与科技要闻                      │  │
│  │                                           │  │
│  │ OpenAI 发布三款实时音频模型...  [💬 批注]  │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
│  ┌─ 💬 批注列表 ────────────────────────────┐  │
│  │ Agent: 这条建议加数据对比             [✓] │  │
│  │ 你: 数据来源？补充后再加             [✏️] │  │
│  └───────────────────────────────────────────┘  │
│                                                 │
└─────────────────────────────────────────────────┘
```

#### 工作流

```
① Agent 生成 output-review.html（含面板）
   → 用户本地浏览器打开

② 用户审阅 + 修改:
   - 内容编辑: 点击段落 → contenteditable → 直接改文字
   - 排版调整: 点击组件 → 调整面板改字号/间距/颜色
   - 批注: 选中区域 → 写批注 → 对话式交互

③ 用户点击 [提交修改]
   → 面板提取所有改动 → 生成结构化的 change-log

④ 用户将 change-log 发送给 Agent:
   改动内容:
   - 标题 "今日AI要闻" → "今日 AI 与科技速览"
   - 卡片间距 16px → 24px
   批注:
   - "第二条新闻建议加一个竞品对比表"

⑤ Agent 解析 change-log:
   内容修改 → 回写 body.md / manifest.json
   排版修改 → 回写 format/design.md
   批注 → 执行或回复

⑥ Agent 重新渲染 output-review.html → 用户继续审阅
   满意后 → 点击 [导出] → 生成干净的 output.html（无面板/无 JS）
   → 用户复制到微信后台
```

#### 为什么不做自动双向同步

```
不做: HTML 自动写文件 → 文件系统权限问题、浏览器安全限制、跨平台兼容性
做:   面板提取 change-log → 用户复制 → Agent 落盘
      → 这个流程可靠、无依赖、所有平台一致
```

#### 实现策略

**Phase 1（手动回流）：** 不用面板。用户看 HTML → 描述改动 → Agent 改源文件 → 重渲染。先跑通"真源回流"这个核心模式。

**Phase 2（加面板）：** `templates/panels/review-panel.html` 是通用面板模板，格式无关。Agent 生成 output-review.html 时注入面板。换格式不影响面板，换面板不影响格式。

### 6.8 发布建议

发布建议是轻量附卡，不是独立工作流。L3 完成后自动生成：

```
发布建议卡：
├── 建议发布平台: 微信公众号
├── 建议发布时间: 今日 20:00-21:00（AI/科技读者活跃时段）
├── 标题变体:
│   ├── 正式版: "AI科技理想图·简报POST | 2026年6月17日"
│   ├── 社交版: "今天AI圈发生了什么？3分钟速览"
│   └── 技术版: "0617 AI要闻：OpenAI新模型 + Meta开源动态"
├── 封面方向: [摘要卡片式 / 关键词视觉化 / 代表性截图拼贴]
└── 注意事项: [是否需要声明数据源受限 / 是否包含未证实消息]
```

### 6.9 三级输出体系

```
content-package（唯一内容资产母版）
    │
    ├── L3: 格式消费（本 Skill 负责）
    │   ├── 文本型平台: 微信公众号/知乎/Newsletter → HTML → 复制粘贴
    │   └── 图片型平台: 小红书 → Brief/提示词 → 对接 infographic-carousel
    │
    ├── L3b: 内容转化（不实现，预留接口）
    │   └── 下游 Skill 消费 content-package → 音频/视频
    │
    └── L4: 发布建议（本 Skill 负责）
```

---

## 六、Phase 实施计划

### Phase 0：宪法 + Schema + SKILL.md 路由骨架

**文件：** CONSTITUTION.md + content-package schema + SKILL.md

**核心验证：** SKILL.md 能正确路由 7 类需求 + 格式消费入口

### Phase 1（第一阶段实现）：GitHub 项目推荐 + 周/月热点分析 + 格式消费

选择这两个类型是因为它们恰好覆盖两种生产模式：

| 类型 | 模式 | 验证目标 |
|------|------|---------|
| **GitHub 项目推荐** | 编辑判断驱动 | 主题检索 → 项目筛选 → 结构组织 → 确认 → HTML |
| **周/月热点分析** | 标准化生产 | 热点获取 → 评分排序 → 趋势分析 → 确认 → HTML |

**Phase 1 文件清单（~14 个）：**

```
CONSTITUTION.md                          # 从 Phase 0 继承
SKILL.md                                 # 从 Phase 0 继承（路由 7 类）
schemas/content-package.md               # 从 Phase 0 继承

references/
├── content-types-guide.md               # 7 类定义（先写 hotspot + github 两节，其余占位）
├── data-sources.md                      # 通用
├── source-and-verification.md           # 通用
└── confirmation/
    ├── information-confirmation.md      # 通用
    ├── editorial-confirmation.md        # 通用
    └── format-confirmation.md           # 通用

templates/
├── content-structure/
│   ├── hotspot-analysis.md              # ✅ 第一阶段
│   └── github-project.md                # ✅ 第一阶段
├── formats/
│   └── wechat-html.md                   # ✅ 第一阶段（含用户参考样式输入机制）
└── publishing-advice.md                 # 通用

workflows/
├── hotspot-analysis.md                  # ✅ 第一阶段
├── github-project.md                    # ✅ 第一阶段
└── format-consume.md                    # 通用格式消费入口

examples/
├── github-project-example/
│   ├── manifest.json
│   └── body.md
└── hotspot-analysis-example/
    ├── manifest.json
    └── body.md
```

### Phase 2：日报 + 专题 + 产品推荐

### Phase 3：社会洞察 + 内容建议 + 格式模板扩展

---

## 七、v1.11 中保留的设计

| 设计 | v1.11 | 本架构 |
|------|-------|--------|
| 确认关卡 + 回流 | 4 层 | 精简到 3 层（信息→内容→格式），保留影响范围判定 |
| 内容类型路由 | 5 种 + 专题变体 | 7 种独立一级类型，路由表放 SKILL.md |
| 事实不可杜撰 | 全文强调 | 保留为核心禁令 |
| 中文输出优先 | 全文强调 | 保留 |
| 来源平台分类 | `source_platform` 结构 | 保留在 content_items 中 |
| 素材类型枚举 | 15 种 item type | 保留，Agent 按需选用 |

---

## 八、全部决策汇总

| # | 议题 | 结论 |
|---|------|------|
| 1 | content-package 格式 | 目录型（manifest.json + body.md + assets/） |
| 2 | HTML 组件粒度 | 完整独立 HTML，内联 style + SVG |
| 3 | 第一阶段类型 | GitHub 项目推荐 + 周/月热点分析 |
| 4 | 编辑角色系统 | 去掉——全局标准放 CONSTITUTION，各 workflow 内聚 |
| 5 | 持久化策略 | 固定 outputs/ 目录长期保存 |
| 6 | 格式消费 | 格式=独立目录，与 content-package 完全解耦 |
| 7 | 样式输入 | 公众号文章链接 + design.md → Agent 消化 → 样例审阅 → 反馈迭代 → 固化 |
| 8 | 工具调用解耦 | 能力声明 + data-sources.md 降级链 |
| 9 | 知识库 | references/frameworks/ 预留挂载点 |
| 10 | 三级输出 | 格式消费优先；音视频转化预留接口，不实现 |

---

*固化日期：2026-06-17 | 讨论轮次：2 | 下一动作：用户确认开放问题 → Phase 0*
