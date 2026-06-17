# Editing Skill — 设计宪法（CONSTITUTION）

> 本文档定义 Editing Skill 的最高设计原则、架构约束和禁止模式。
> 所有 Phase、所有文件、所有决策均受此宪法约束。与本文冲突者，以本文为准。
>
> 固化日期：2026-06-17 | 版本：v1.0 | 状态：Phase 0 生效

---

## 一、5 条设计原则

### 1. 高内聚 HIGH COHESION — One file, one job.

- 每种内容类型（日报/周月热点/专题/GitHub/产品/社会洞察/内容建议）的**编辑知识**只存在于一个 workflow 文件和一个结构模板中，绝不重复。
- 格式消费（format-consume）的知识只存在于对应格式目录（templates/formats/<format>/），不散落在 workflow 或 content-package 中。
- 来源验证规则只存在于 references/source-and-verification.md，不在任何 workflow 中重写。

### 2. 低耦合 LOW COUPLING — 改一个不动其他。

- **格式**与 **content-package** 零耦合：content-package 不存颜色/字号/组件信息，格式不存标题/来源/评分。
- 换数据获取工具 → **只改** `references/data-sources.md` 中的降级链，不动 workflow。
- 换内容筛选标准 → **只改**对应 workflow 的评分/筛选逻辑，不动 content-package schema。
- 新增格式 → **只新增** `templates/formats/<new-format>/` 目录，不动任何已有文件。

### 3. 可扩展 EXTENSIBLE — 新能力 = 新文件 + 一行路由。

- **新内容类型** = 新 workflow + 新结构模板 + SKILL.md 路由表加一行 + content-package schema 枚举加一个值。
- **新格式** = 复制 `_TEMPLATE/` 目录 → 填入 design.md → 完成。不修改 content-package schema、不影响已有格式。
- **新分析框架** = 在 `references/frameworks/` 下新增一个 `.md` 文件，workflow 中加一行加载声明。
- 零代码改动，零已有文件修改。

### 4. 易维护 MAINTAINABLE — 可追溯性好过聪明。

- **每个产出来源可追溯**：最终 HTML 中的每段内容 → body.md（Markdown 成稿） → content_items[]（素材层） → 有 url + source_platform 的原始来源。
- **`body.md` 和 `manifest.json` 是内容真源**（source of truth），HTML 是渲染产物。修改内容必须回写 body.md/manifest.json 后重新渲染，不允许在 HTML 上改了不回写。
- **编辑手记可读**：每个 content_item 有 editorial_note（为什么入选/剔除），全局 editorial_notes 记录取舍逻辑、数据局限、风险提示。
- **outputs/ 持久化**：每个 content-package 持久化到 `outputs/<type>-<date>/`，可随时回溯。

### 5. 高效简洁 LEAN — <3 个消费者就不抽象。

- **两层内容模型足够**：`content_items[]`（素材层）+ `body_markdown`（成稿），不做 `content_blocks` 中间层。
- **不做 provider 契约层**：只有 1 个 data consumer（Agent）时，不做 adapter/field_mapping/provider 接口抽象。工具选择通过 data-sources.md 降级链解决。
- **不做维护元数据体系**：Phase 0 不建 registry.yaml / fields.yaml / dependencies.yaml / CHANGELOG.md。复杂度不值得。用 git log + 文件头部注释替代。
- **编辑角色系统砍掉**：全局编辑标准放在本宪法，类型特定的筛选/取舍/组织逻辑内聚在各自 workflow 中。

---

## 二、目录布局

