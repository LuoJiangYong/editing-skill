# 产品/工具推荐 — 工作流

> 路径：workflows/product-recommendation.md
> 职责：定义产品/工具推荐（product-recommendation）的完整生产流程（L1→L2），从产品发现到 content-package 生成
> Phase：3
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md（第六节）, templates/content-structure/product-recommendation.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：编辑判断驱动（选题先行 → 产品发现 → 评测收集 → 筛选分组）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md`（第六节） |
| L2 | `templates/content-structure/product-recommendation.md` |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `web_search` | 产品发现（搜索品类关键词）+ 评测检索（"产品名 + 评测/使用体验/对比"） | ≥ 8 个候选产品 |
| `web_verification` | 验证产品官网可访问、价格信息准确、评测来源真实 | 对所有 candidate item 执行 |

> **明确不需要的能力：** `trending_discovery`（不走热点流）、`trend_signal`（不做趋势分析）、评分（任何形式）。产品推荐依赖编辑判断——功能/价格/体验/竞品，不依赖热度信号。

---

## L1 工作流：产品发现与筛选

### 步骤 1：明确推荐方向

```
输入：用户需求中的品类/场景表达
输出：明确的推荐方向

示例输入 → 输出：
  "推荐 AI 视频生成工具" → 品类：AI 视频生成 | 方向：综合推荐 | 范围：SaaS 产品（非开源库）
  "本地推理工具哪个好" → 品类：本地 LLM 推理 | 方向：横向对比 | 范围：可本地运行的产品/工具
  "有什么好用的 AI 编程助手" → 品类：AI 编程助手 | 方向：综合推荐 | 范围：IDE 插件 + CLI 工具

如果用户未指定品类，Agent 主动确认：
  "你想了解哪类产品？AI 视频生成 / 本地推理 / 编程助手 / 其他？"

关键判断：确认推荐对象是"产品/服务"还是"开源代码项目"。
→ 如果是开源代码项目（GitHub 上的库/框架），走 workflows/github-project.md
→ 如果是可注册使用、有定价页面的产品/服务/SaaS/模型 API，走本 workflow
```

### 步骤 2：产品发现

```
能力：web_search

操作：
1. 搜索品类关键词（如"2026 AI 视频生成工具 推荐"）
2. 搜索评测/横评文章（如"AI 视频生成工具 横评 对比"）
3. 搜索社区推荐（如 Reddit/Hacker News/知乎 "best AI video tools 2026"）
4. 候选池门槛：≥ 8 个候选产品
5. 不满足门槛 → 扩展搜索（放宽关键词 / 扩大时间范围）

输出：候选产品列表（名称 + 一句话描述 + 官网 URL + 价格信息 + 适用人群）
```

### 步骤 3：评测收集

```
能力：web_search

本步骤是产品推荐与 GitHub 项目推荐的关键差异——产品推荐需要收集真实用户反馈。

操作：
1. 对每个候选产品，搜索"产品名 + 评测/使用体验/review"
2. 优先收集：
   - 权威科技媒体的横评文章
   - 真实用户的长期使用反馈（Reddit/知乎/社区）
   - 官方更新日志（了解产品迭代速度）
3. 记录产品已知问题/局限
4. 确认价格信息（从官网定价页获取，标注获取日期）

输出：每个产品的评测摘要 + 已知局限
```

### 步骤 4：产品筛选

```
对每个候选产品进行全量编辑判断（不评分）：
```

#### 筛选维度

| 维度 | 关注 | 判断标准 |
|------|------|---------|
| 功能完整性 | 是否覆盖该品类的核心需求？ | 有明确品类覆盖 → 通过。功能有明显缺口 → 降级或剔除 |
| 价格合理性 | 定价是否与功能匹配？ | 有免费层/试用 → 优先。定价远高于同类且无差异化 → 剔除 |
| 用户体验 | 上手难度、文档质量、社区支持 | 评测中反复出现的体验问题 → 标注"编辑点评" |
| 竞品差异化 | 与同类产品相比有什么独特优势？ | 有明确差异化 → 优先。"me-too"产品 → 降级 |
| 适用人群匹配 | 是否有明确的适用人群画像？ | 有清晰用户画像 → 通过。号称"适合所有人" → 谨慎 |

#### 不关注

- ❌ 是否"大厂出品"（独立团队产品同等对待）
- ❌ 是否"最火/最热门"（关注实际质量）
- ❌ 媒体曝光量（可能来自 PR 预算而非产品质量）

#### 输出

```
每个候选产品：
├── content_item id (item-001, item-002, ...)
├── type: "product"
├── status: selected / background / rejected
├── source_platform: { name: "产品官网", category: "official", owner: "<公司名>", display: "<公司名> 官网" }
├── editorial_note：入选理由 / 剔除原因
└── 验证字段待填写（步骤 6）
```

### 步骤 5：分组归类

```
将 selected 产品按用户画像或使用场景分组。

