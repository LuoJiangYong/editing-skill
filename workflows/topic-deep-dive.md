# 通用专题分析 — 工作流

> 路径：workflows/topic-deep-dive.md
> 职责：定义通用专题分析（topic-deep-dive）的完整生产流程（L1→L2），从选题明确到 content-package 生成
> Phase：2
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md（第五节）, templates/content-structure/topic-deep-dive.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：编辑判断驱动（选题先行 → 多源深度检索 → 灵活组织）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md`（第五节） |
| L2 | `templates/content-structure/topic-deep-dive.md` |
| 可选 | `references/frameworks/` 下适用的分析框架（按需加载） |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `web_search` | 多源深度检索：官方公告、科技媒体、学术论文、社区讨论 | 核心事实至少交叉验证 2 个独立来源 |
| `web_verification` | 来源验证：打开链接确认页面存在、内容匹配、发布时间准确 | 对所有 url 非空的 candidate item 执行 |

> **明确不需要的能力：** `trending_discovery`（不走热点流）、`trend_signal`（不做趋势信号分析）、评分（任何形式）。通用专题依赖多源深度检索和编辑判断，不依赖热度信号。

---

## L1 工作流：选题明确与信息采集

### 步骤 1：明确选题边界

```
输入：用户需求中的主题/方向表达
输出：明确的选题边界（主题类型 + 分析角度 + 范围限定）

如果用户表述模糊，Agent 主动确认：
  "你想从哪个角度切入？公司分析 / 技术分析 / 事件复盘 / 产业分析？"

示例输入 → 输出：
  "分析一下 OpenAI 的战略" → 主题类型：公司分析 | 角度：商业策略+竞争 | 范围：2024-2026
  "Transformer 架构还能走多远" → 主题类型：技术分析 | 角度：技术演进+局限 | 范围：当前至未来2-3年
  "解读欧盟 AI 法案" → 主题类型：事件/政策分析 | 角度：条款解读+影响 | 范围：法案全文+各方反应

输出格式：
{
  topic_type: "company" | "technology" | "event" | "industry",
  main_angle: "<核心分析角度>",
  scope: {
    time_range: {start, end} 或 "evergreen",
    geographic_focus: "global" | "china" | "us" | "...",
    depth: "comprehensive" | "focused"
  }
}
```

### 步骤 2：多源深度检索

```
能力：web_search

检索策略（按优先级分层）：

第一层 — 官方信息源：
  • 公司官方博客/新闻稿
  • 论文原始 PDF（arXiv / 会议 proceedings）
  • 监管机构官方文件
  • 产品官网/文档
  → source_tier: "official"

第二层 — 主流科技媒体：
  • 中文：机器之心、36Kr、量子位、虎嗅
  • 英文：TechCrunch、The Verge、Ars Technica、Wired
  → source_tier: "media"

第三层 — 学术与深度分析：
  • 学术论文引用（Google Scholar / Semantic Scholar）
  • 行业分析报告（CB Insights / 艾瑞等）
  • 技术博客（知名工程师/研究者的深度文章）
  → source_tier: "media" | "academic"

第四层 — 社区与补充视角：
  • Hacker News / Reddit 讨论
  • 知乎高赞分析
  • Twitter/X 上的行业大 V 观点
  → source_tier: "community"

检索输出：每个来源 → content_item（type 归类 + 摘要 + url + source_platform）
```

### 步骤 3：素材组织与覆盖度检查

```
将检索到的素材按分析角度分组。检查覆盖度：

| 检查维度 | 标准 | 不满足时的处理 |
|---------|------|--------------|
| 核心事实覆盖 | 该专题的核心事件/数据是否完整？ | 回到步骤 2 补充检索 |
| 多角度覆盖 | 是否从 ≥ 2 个角度有素材？ | 补充缺失角度的检索 |
| 来源交叉验证 | 核心事实是否 ≥ 2 个独立来源？ | 标注"单一来源，待交叉验证" |
| 信息时效 | 关键信息是否反映了最新状态？ | 标注信息截止日期 |

覆盖度不足 → 回到步骤 2 补充检索
覆盖度满足 → 进入步骤 4
```

### 步骤 4：可选 — 加载分析框架

```
如果专题适合使用分析框架辅助思考，加载对应框架文件。

示例：
  公司分析 → 可选加载 references/frameworks/swot.md
  产业分析 → 可选加载 references/frameworks/pest.md 或 porter-five-forces.md

规则：
- 加载框架是辅助工具，不替代编辑判断
- 数据不足时不用，标注"数据局限"即可
- 框架的结论融入已有分析，不额外增加独立章节
```

### 步骤 5：来源验证

```
能力：web_verification

