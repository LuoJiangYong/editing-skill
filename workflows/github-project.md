# GitHub 项目推荐 — 工作流

> 路径：workflows/github-project.md
> 职责：定义 GitHub 项目推荐的完整生产流程（L1→L2），从项目发现到 content-package 生成
> Phase：1
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md, templates/content-structure/github-project.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：编辑判断驱动（主题先行 → 深度检索 → 编辑组织）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md`（第三节） |
| L2 | `templates/content-structure/github-project.md` |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `github_discovery` | 从 GitHub 发现符合条件的开源项目 | ≥ 10 个候选项目 |
| `web_verification` | 验证项目链接可访问、README 可读、社区信号交叉验证 | 对所有候选项目执行 |

> **注意：** GitHub 项目推荐不需要 `trending_discovery`（不走热点流），不需要 `trend_signal`（不做趋势分析），不需要 `period_comparison`（不做周期对比）。

---

## L1 工作流：项目发现与筛选

### 步骤 1：明确选题主题

```
输入：用户需求中的主题/场景表达
输出：明确的选题边界

示例输入 → 输出：
  "推荐本地部署大模型的项目" → 主题：本地模型部署与推理
  "最近有什么好的 AI 工具" → 主题：AI 开发者工具（宽泛时缩小范围）
  "推荐 GitHub 上最近火的开源项目" → 主题：近期热门 AI 开源项目（不限场景）

如果用户未指定主题，Agent 主动提议：
  "你关注哪个方向？本地推理 / 模型训练 / 数据工具 / 应用框架 / 综合推荐？"
```

### 步骤 2：发现候选项目

```
能力：github_discovery

操作：
1. 按 data-sources.md → github_discovery 降级链发现项目
2. 候选池门槛：≥ 10 个项目
3. 不满足门槛 → 扩展搜索（放宽关键词 / 扩大时间范围 / 启用下一优先级工具）

搜索策略：
- gh CLI：gh search repos --sort stars --limit 30 "<主题关键词>"
- browser：github.com/trending 按语言/时间筛选
- web_search：中文推荐列表交叉验证

输出：候选项目列表（名称 + 描述 + stars + 语言 + 最近更新 + URL）
```

### 步骤 3：项目筛选

```
对每个候选项目进行全量编辑判断（不评分）：
```

#### 筛选维度

| 维度 | 关注 | 判断标准 |
|------|------|---------|
| 活跃度 | 最近 3 个月内的提交/Release | 有近 3 个月活动 → 通过。超过 6 个月无活动 → 剔除（除非是"完成态"工具） |
| 文档质量 | README 是否清晰、有无 Getting Started | 有清晰 README + 使用说明 → 通过。仅有单行描述 → 剔除 |
| 实用性 | 解决的问题是否明确 | 有明确 use case → 通过。"万能工具"但无实际用例 → 剔除 |
| 社区健康 | Issues/Discussions 活跃度 | 非硬性门槛，但社区死亡的项目降级 |
| 许可证 | 是否为开源许可证 | MIT/Apache-2.0/GPL系列 → 通过。无许可证 → 剔除或标注风险 |

#### 不关注

- ❌ Stars 绝对数量（可刷，只参考不依赖）
- ❌ 是否"大厂出品"（独立开发者项目同等对待）
- ❌ 是否"网红项目"（关注实际质量）

#### 输出

```
每个候选项目：
├── content_item id (item-001, item-002, ...)
├── type: "github"
├── status: selected / background / rejected
├── source_platform: { name: "GitHub", category: "code_host", owner: "<org/user>", display: "GitHub / <org>" }
├── editorial_note：入选理由 / 剔除原因
└── 验证字段待填写（步骤 5）
```

### 步骤 4：分组归类

```
将 selected 项目按适用场景或技术栈分组。

分组原则：
- 每组 2-5 个项目
- 同组内项目功能相近（方便读者对比）
- 分组名体现"解决什么问题"而非"是什么技术"