```
editing-skill/
├── CONSTITUTION.md                   # 本文件：设计宪法
├── SKILL.md                          # 路由中枢（≤3500 chars 正文）
│
├── schemas/
│   └── content-package.md            # 内容资产包结构定义（两层模型）
│
├── references/                       # 领域知识（按需加载）
│   ├── content-types-guide.md        # 7 类内容定义 + 适用场景 + 差异
│   ├── data-sources.md               # 数据源清单（能力→工具降级链）
│   ├── source-and-verification.md    # 来源分类与验证规则
│   ├── frameworks/                   # 分析框架知识库（按需扩展）
│   │   └── _README.md                # 挂载点说明
│   └── confirmation/                 # 确认协议
│       ├── information-confirmation.md
│       ├── editorial-confirmation.md
│       └── format-confirmation.md
│
├── templates/                        # 模板层（内容无关，纯格式/结构）
│   ├── content-structure/            # 正文结构指南（每种类型 1 个）
│   │   ├── daily-news.md
│   │   ├── hotspot-analysis.md
│   │   ├── topic-deep-dive.md
│   │   ├── product-recommendation.md
│   │   ├── github-project.md
│   │   ├── social-insight.md
│   │   └── content-advice.md
│   ├── formats/                      # 格式消费模板（每个格式=独立目录）
│   │   ├── tech-magazine/            # 第一阶段格式
│   │   │   ├── design.md             # 设计规范（配色/字号/组件/思想）
│   │   │   ├── REFERENCE.md          # 参考来源（文章链接 + 视觉特征）
│   │   │   └── example.html          # 完整样例
│   │   └── _TEMPLATE/                # 新建格式模板
│   │       ├── design.md
│   │       ├── REFERENCE.md
│   │       └── example.html
│   ├── panels/                       # 编辑面板模板（Phase 2）
│   │   └── review-panel.html
│   └── publishing-advice.md          # 发布建议卡模板
│
├── workflows/                        # 工作流定义（7 种 + 格式消费入口）
│   ├── daily-news.md
│   ├── hotspot-analysis.md           # 周/月热点分析
│   ├── topic-deep-dive.md            # 通用专题深度分析
│   ├── product-recommendation.md
│   ├── github-project.md
│   ├── social-insight.md
│   ├── content-advice.md
│   └── format-consume.md             # 格式消费总入口（L3）
│
├── examples/                         # 完整样例（可运行）
│   ├── github-project-example/
│   │   ├── manifest.json
│   │   └── body.md
│   └── hotspot-analysis-example/
│       ├── manifest.json
│       └── body.md
│
├── outputs/                          # 运行时产出（持久化，.gitignore）
│   └── <type>-<date>/
│       ├── manifest.json
│       ├── body.md
│       ├── assets/
│       ├── output-review.html
│       └── output.html
│
└── .gitignore
```

---

## 三、数据流

```
用户需求
    │
    ▼
┌─────────────────────────────────────────┐
│          SKILL.md 路由中枢                │
│  日报 / 周月热点 / 专题 / 产品推荐 /       │
│  GitHub / 社会洞察 / 内容建议             │
│  → 格式消费 → 发布建议                    │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│  L1: 内容采编 + 信息获取                  │
│  • 用户提供材料 / web_search 补充          │
│  • 日报/周报：可选 TrendRadar 热点         │
│  • 候选池 → 筛选 → 来源归集               │
│  ➤ ① 信息确认关卡 ←→ L1 回流              │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│  L2: 内容资产包 (content-package)         │
│  • content_items[]（素材层）              │
│  • body_markdown（Markdown 成稿）         │
│  • references[]（来源清单）               │
│  • editorial_notes（编辑手记）            │
│  ➤ ② 内容确认关卡 ←→ L1/L2 回流          │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│  L3: 格式消费                             │
│  • 读取 confirmed content-package        │
│  • 匹配预设格式模板 → 渲染                │
│  • 生成 output-review.html + output.html  │
│  ➤ ③ 格式确认关卡 ←→ L3 回流             │
└─────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────┐
│  L4: 发布建议                             │
│  • 发布时间/平台/标题变体/封面方向         │
│  ➤ 最终交付：content-package + HTML      │
│     + 发布建议卡                          │
└─────────────────────────────────────────┘
```

### 三级输出体系

```
content-package（唯一内容资产母版，格式无关）
    │
    ├── L3: 格式消费（本 Skill 负责）
    │   ├── 文本型平台: 微信公众号/知乎/Newsletter → HTML
    │   └── 图片型平台: 小红书 → Brief/提示词 → 对接 infographic-carousel
    │
    ├── L3b: 内容转化（预留接口，不实现）
    │   └── 下游 Skill 消费 content-package → 音频/视频
    │
    └── L4: 发布建议（本 Skill 负责）
```

---

## 四、禁止模式

这些是**硬性禁止项**。违反任一条，产出无效。

### 数据与来源

- ❌ **不硬编码 TrendRadar 调用**：workflow 声明"我需要什么数据能力"，不声明"用 TrendRadar"。工具选择走 data-sources.md 降级链。
- ❌ **不杜撰新闻/链接/数据**：所有内容必须有可追溯的真实来源。不确定的信息标记"未经独立验证"。
- ❌ **不过度依赖单一来源**：关键事实至少交叉验证 2 个独立来源。