操作：
1. 对所有 url 非空的 candidate item，browser 打开 URL
2. 通过 3 项硬检查（见 references/source-and-verification.md）
3. 填写 source_verified / link_accessible / source_tier
4. 核心事实必须有 ≥ 2 个独立来源交叉验证
5. 仅有社区来源的核心事实 → 标注"未经独立验证"

专题特有：
- 学术论文：确认 preprint 版本和正式出版版本的一致性
- 公司博客：确认是否为官方域名（防止仿冒）
- 付费墙内容：标注 access_status: "paywalled"
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 选题边界 — 主题类型 + 分析角度 + 范围
2. 素材覆盖度 — 核心事实 X 个 / 已交叉验证 Y 对 / 单一来源 Z 个
3. 来源分布 — 官方 X 条 / 媒体 X 条 / 学术 X 条 / 社区 X 条
4. 覆盖度评估 — 建议采用的结构变体（公司/技术/事件/产业）

回流：用户要求调整 → 回到步骤 1/2/3
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 6：选择结构变体与加载模板

```
加载：templates/content-structure/topic-deep-dive.md

根据步骤 1 确定的主题类型，选择对应结构变体：
  company → 2.1 公司分析
  technology → 2.2 技术分析
  event → 2.3 事件分析
  industry → 2.4 产业分析

可跨变体混合使用元素（如公司分析中加入技术分析的技术原理章节）。
```

### 步骤 7：撰写 body.md

```
输入：selected items + 结构变体 + 编辑判断
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分信息
- ❌ 不含格式指令
- ✅ 纯 Markdown + [^ref-xxx] 脚注
- ✅ 关键事实至少 2 个独立来源
- ✅ 编辑观点与事实明确分开
- ✅ 标注数据局限和不确定性
- ✅ 正文参考所选结构变体，但允许灵活调整

质量要求：
- 开头有力：概览 30 秒内说明主题和重要性
- 逻辑链清晰：每个结论有事实支撑
- 多角度覆盖：不只讲一面
- 诚实标注：不确定的标注"未经独立验证""数据局限"
```

### 步骤 8：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "topic-deep-dive-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, subtitle?, date, created_at,
    content_type: "topic-deep-dive",
    keywords[], language: "zh-CN",
    time_sensitivity: "evergreen" | "monthly",
    data_sources_used[], data_limitations?
  },
  content_items: [所有 item, status 含 selected/rejected/background],
  body_markdown: "<body.md 全文>",
  references: [所有被引用来源],
  editorial_notes: {
    selection_rationale, fact_check_notes,
    data_limitations, risk_notes?, editorial_decisions?
  }
}
```

### 步骤 9：写入 outputs/

```
创建目录：outputs/topic-deep-dive-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 采用的变体类型 + 章节结构 + 字数
2. 覆盖度核查 — 各角度覆盖情况 + 交叉验证对数
3. 事实核查 — 已核查数 / 未验证数
4. 引用完整性 — body.md 脚注 ↔ references[] 匹配率
5. 分析质量 — 是否有逻辑跳跃、是否有无来源的断言

回流：用户要求修改 → 回到步骤 7/8/9
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## 通用专题的差异特征

与 GitHub 项目推荐（同为编辑判断驱动）的不同之处：

| 维度 | 通用专题 | GitHub 项目 |
|------|---------|------------|
| 主题范围 | 公司/技术/产业/事件/趋势 | 仅开源项目 |
| 信息源 | 多源交叉（官方+媒体+学术+社区） | 以 GitHub 为主 |
| 筛选方式 | 按主题覆盖度判断 | 按项目活跃度/文档/实用性判断 |
| 分组 | 按分析角度（灵活） | 按场景/技术栈（必须分组） |
| 结构 | 4 种变体，灵活选择 | 固定：概览→分组→对比→建议 |
| 分析框架 | 可选加载（PEST/五力/SWOT） | 不需要 |
| 时效性 | evergreen（可能跨数年） | evergreen |
| 项目卡片 | 无 | 有（stars/语言/许可证） |

与热点分析的不同之处：

| 维度 | 通用专题 | 热点分析 |
|------|---------|---------|
| 驱动 | 选题先行 | 数据驱动（热点信号触发） |
| 时间窗口 | 不限定 | 固定周期（周/月） |
| 评分 | 无 | 有 |
| 趋势信号 | 如适用则为核心板块 | 可选 |
| 周期对比 | 不需要 | 可选 |
| 叙事线 | 由分析角度决定 | 从预设叙事线中选择 |