示例：
  主题"本地模型部署" →
    分组 1：轻量级本地推理（llama.cpp, Ollama, MLX）
    分组 2：高性能服务化部署（vLLM, TGI, Triton）
    分组 3：工具链与辅助（llamafile, Open WebUI）
```

### 步骤 5：来源验证

```
能力：web_verification

操作：
1. 对每个 candidate item，browser 打开 GitHub 仓库页面
2. 验证：仓库存在、README 可读、最近更新日期准确、stars 数确认
3. 通过 3 项硬检查（见 references/source-and-verification.md）
   - GitHub 仓库作为官方信息源：source_tier = "official"
   - 如果有社区评测/讨论作为补充来源，单独建 content_item
4. 填写 source_verified / link_accessible / source_tier

特殊处理：
- 仓库被归档/删除 → link_accessible: "false", 剔除
- 仓库为镜像（非主仓库）→ editorial_note 注明，优先找主仓库
```

### 步骤 6：收集社区信号（可选）

```
通过 web_search 搜索项目名 + "评测/使用体验/对比" 获取社区信号。

每个重点项目的社区评价并入 editorial_note，不作为独立 content_item 评分。
社区信号用途：验证项目实用性、发现已知局限。
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 选题范围 — 主题 + 分组规划
2. 候选统计 — 候选 XX 个 / 入选 XX 个（分 X 组）
3. 来源覆盖 — GitHub 仓库均已验证（URL 可访问），额外社区来源

回流：用户要求调整 → 回到步骤 1/2/3/4
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 7：加载结构模板

```
加载：templates/content-structure/github-project.md
确定：标题格式、分组命名、项目卡片字段
```

### 步骤 8：撰写 body.md

```
输入：selected items（已分组）+ 结构模板
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

章节：
# <标题>
## 一、推荐概览（选题理由 + 筛选说明 + 分组预览）
## 二、项目分组
  每组：
  ### 🚀 <分组名>
  #### <项目名>
  项目卡片内容（见结构模板）：一句话描述 + 属性表 + 核心特性 + 适用场景 + 编辑点评
## 三、对比总览（可选，同组 ≥3 个项目时使用）
## 四、选择建议
## 参考来源

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分
- ❌ 不含格式指令
- ✅ 纯 Markdown + [^ref-xxx] 脚注
- ✅ 所有 stars 数、最近更新日期为真实数据
- ✅ 许可证准确
```

### 步骤 9：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "github-project-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, subtitle?, date, created_at,
    content_type: "github-project",
    keywords[], language: "zh-CN",
    time_sensitivity: "evergreen",
    data_sources_used[], data_limitations?
  },
  content_items: [所有 item, status 含 selected/rejected],
  body_markdown: "<body.md 全文>",
  references: [所有被引用来源, 每个项目至少 1 条],
  editorial_notes: {
    selection_rationale, fact_check_notes,
    data_limitations, risk_notes?, editorial_decisions?
  }
}
```

### 步骤 10：写入 outputs/

```
创建目录：outputs/github-project-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 分组数 + 每组项目数 + 结构
2. 事实核查 — GitHub 数据（stars/日期/许可证）已验证
3. 引用完整性 — 每个项目至少 1 个 reference

回流：用户要求修改 → 回到步骤 8/9/10
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## GitHub 项目推荐的差异特征

与热点分析的不同之处：

| 维度 | 热点分析 | GitHub 项目推荐 |
|------|---------|---------------|
| 评分 | 有（热度+权威+编辑） | 无（全量编辑判断） |
| 趋势分析 | 有 | 无 |
| 周期对比 | 可选 | 无 |
| 分组 | 按主题（可选） | 按场景/技术栈（必须） |
| 时效性 | weekly/monthly | evergreen |
| 项目卡片 | 无 | 有（含 stars/语言/许可证/特性） |
| 对比表格 | 偶尔 | 推荐（同组 ≥3 个项目） |
