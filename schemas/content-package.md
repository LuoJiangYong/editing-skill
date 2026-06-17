# Content-Package — 内容资产包结构定义

> 路径：schemas/content-package.md
> 职责：定义 content-package 的两层模型结构、字段语义、类型约束和持久化格式
> Phase：0
> 依赖：CONSTITUTION.md（设计宪法）
> 被依赖：workflows/*.md, templates/formats/*, SKILL.md
> 最后更新：2026-06-17

---

## 一、核心原则

1. **两层模型**：`content_items[]`（素材层）+ `body_markdown`（成稿），不做 `content_blocks` 中间层。
2. **格式无关**：不存颜色/字号/间距/组件等任何格式信息。
3. **不存评分**：评分/排序/加权逻辑在各 workflow 中完成，content-package 只承载筛选后的最终结果。
4. **目录型输出**：一个 content-package = 一个目录，包含 `manifest.json` + `body.md` + `assets/`。
5. **持久化**：输出到 `outputs/<type>-<date>/`，长期保留，可随时回溯。

---

## 二、目录结构

```
outputs/<type>-<date>/           # 例：outputs/hotspot-analysis-2026-06-17/
├── manifest.json                # 元数据 + content_items[] + references[] + editorial_notes
├── body.md                      # Markdown 成稿（纯内容，不含格式标记外的任何格式信息）
├── editorial-notes.md           # 编辑手记长文版（可选，内容同 manifest.json 中的 editorial_notes）
└── assets/                      # 图片等二进制素材（外挂目录）
    ├── cover.png
    ├── chart-01.png
    └── screenshot-xxx.png
```

### 读写规则

- **manifest.json**：Agent 生成 → 用户确认 → 落盘。结构化的机器可读内容。
- **body.md**：Agent 生成 → 用户确认 → 落盘。人类可读的 Markdown 正文。
- **assets/**：Agent 下载/生成 → 落盘。body.md 中以 `![](assets/xxx.png)` 引用。
- **editorial-notes.md**：可选。当 editorial_notes 较长时单独输出，manifest.json 中引用该文件路径。

---

## 三、manifest.json 结构

### 3.1 顶层字段

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `schema_version` | string | ✅ | `"1.0.0"`，Schema 版本号 |
| `content_package_id` | string | ✅ | 唯一标识，格式：`<type>-<YYYY-MM-DD>-<uuid-short>` |
| `metadata` | object | ✅ | 内容包元信息（见 3.2） |
| `content_items` | array | ✅ | 素材层：所有候选和入选的原始素材 |
| `body_markdown` | string | ✅ | 完整 Markdown 成稿（纯文本，含 ![引用](...) 标注） |
| `references` | array | ✅ | 来源清单 |
| `editorial_notes` | object | ✅ | 全局编辑手记 |

### 3.2 metadata（元信息）

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `title` | string | ✅ | 内容包标题（成品标题） |
| `subtitle` | string | ❌ | 副标题 |
| `date` | string | ✅ | 内容日期，ISO 8601：`"2026-06-17"` |
| `created_at` | string | ✅ | 生成时间戳，ISO 8601：`"2026-06-17T14:30:00+08:00"` |
| `content_type` | string | ✅ | 内容类型枚举，见下方 |
| `keywords` | string[] | ✅ | 关键词标签，用于检索和分类 |
| `language` | string | ✅ | 正文语言，默认 `"zh-CN"` |
| `time_sensitivity` | string | ✅ | 时效性：`"real-time"` \| `"daily"` \| `"weekly"` \| `"monthly"` \| `"evergreen"` |
| `data_sources_used` | string[] | ✅ | 本次实际使用的数据源（能力名，如 `["web_search", "trendradar_trending"]`） |
| `data_limitations` | string | ❌ | 数据局限说明（如"仅覆盖中文源""某工具不可用时降级"） |

#### content_type 枚举（7 种）

| 值 | 中文名 | 说明 |
|----|--------|------|
| `daily-news` | 日报 | 当日 AI/科技要闻速览 |
| `hotspot-analysis` | 周/月热点分析 | 周期性热点复盘 + 趋势 + 排行 |
| `topic-deep-dive` | 通用专题分析 | 公司/技术/事件/产业深度分析 |
| `product-recommendation` | 产品/工具推荐 | AI 工具/科技产品/模型服务推荐 |
| `github-project` | GitHub 项目推荐 | 开源项目/AI 工具库/开发框架推荐 |
| `social-insight` | AI 与科技社会洞察 | 技术变化对社会/职业/教育/文化影响 |
| `content-advice` | 推荐内容建议 | 选题推荐/标题方向/传播角度 |

### 3.3 content_items[]（素材层）

每条素材代表一个原始信息单元——新闻、事件、项目、论文、数据等。

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | 唯一标识，格式：`item-<序号>`（如 `item-001`） |
| `type` | string | ✅ | 素材类型枚举（15 种，见 3.3.1） |
| `title` | string | ✅ | 素材标题（原始标题或摘要提炼标题） |
| `summary` | string | ✅ | 素材摘要（1-3 句话，Agent 从原文提取） |
| `url` | string | ❌ | 原始链接。无链接时留空 `""` |
| `source_platform` | object | ✅ | 来源平台信息（见 3.3.2） |
| `published_at` | string | ❌ | 发布时间，ISO 8601。无法确认时留空 |
| `status` | string | ✅ | 素材状态枚举 |
| `source_verified` | boolean | ✅ | 来源真实性是否通过验证 |
| `link_accessible` | string | ✅ | `"true"` \| `"false"` \| `"no_link"` |
| `source_tier` | string | ✅ | `"official"` \| `"media"` \| `"community"` \| `"unknown"` |
| `editorial_note` | string | ❌ | 编辑评注（为什么选用/为什么剔除/为什么标注为 background） |
| `body_reference` | string | ❌ | 在 body.md 中被引用到的章节/段落标识，便于回溯 |

#### 3.3.1 type 枚举（15 种素材类型）

| 值 | 中文名 | 说明 |
|----|--------|------|
| `news` | 新闻 | 时效性科技/AI 新闻报道 |
| `event` | 事件 | 重大事件（发布/收购/人事/事故） |
| `trend` | 趋势 | 行业趋势/技术发展方向 |
| `product` | 产品 | 产品发布/更新/评测 |
| `paper` | 论文 | 学术论文/技术报告 |
| `github` | 开源项目 | GitHub/GitLab 等平台的仓库 |
| `data_point` | 数据点 | 统计数据/市场报告数字 |
| `quote` | 引用 | 人物公开言论/采访摘录 |
| `opinion` | 评论 | 业界评论/技术分析/专栏文章 |
| `case_study` | 案例 | 应用案例/落地实践 |
| `background` | 背景 | 背景知识/概念解释 |
| `prediction` | 预测 | 行业预测/趋势展望 |
| `comparison` | 对比 | 产品/技术/方案对比 |
| `regulation` | 政策 | 政策法规/监管动态 |
| `interview` | 访谈 | 人物访谈/圆桌讨论 |

#### 3.3.2 source_platform（来源平台）

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 平台/来源名称（如 "GitHub", "TechCrunch", "机器之心"） |
| `category` | string | ✅ | 平台类别：`"official"` \| `"tech_media"` \| `"social"` \| `"academic"` \| `"code_host"` \| `"other"` |
| `owner` | string | ❌ | 发布者/账号（如 "@OpenAI", "张三"） |
| `channel` | string | ❌ | 渠道（如 "blog", "twitter", "newsletter"） |
| `display` | string | ✅ | 展示用文本（如 "GitHub / @OpenAI", "机器之心 微信公众号"） |

#### 3.3.3 status 枚举

| 值 | 说明 |
|----|------|
| `candidate` | 候选素材：进入初筛池 |
| `selected` | 入选素材：最终纳入 content-package |
| `background` | 背景素材：未被直接引用，但提供了上下文 |
| `rejected` | 剔除素材：经筛选后排除（editorial_note 必须说明原因） |

### 3.4 references[]（来源清单）

所有被引用来源的去重列表。注意：`content_items[]` 中的每个 item 可能来自同一个 reference（如同一篇文章的多条信息）。

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | 唯一标识，格式：`ref-<序号>`（如 `ref-001`） |
| `title` | string | ✅ | 来源标题（文章标题/页面标题/报告名） |
| `url` | string | ❌ | 来源 URL。无链接时留空 |
| `source_type` | string | ✅ | 来源类型：`"article"` \| `"paper"` \| `"github_repo"` \| `"official_blog"` \| `"social_post"` \| `"report"` \| `"other"` |
| `source_platform` | object | ✅ | 同 3.3.2 结构 |
| `reliability` | string | ✅ | 可靠性评估：`"high"` \| `"medium"` \| `"low"` \| `"unverified"` |
| `access_status` | string | ✅ | 访问状态：`"accessible"` \| `"inaccessible"` \| `"paywalled"` \| `"not_checked"` |
| `note` | string | ❌ | 备注（如"仅摘要可见"、"需科学上网"） |

### 3.5 editorial_notes（全局编辑手记）

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `selection_rationale` | string | ❌ | 选题理由：为什么这个选题、目标读者 |
| `rejection_summary` | string | ❌ | 剔除说明：主要剔除素材的汇总理由 |
| `fact_check_notes` | string | ❌ | 事实核查记录：交叉验证情况、不确定信息标注 |
| `data_limitations` | string | ❌ | 数据局限：本次采编的数据覆盖面、时效性等局限 |
| `risk_notes` | string | ❌ | 风险提示：可能引起争议的内容、需要声明的事项 |
| `editorial_decisions` | string | ❌ | 关键编辑决策：结构选择、角度取舍、叙事线设计 |
| `references_editorial_notes_file` | string | ❌ | 当内容较长时，指向 editorial-notes.md 的路径 |

---

## 四、body.md 规范

`body.md` 是完整的 Markdown 成稿，直接作为格式消费（L3）的输入。

### 4.1 内容约束

- ✅ **纯 Markdown**：`#` 标题、`##` 二级标题、`**粗体**`、`*斜体*`、`- ` 列表、`> ` 引用、`[text](url)` 链接、`![](assets/xxx.png)` 图片。
- ✅ **引用标注**：关键事实可在正文中用 `[^ref-001]` 脚注标注来源，与 manifest.json 中的 `references[]` 对应。
- ❌ **不含 HTML/CSS**：不写 `<div>`、`<style>`、`style=""` 等 HTML 标记。这些属于格式消费层。
- ❌ **不含评分/排序信息**：不出现"热度：4/5""权威：3/5"等评分。
- ❌ **不含格式指令**：不写"此处放卡片""使用蓝底白字"等格式指令。

### 4.2 结构建议（非强制）

每种内容类型在 `templates/content-structure/<type>.md` 中有推荐结构，body.md 可以参考但不强制遵循。Agent 根据实际素材灵活调整。

### 4.3 示例片段

```markdown
# 2026年6月第三周 AI 热点回顾

## 一、本周概览

本周 AI 领域延续了多模态模型竞赛的态势。OpenAI 发布三款实时音频模型，
Meta 开源新一代视觉语言模型，国内多家厂商跟进多模态能力。

## 二、热点排行

### 1. OpenAI 发布 GPT-5o 实时音频系列

OpenAI 于 6 月 15 日发布三款音频模型：GPT-5o-mini-speech、GPT-5o-speech
和 GPT-5o-speech-pro[^ref-001]……

### 2. Meta 开源 VLM-3 视觉语言模型

Meta 于 6 月 16 日开源 VLM-3 系列[^ref-002]，涵盖 7B/13B/34B 三个规模……
```

---

## 五、字段互斥与完整性规则

### 5.1 状态流转

```
candidate → selected （入选）
candidate → rejected  （剔除）
candidate → background（背景）
```

- `selected` 的 item 必须在 body.md 中有对应引用。
- `rejected` 的 item 必须有 `editorial_note` 说明剔除原因。
- `background` 的 item 不要求被 body.md 引用。

### 5.2 引用一致性

- body.md 中每个 `[^ref-xxx]` 脚注 → 必须在 `references[]` 中存在对应的 `id`。
- body.md 中每个 `[](assets/xxx)` 图片 → 必须在 `assets/` 目录中存在对应文件。
- `content_items[]` 中每个 `status=selected` 且 `url` 非空的 item → 必须在 `references[]` 中有对应的引用记录。

### 5.3 来源验证完整性

- 每个 `content_item` 的 `source_verified`、`link_accessible`、`source_tier` 必须填写。
- `source_verified=false` 或 `link_accessible="false"` 的 item → 必须在 `editorial_note` 中注明处理方式（如"已通过其他来源交叉验证""已标注待核实"）。

---

## 六、Schema 版本兼容

| 版本 | 日期 | 变更 |
|------|------|------|
| `1.0.0` | 2026-06-17 | 初始版本，两层模型（items + markdown），7 种 content_type 枚举，15 种 item type 枚举 |

### 兼容策略

- **向前兼容**：新字段必须设默认值，旧 reader 忽略未知字段不报错。
- **破坏性变更**：删字段 / 改枚举值 / 改字段语义 → 升级 `schema_version` 主版本号（`2.0.0`）。
- **非破坏性变更**：加字段（有默认值）/ 加枚举值 → 升级次版本号（`1.1.0`）。

---

## 七、完整 manifest.json 示例

```json
{
  "schema_version": "1.0.0",
  "content_package_id": "hotspot-analysis-2026-06-17-a1b2c3",
  "metadata": {
    "title": "2026年6月第三周 AI 热点回顾",
    "subtitle": "多模态竞赛加速，开源生态持续繁荣",
    "date": "2026-06-17",
    "created_at": "2026-06-17T15:00:00+08:00",
    "content_type": "hotspot-analysis",
    "keywords": ["AI", "多模态", "开源", "OpenAI", "Meta", "大模型"],
    "language": "zh-CN",
    "time_sensitivity": "weekly",
    "data_sources_used": ["web_search", "trendradar_trending"],
    "data_limitations": "仅覆盖中英文主流科技媒体，未覆盖学术预印本和日韩来源"
  },
  "content_items": [
    {
      "id": "item-001",
      "type": "product",
      "title": "OpenAI 发布 GPT-5o 实时音频系列",
      "summary": "OpenAI 于 6 月 15 日发布三款音频模型，覆盖轻量到专业级，低延迟实时语音交互。",
      "url": "https://openai.com/blog/gpt-5o-audio-models",
      "source_platform": {
        "name": "OpenAI Blog",
        "category": "official",
        "owner": "OpenAI",
        "channel": "blog",
        "display": "OpenAI 官方博客"
      },
      "published_at": "2026-06-15",
      "status": "selected",
      "source_verified": true,
      "link_accessible": true,
      "source_tier": "official",
      "editorial_note": "本周最重要发布，官方一手信息，多个科技媒体交叉验证",
      "body_reference": "## 二、1. OpenAI 发布 GPT-5o"
    }
  ],
  "body_markdown": "# 2026年6月第三周 AI 热点回顾\n\n## 一、本周概览\n...",
  "references": [
    {
      "id": "ref-001",
      "title": "Introducing GPT-5o Audio Models",
      "url": "https://openai.com/blog/gpt-5o-audio-models",
      "source_type": "official_blog",
      "source_platform": {
        "name": "OpenAI Blog",
        "category": "official",
        "owner": "OpenAI",
        "channel": "blog",
        "display": "OpenAI 官方博客"
      },
      "reliability": "high",
      "access_status": "accessible"
    }
  ],
  "editorial_notes": {
    "selection_rationale": "本周是 6 月第三周，重点聚焦多模态模型竞赛和开源生态动态。",
    "fact_check_notes": "OpenAI 发布时间已通过官方博客确认。Meta VLM-3 已通过 GitHub Release 确认。",
    "data_limitations": "本周 TrendRadar 中文源权重较高，英文源可能覆盖不足。",
    "risk_notes": "部分国内厂商的动态仅来自媒体报道，未经官方确认，已标注。",
    "editorial_decisions": "本周将'开源动态'独立成章，与'商业发布'分开，反映开源/闭源两条线。"
  }
}
```

---

*本 Schema 由 CONSTITUTION.md v1.0 派生。与宪法冲突者，以宪法为准。*