### 架构与抽象

- ❌ **不用三层内容模型**：`content_items[]` + `body_markdown` 两层足够。不引入 `content_blocks` 中间层。
- ❌ **不把格式信息写入 content-package**：颜色/字号/间距/组件选择属于格式层，不属于内容层。
- ❌ **content-package 不存评分**：评分在各 workflow 层完成。content-package 只承载已筛选后的结果。
- ❌ **<3 个消费者不抽象**：不做 provider 契约层、不做 adapter 接口、不做发布映射中间层。

### 流程与确认

- ❌ **不跳过确认关卡**：三层确认（信息→内容→格式）必须逐层通过，不可跳过。用户明确说"跳过"时除外。
- ❌ **不把 HTML 当源文件回写**：内容修改必须回写 body.md + manifest.json → 重新渲染 HTML。不允许只改 HTML 不回写。
- ❌ **不做自动推送/发送**：任何产出需用户明确确认后才发往外部平台。

---

## 五、知识库扩展规则

`references/frameworks/` 是分析框架知识库的挂载点。规则：

1. **新增框架** = 新增一个 `.md` 文件，放入 `references/frameworks/` 目录。
2. **workflow 中声明** = 在工作流文件中加一行加载声明（如 `加载 references/frameworks/porter-five-forces.md`）。
3. **不创建新协议** = 沿用已有的 "加载 references/<path>" 模式。
4. **可选、不强制** = 框架是辅助工具，不替代编辑判断。数据不足时宁可标"数据局限"也不强行套框架。
5. **不纳入 content-package schema** = 框架是编辑阶段的工具，不是内容资产的字段。

---

## 六、来源验证规则

每条被引用的信息必须通过三项硬检查：

### 1. 来源真实（Source Authenticity）

- 有**明确的发布主体**（公司/机构/个人名称）。
- 有**明确的发布时间**（至少到日）。
- **不是二手转述链**（"据外媒报道""网友称""多方消息"）。
- 无法确认主体或时间 → 标记 `source_verified: false`，editorial_note 写明"来源待核实"。

### 2. 链接可访问（Link Accessibility）

- Agent 用 browser 打开 URL → 确认页面存在且内容匹配摘要。
- 链接失效/404/需登录 → 标记 `link_accessible: false`。
- 无链接 → 标记 `link_accessible: no_link`。

### 3. 权威优先（Authority Tier）

同一事件有多个来源时，优先选：

```
官方公告/论文/公司博客  >  主流科技媒体（TechCrunch/36Kr/机器之心）  >  社区讨论/个人博客
```

标记方式：`source_tier: official | media | community | unknown`

- 有官方来源时**不引用**二手报道。
- 仅有社区来源 → 标注"未经独立验证"。
- 权威源不可访问时 → 可用二手源但 editorial_note 说明。

---

## 七、元数据治理

### 文件级元数据

每个 `.md` 文件头部包含：

```markdown
> 路径：<vault-relative-path>
> 职责：<一句话>
> Phase：<0|1|2|3>
> 依赖：[<list of file paths this file depends on>]
> 被依赖：[<list of file paths that depend on this file>]
> 最后更新：<YYYY-MM-DD>
```

### 版本管理

- 通过 git 管理，不建 CHANGELOG.md。
- 每个 commit 必须有清晰的 message：`feat(phase<N>): <描述>`。
- 每完成一个文件 → git add → git commit → git push。

### 不建单独的元数据体系

- 不做 registry.yaml / fields.yaml / dependencies.yaml。
- 用文件头部注释 + `git log` 替代，避免元数据本身变成维护负担。

---

## 八、变更控制

1. **修改本宪法** = 需用户明确确认，讨论影响范围，同时更新相关文件。
2. **新增内容类型** = 需更新本宪法第二节（目录布局）和第五节（内容类型体系），但不需要修改原则。
3. **踩线判定** = 当设计方案与宪法原则可能冲突时，引用具体条款讨论。

---

*本宪法由 architecture-baseline-v1.0.md 的 12 项核心决策 + 3 轮讨论校准后固化。*
*与本宪法冲突的任何文档/设计/实现，以本宪法为准。*
