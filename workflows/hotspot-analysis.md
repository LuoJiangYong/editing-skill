# 周/月热点分析 — 工作流

> 路径：workflows/hotspot-analysis.md
> 职责：定义周/月热点分析的完整生产流程（L1→L2），从热点获取到 content-package 生成
> Phase：1
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md, templates/content-structure/hotspot-analysis.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：标准化生产（数据驱动 → 评分筛选 → 结构模板）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md` |
| L2 | `templates/content-structure/hotspot-analysis.md` |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `date_resolution` | 把"本周""本月"等用户表达转为标准日期范围 | — |
| `trending_discovery` | AI/科技领域本期热度最高的新闻标题、排名、平台覆盖 | ≥ 3 个平台 / ≥ 30 条候选 |
| `trend_signal` | 核心话题的热度变化曲线和趋势判断 | 可选，数据不足时不输出趋势信号块 |
| `period_comparison` | 本期 vs 上期话题消长 | 可选（周报推荐，月报可选） |
| `web_verification` | 来源验证：打开链接确认页面存在 | 对所有 url 非空的 candidate item 执行 |

---

## L1 工作流：内容采编

### 步骤 1：解析时间范围

```
输入：用户需求中的时间表达（"本周""本月""6月"等）
输出：标准日期范围 {start: "YYYY-MM-DD", end: "YYYY-MM-DD"}

能力：date_resolution
```

### 步骤 2：获取热点候选

```
能力：trending_discovery

操作：
1. 按 data-sources.md → trending_discovery 降级链获取新闻候选
2. 候选池门槛：≥ 30 条、≥ 3 个平台
3. 不满足门槛 → 扩大检索（换平台/换关键词/放大时间窗口）
4. 候选不足时 → 降级到下一优先级工具补充

输出：候选新闻列表（标题 + 摘要 + 平台 + 日期）
```

### 步骤 3：评分与筛选

仅对候选池中来源初步过关的条目进行评分。

#### 评分维度

| 维度 | 分值 | 评分标准 |
|------|------|---------|
| 热度 | 1-5 | 提及次数、跨平台覆盖度、趋势方向（上升=高分） |
| 来源权威 | 1-5 | official=5, tech_media=4, academic=4, social=2-3, unknown=1 |
| 编辑判断 | 1-5 | 行业重要性、用户兴趣对齐、叙事线匹配度 |

#### 入选规则

```
总分 = 热度 + 来源权威 + 编辑判断
总分 ≥ 10 → selected
总分 < 10 且 编辑判断 == 5 → 编辑强制入选（标记 editorial_note）
总分 < 10 → rejected（editorial_note 说明原因）
```

#### 输出

```
每个候选 item：
├── 分配 content_item id (item-001, item-002, ...)
├── type 归类（news/event/trend/product/paper 等 15 种）
├── 评分：热度 X / 来源权威 X / 编辑判断 X = 总分 XX
├── status: selected / rejected / background
└── editorial_note：入选理由 / 剔除原因
```

### 步骤 4：趋势分析（可选）

```
能力：trend_signal

操作：
1. 从 selected items 中提取核心话题
2. 判断每个话题的趋势方向（升温/初见/高峰/降温）
3. 数据不足时省略趋势信号块，不强行编造

输出：趋势信号列表（每个含：话题名、信号强度、支撑证据）
```

### 步骤 5：周期对比（可选）

```
能力：period_comparison

操作：
1. 获取上一期（上周/上月）的热点数据
2. 对比本期 vs 上期话题消长
3. 识别：新出现话题、热度上升话题、热度衰减话题

输出：周期对比摘要（合并到趋势分析中或独立成块）
```

### 步骤 6：来源验证

```
能力：web_verification

操作：
1. 对每个 url 非空的 candidate item，browser 打开 URL
2. 通过 3 项硬检查（见 references/source-and-verification.md）
3. 填写 source_verified / link_accessible / source_tier
4. 未通过验证的在 editorial_note 中注明处理方式

输出：每个 item 的验证字段已填写
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 选题范围 — 时间范围 + 主题聚焦 + 候选统计
2. 候选数量 — 候选池 XX 条 / 入选 XX 条 / 剔除 XX 条 / 背景 XX 条
3. 来源覆盖 — 平台清单 + 权威层级分布 + 验证状态

回流：用户要求调整 → 回到步骤 2/3/4/5/6
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 7：加载结构模板

```
加载：templates/content-structure/hotspot-analysis.md
确定：章节结构、叙事线、信号块类型
```

### 步骤 8：选择主叙事线

从以下叙事线中选择最能串起本期热点的：

- 竞赛加速 / 开源 vs 闭源 / 技术落地 / 监管博弈 / 范式转变
- 如果热点分散无统一叙事，可省略，使用"本期热点回顾"标准结构

### 步骤 9：撰写 body.md

```
输入：selected items + 结构模板 + 主叙事线
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

章节：
# 标题
## 一、本期概览（含关键词标签）
## 二、热点排行（按评分降序）
  每条：### N. 标题 → 概述 → 细节 → 编辑视角[可选] → 来源引用
## 三、趋势分析（如有数据）
  每个趋势：信号强度 + 支撑证据
## 四、下周/下月展望（如有已知事件）
## 参考来源（自动生成）

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分信息
- ❌ 不含格式指令
- ✅ 纯 Markdown + [^ref-xxx] 脚注
```

### 步骤 10：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "hotspot-analysis-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, subtitle?, date, created_at,
    content_type: "hotspot-analysis",
    keywords[], language: "zh-CN",
    time_sensitivity: "weekly" | "monthly",
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

### 步骤 11：写入 outputs/

```
创建目录：outputs/hotspot-analysis-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 章节结构 + 字数 + 建议
2. 事实核查 — 已核查数 / 未验证数 / 交叉验证对数
3. 引用完整性 — body.md 脚注 ↔ references[] 匹配率
4. 图片资源（如有）

回流：用户要求修改 → 回到步骤 9/10/11 修改后重新确认
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## 快速模式

当以下条件全部满足时，可跳过确认关卡：

1. 用户明确说"快速模式""跳过确认""直接生成"
2. 或：同一用户第 N 次执行热点分析，流程和来源偏好已稳定

快速模式下，Agent 仍然完成所有步骤（包括来源验证），仅跳过向用户呈现确认块。在 body.md 生成完成后，直接进入 L3 格式消费。
