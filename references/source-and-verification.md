# 来源分类与验证规则

> 路径：references/source-and-verification.md
> 职责：定义来源分类体系、三项硬检查规则和标记方式。所有内容类型的来源验证均以此文档为准。
> Phase：1
> 依赖：CONSTITUTION.md（第六节：来源验证规则）
> 被依赖：workflows/*.md, references/content-types-guide.md
> 最后更新：2026-06-17

---

## 一、来源分类体系

### 1.1 权威层级（source_tier）

```
官方公告/论文/公司博客  >  主流科技媒体  >  社区讨论/个人博客
    official                media            community
```

### 1.2 平台类别（source_platform.category）

| 类别 | 标识 | 典型来源 | 可靠性 |
|------|------|---------|-------|
| 官方发布 | `official` | 公司博客、产品官网、政府公告、学术论文 | 高（原始信息） |
| 科技媒体 | `tech_media` | TechCrunch、The Verge、36Kr、机器之心、量子位 | 中-高（有编辑审核） |
| 社交媒体 | `social` | Twitter/X、微博、知乎、Reddit、即刻 | 中（需交叉验证） |
| 学术 | `academic` | arXiv、IEEE、Google Scholar | 高（同行评审/预印本，需注明） |
| 代码托管 | `code_host` | GitHub、GitLab、HuggingFace | 高（原始信息，但需注意可信度——star 数可刷） |
| 其他 | `other` | 个人博客、Newsletter、播客、论坛 | 低-中（需额外验证） |

### 1.3 来源类型（用于 references[].source_type）

| 类型 | 说明 |
|------|------|
| `article` | 科技媒体/博客文章 |
| `paper` | 学术论文/技术报告 |
| `github_repo` | GitHub/GitLab 仓库 |
| `official_blog` | 公司/产品官方博客 |
| `social_post` | 社交媒体帖子 |
| `report` | 行业报告/市场研究 |
| `other` | 其他类型 |

---

## 二、三项硬检查

每条被引用的信息必须通过以下三项检查。检查结果写入 `content_item` 的三个验证字段。

### 2.1 检查一：来源真实（Source Authenticity）

**判断标准：**

- ✅ 有**明确的发布主体**（公司/机构/个人名称）
- ✅ 有**明确的发布时间**（至少精确到日）
- ✅ **不是二手转述链**（"据外媒报道""网友称""多方消息""知情人士透露"）

**常见不可信模式：**

| 模式 | 示例 | 判定 |
|------|------|------|
| 匿名信源 | "据知情人士透露" | ❌ source_verified: false |
| 转述链 | "据外媒报道……"但未给出原始报道链接 | ❌ source_verified: false |
| 无时间 | "近日""最新消息"无具体日期 | ⚠️ 如能交叉验证到确定日期则通过，否则 false |
| AI 生成痕迹 | 无发布主体、段落结构异常、数据日期冲突 | ❌ source_verified: false |

**处理方式：**

```
source_verified: true  → 正常引用
source_verified: false → editorial_note 必须写明原因，且 source_tier 至少降一级
```

### 2.2 检查二：链接可访问（Link Accessibility）

**执行方式：** Agent 用 browser 打开 URL → 确认页面存在且内容与摘要匹配。

**判定：**

| 情况 | 标记 | 处理 |
|------|------|------|
| URL 可正常访问，内容匹配 | `link_accessible: "true"` | 正常引用 |
| URL 返回 404/5xx / 需登录 / 被墙 | `link_accessible: "false"` | editorial_note 注明。如有其他来源交叉验证，可降级引用 |
| 素材本身无链接（如来自趋势工具的标题摘要） | `link_accessible: "no_link"` | editorial_note 注明"无原始链接"。不标记 source_verified |

**批量验证建议：** 当 content_items 超过 10 条时，先验证 source_tier 为 `official` 和 `media` 的链接，`community` 来源后验证。

### 2.3 检查三：权威优先（Authority Tier）

**选择优先级：** 同一事件有多个来源时，优先选：

1. **官方公告/论文/公司博客**（`official`）— 一手信息
2. **主流科技媒体**（`media`）— 经过编辑审核
3. **社区讨论/个人博客**（`community`）— 需交叉验证

**硬规则：**

- 有官方来源时**不引用**二手报道作为主要引用（可作为补充视角）。
- 仅有社区来源 → 标注"未经独立验证"，source_tier 标记为 `unknown`。
- 权威源不可访问时 → 可用二手源，但 editorial_note 说明"权威源不可访问，使用二手源"。
- 无法判定层级 → 标记 `unknown`。

---

## 三、标记汇总

每个 `content_item` 的三个验证字段：

| 字段 | 类型 | 值 |
|------|------|-----|
| `source_verified` | boolean | `true` / `false` |
| `link_accessible` | string | `"true"` / `"false"` / `"no_link"` |
| `source_tier` | string | `"official"` / `"media"` / `"community"` / `"unknown"` |

每个 `reference` 的可靠性评估：

| 字段 | 类型 | 值 |
|------|------|-----|
| `reliability` | string | `"high"` / `"medium"` / `"low"` / `"unverified"` |
| `access_status` | string | `"accessible"` / `"inaccessible"` / `"paywalled"` / `"not_checked"` |

### 可靠性判定映射

| source_tier | 默认 reliability | 说明 |
|------------|-----------------|------|
| `official` | `high` | 官方一手信息，确定性高 |
| `media` | `medium` | 有编辑审核，但仍可能有选择性报道 |
| `community` | `low` | 需交叉验证 |
| `unknown` | `unverified` | 无法判定来源层级 |

如有交叉验证（≥2 个独立来源确认同一事实），`reliability` 可上调一级。

---

## 四、特殊场景处理

### 4.1 预印本（arXiv 等）

- `source_tier`: 标记为 `official`（原始论文），但 editorial_note 注明"预印本，未经同行评审"。
- `source_platform.category`: `academic`。

### 4.2 GitHub 项目

- 仓库 README 本身是官方信息来源：`source_tier: official`, `source_platform.category: code_host`。
- 但 stars / forks 数可能被刷，不视为硬指标。以代码质量、文档完整性、维护频率为判断依据。

### 4.3 社交媒体爆料

- 知名人物（如 Sam Altman、Elon Musk）的官方账号：可视为 `official`，但仍需注明"社交媒体发布，非正式公告"。
- 非知名账号：`source_tier: community`，必须交叉验证。

### 4.4 付费墙内容

- `access_status: "paywalled"`，但如能通过摘要或免费预览确认核心信息，仍可引用。
- editorial_note 注明"付费墙内容，仅可见摘要"。

---

## 五、来源验证流程速查

```
每条 content_item (status = candidate/selected):
  │
  ├── 1. 检查发布主体 + 发布时间
  │    ├── 都明确 → source_verified = true
  │    └── 任一缺失/模糊 → source_verified = false，写 editorial_note
  │
  ├── 2. browser 打开 URL（如有）
  │    ├── 可访问且内容匹配 → link_accessible = "true"
  │    ├── 不可访问 → link_accessible = "false"，写 editorial_note
  │    └── 无 URL → link_accessible = "no_link"
  │
  └── 3. 判定权威层级
       ├── official / media / community / unknown
       └── 有多个来源 → 选权威最高的为主引用
```