分组原则：
- 每组 2-5 个产品
- 同组内产品功能相近（方便读者对比）
- 分组名体现"谁适合用"或"什么场景用"

示例：
  品类"AI 视频生成" →
    分组 1：专业创作者（Runway, Pika Pro, Synthesia）
    分组 2：快速尝鲜（Pika Free, MoonValley, Pixverse）
    分组 3：企业/API 对接（HeyGen, Tavus）
```

### 步骤 6：来源验证

```
能力：web_verification

操作：
1. 对每个 candidate item，browser 打开产品官网
2. 验证：产品存在、定价页可访问、核心功能描述准确
3. 通过 3 项硬检查（见 references/source-and-verification.md）
   - 官网作为官方信息源：source_tier = "official"
   - 评测文章作为媒体来源：source_tier = "media"
   - 社区反馈作为补充来源：source_tier = "community"
4. 填写 source_verified / link_accessible / source_tier

产品推荐特有：
- 价格信息标注获取日期（"截至 2026-06，官网定价"）
- 如果价格页面需要登录才能查看 → 标注 access_status: "paywalled"
- 如果产品已停止服务（404/维护公告）→ 剔除，editorial_note 说明
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 推荐方向 — 品类 + 范围
2. 候选统计 — 候选 XX 个 / 入选 XX 个（分 X 组）
3. 评测覆盖 — 每个产品至少 1 条评测/用户反馈来源
4. 价格准确性 — 所有价格信息已标注获取日期

回流：用户要求调整 → 回到步骤 1/2/3/4/5
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 7：加载结构模板

```
加载：templates/content-structure/product-recommendation.md
确定：标题格式、分组命名、产品卡片字段
```

### 步骤 8：撰写 body.md

```
输入：selected items（已分组）+ 结构模板
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

章节：
# <标题>
## 一、推荐概览（选题理由 + 筛选说明 + 分组预览）
## 二、产品分组
  每组：
  ### <分组名>
  #### <产品名>
  产品卡片内容（见结构模板）：一句话定位 + 核心功能 + 属性表（价格/适用人群/竞品） + 编辑点评
## 三、对比总览（推荐：同组 ≥2 个产品时）
## 四、选择建议
## 参考来源

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分
- ❌ 不含格式指令
- ✅ 纯 Markdown + [^ref-xxx] 脚注
- ✅ 所有价格为真实数据，标注获取日期
- ✅ 竞品对比基于实际差异，不编造
```

### 步骤 9：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "product-recommendation-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, subtitle?, date, created_at,
    content_type: "product-recommendation",
    keywords[], language: "zh-CN",
    time_sensitivity: "evergreen",
    data_sources_used[], data_limitations?
  },
  content_items: [所有 item, status 含 selected/rejected],
  body_markdown: "<body.md 全文>",
  references: [所有被引用来源, 每个产品至少 1 条官网引用 + 1 条评测引用],
  editorial_notes: {
    selection_rationale, fact_check_notes,
    data_limitations, risk_notes?, editorial_decisions?
  }
}
```

### 步骤 10：写入 outputs/

```
创建目录：outputs/product-recommendation-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 分组数 + 每组产品数 + 是否有对比表
2. 事实核查 — 价格准确性 + 产品功能描述准确性
3. 评测覆盖 — 每个产品是否有真实评测/反馈支撑
4. 引用完整性 — 每个产品至少 1 个 reference

回流：用户要求修改 → 回到步骤 8/9/10
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## 产品推荐的差异特征

与 GitHub 项目推荐（同为编辑判断驱动+分组+卡片+对比+建议结构）的不同之处：

| 维度 | 产品推荐 | GitHub 项目 |
|------|---------|------------|
| 推荐对象 | 产品/服务/SaaS/模型 API | 开源代码项目 |
| 信息获取 | 官网 + 评测 + 用户反馈 + 竞品对比 | 以 GitHub + 社区讨论为主 |
| 特有步骤 | 评测收集（搜"产品名+评测"）+ 价格验证 | 社区信号收集（可选） |
| 筛选维度 | 功能/价格/体验/竞品/适用人群 | 活跃度/文档/实用性/社区/许可证 |
| 卡片字段 | 定位/功能/价格/适用人群/竞品/点评 | 描述/stars/语言/许可证/特性/场景/点评 |
| 时效性 | evergreen（价格/功能会变化） | evergreen（代码持续演进） |
| 评分 | 无 | 无 |

与热点分析的不同之处：

| 维度 | 产品推荐 | 热点分析 |
|------|---------|---------|
| 驱动 | 编辑判断（选题先行） | 数据驱动（热点信号触发） |
| 评分 | 无 | 有（热度+权威+编辑） |
| 趋势分析 | 无 | 有 |
| 周期对比 | 无 | 可选 |
| 分组 | 按用户画像/场景（必须） | 按主题（可选） |
| 时效性 | evergreen | weekly/monthly |
