# 日报 — 工作流

> 路径：workflows/daily-news.md
> 职责：定义日报（daily-news）的完整生产流程（L1→L2），从热点获取到 content-package 生成
> Phase：2
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md（第四节）, templates/content-structure/daily-news.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：标准化生产（时效驱动 → 编辑筛选 → 结构固定）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md`（第四节） |
| L2 | `templates/content-structure/daily-news.md` |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `date_resolution` | 解析"今天""昨天"等用户表达为标准日期 | — |
| `trending_discovery` | 当日 AI/科技领域最新新闻标题、来源、发布时间 | ≥ 15 条候选 |
| `web_verification` | 来源验证：打开链接确认页面存在且发布时间在 24h 内 | 对所有 url 非空的 candidate item 执行 |

> **明确不需要的能力：** `trend_signal`（趋势分析）、`period_comparison`（周期对比）、评分（任何形式）。日报的筛选器是时效性本身，不是热度评分。

---

## L1 工作流：内容采编

### 步骤 1：解析时间范围

```
输入：用户需求中的时间表达（"今天""昨日"等）
输出：标准日期范围 {start: "YYYY-MM-DD", end: "YYYY-MM-DD"}，窗口 ≤ 24h

能力：date_resolution
```

日报的时间窗口默认为当前日期。如果用户在非标准时间请求（如凌晨 2:00 要"昨天"的日报），按用户指定调整。

### 步骤 2：获取热点候选

```
能力：trending_discovery

操作：
1. 按 data-sources.md → trending_discovery 降级链获取当日新闻候选
2. 候选池门槛：≥ 15 条、≥ 2 个平台
3. 不满足门槛 → 扩大检索（换平台/放宽关键词/扩展时间窗口 ±12h）
4. 候选不足时 → 降级到下一优先级工具补充

输出：候选新闻列表（标题 + 摘要 + 平台 + 发布日期 + URL 如有）
```

### 步骤 3：时效筛选

**日报的核心筛选逻辑。** 不评分，只检查时效和编辑判断。

#### 时效门槛（硬性）

```
item.published_at 必须在 [当前时间 - 24h, 当前时间] 范围内
```

- published_at 为空（无法确认时间）→ 降级为 background，不纳入要闻列表
- published_at 超过 24h → rejected（editorial_note 说明"超出日报时效窗口"）
- 跨日事件（如持续 3 天的发布会 Day 2）→ selected（editorial_note 注明"Day N of M"）

#### 编辑判断筛选（在时效门槛内）

对通过时效门槛的候选，按以下维度精选 5-10 条：

| 维度 | 判断标准 |
|------|---------|
| 行业重要性 | 是否影响行业格局？是否涉及头部公司/技术？ |
| 用户兴趣 | 是否与目标读者（AI/科技从业者）相关？ |
| 信息密度 | 是否有实质内容（非标题党/非纯营销）？ |
| 多样性 | 是否覆盖了不同主题方向（避免全部是同一家公司）？ |

**不关注的维度：**
- ❌ 热度绝对数值（不评分）
- ❌ Stars/点赞/转发量
- ❌ 是否符合本周叙事线（日报不做叙事线）

#### 输出

```
每个候选 item：
├── content_item id (item-001, item-002, ...)
├── type 归类（news/event/product/paper 等 15 种）
├── status: selected / rejected / background
├── published_at 已确认
└── editorial_note：入选理由 / 剔除原因（简洁，不展开）
```

### 步骤 4：来源验证

```
能力：web_verification

操作：
1. 对每个 url 非空的 selected item，browser 打开 URL
2. 确认页面存在且内容与摘要匹配
3. 确认 published_at 日期准确
4. 通过 3 项硬检查（见 references/source-and-verification.md）
5. 填写 source_verified / link_accessible / source_tier

日报特有：
- 时效验证是日报来源验证的额外步骤：确认 published_at 在 24h 内
- 无链接的 item（如仅在其他平台看到标题）→ 标记 link_accessible: "no_link"，尝试 web_search 找原始来源
- 无法在 3 分钟内完成验证的 item → 标注"来源待验证"，不阻塞日报生成
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 日期范围 — 覆盖的 24h 窗口
2. 候选统计 — 候选池 XX 条 / 时效筛选后 XX 条 / 入选 XX 条（≤10）/ 剔除 XX 条
3. 时效覆盖 — selected items 的 published_at 分布（确认均在 24h 内）
4. 来源覆盖 — 平台清单 + 验证状态

回流：用户要求调整 → 回到步骤 2/3/4
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 5：加载结构模板

```
加载：templates/content-structure/daily-news.md
确定：标题格式、章节顺序、条目模板
```

### 步骤 6：撰写 body.md

```
输入：selected items（≤10 条）+ 结构模板
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

章节：
# <标题>
## 一、今日速览
  日期标注 + 2-3 句核心事件概括
## 二、要闻列表
  每条：### N. <标题> → 2-3 句话概述 → 📎 来源引用
## 三、明日关注（如有内容）
## 参考来源（自动生成）

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分信息
- ❌ 不含趋势分析/周期对比
- ❌ 不含信号块（💡📈👀⚠️）
- ❌ 每条不超过 4 句话（2-3 句事实 + 1 句来源）
- ✅ 纯 Markdown + [^ref-xxx] 脚注
- ✅ 每条 must 包含 published_at 确认
- ✅ 明日关注只写有信息来源的展望
```

### 步骤 7：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "daily-news-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, date, created_at,
    content_type: "daily-news",
    keywords[], language: "zh-CN",
    time_sensitivity: "daily" | "real-time",
    data_sources_used[], data_limitations?
  },
  content_items: [所有 item, status 含 selected/rejected/background],
  body_markdown: "<body.md 全文>",
  references: [所有被引用来源],
  editorial_notes: {
    selection_rationale, rejection_summary?, fact_check_notes,
    data_limitations, risk_notes?, editorial_decisions?
  }
}
```

### 步骤 8：写入 outputs/

```
创建目录：outputs/daily-news-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 章节 + 条数 + 字数
2. 时效核查 — 所有 selected items 的 published_at 在 24h 内
3. 事实核查 — 已核查数 / 未验证数
4. 引用完整性 — body.md 脚注 ↔ references[] 匹配率

回流：用户要求修改 → 回到步骤 6/7/8
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## 快速模式

日报支持快速模式（日报是每日固定任务，流程高度重复）：

1. 用户明确说"快速模式""跳过确认""直接出日报"
2. 或：连续第 3 次以上执行日报，来源偏好和流程已稳定

快速模式下：
- 跳过确认 ① 和 ② 的呈现
- 仍完成所有步骤（包括来源验证和时效检查）
- 生成后直接进入 L3 格式消费

---

## 日报与热点分析的差异速查

| 维度 | 日报 | 热点分析 |
|------|------|---------|
| 时间窗口 | 24h | 7-30 天 |
| 评分 | 无 | 有（热度+权威+编辑判断） |
| 趋势分析 | 无 | 有 |
| 周期对比 | 无 | 可选 |
| 信号块 | 无 | 有 |
| 筛选逻辑 | 时效门槛 → 编辑判断 | 评分 → ≥10 入选 |
| 条目深度 | 简讯（2-3 句） | 展开（100-200 字） |
| 叙事线 | 无 | 有 |
| 展望 | 明日关注（选项） | 下周/下月展望（常规） |
| 资源加载 | 不加载 trend_signal/period_comparison | 加载全部 5 项能力 |
