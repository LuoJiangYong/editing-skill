# AI 与科技社会洞察 — 工作流

> 路径：workflows/social-insight.md
> 职责：定义 AI 与科技社会洞察（social-insight）的完整生产流程（L1→L2），从选题明确到 content-package 生成
> Phase：3
> 依赖：CONSTITUTION.md, schemas/content-package.md, references/data-sources.md, references/source-and-verification.md, references/content-types-guide.md（第七节）, templates/content-structure/social-insight.md, references/confirmation/information-confirmation.md, references/confirmation/editorial-confirmation.md
> 生产模式：编辑判断驱动（选题先行 → 多源深度检索 → 多角度交叉分析）
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时，按 SKILL.md 定义的顺序加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | `CONSTITUTION.md` |
| L1 | `references/data-sources.md` → `references/source-and-verification.md` → `references/content-types-guide.md`（第七节） |
| L2 | `templates/content-structure/social-insight.md` |
| 可选 | `references/frameworks/` 下适用的社会经济学分析框架（按需加载） |
| 确认 ① | `references/confirmation/information-confirmation.md` |
| 确认 ② | `references/confirmation/editorial-confirmation.md` |

---

## 能力声明

本 workflow 需要以下数据能力（声明能力名，不声明工具名。运行时查 data-sources.md 降级链决定用什么工具）：

| 能力 | 说明 | 门槛 |
|------|------|------|
| `web_search` | 多源深度检索：官方报告、主流媒体、学术论文、社区讨论、统计数据/行业报告 | 至少覆盖 3 类来源，核心观点至少 2 个独立来源支撑 |
| `web_verification` | 来源验证：打开链接确认页面存在、内容匹配、发布时间准确 | 对所有 url 非空的 candidate item 执行 |

> **明确不需要的能力：** `trending_discovery`（不走热点流）、`trend_signal`（不做趋势信号分析）、评分（任何形式）。社会洞察关注的是"这意味着什么"，不依赖热度信号。

---

## L1 工作流：选题明确与信息采集

### 步骤 1：明确选题边界

```
输入：用户需求中的主题/方向表达
输出：明确的选题边界（技术变化 + 社会影响层面 + 分析角度）

如果用户表述模糊，Agent 主动确认：
  "你想从哪个角度切入？对职业的影响 / 对教育的影响 / 对产业生态的影响 / 对监管的影响？"

示例输入 → 输出：
  "AI 编程助手对开发者的影响" → 技术：AI 编程助手 | 影响层面：职业+教育 | 角度：初级/资深开发者的差异化影响
  "开源大模型改变了什么" → 技术：开源大模型 | 影响层面：产业生态+经济 | 角度：对创业公司/闭源厂商/开发者的连锁影响
  "AI 监管各国差异" → 技术：AI 监管政策 | 影响层面：监管+经济 | 角度：三条监管路径的对比与各自影响

输出格式：
{
  technology: "<技术/现象>",
  impact_areas: ["economic", "career", "education", "cultural", "ethical", "regulatory"],
  primary_angles: ["<角度1>", "<角度2>"],
  scope: {
    time_range: {start, end} 或 "evergreen",
    geographic_focus: "global" | "china" | "us" | "comparative",
    depth: "comprehensive" | "focused"
  }
}
```

### 步骤 2：多源深度检索

```
能力：web_search

检索策略（按来源类别分层，每层不少于 2 条）：

第一层 — 官方报告与数据：
  • 政府/国际组织报告（如国家统计局、OECD、World Bank）
  • 行业权威报告（如 CB Insights、麦肯锡、Gartner）
  • 公司官方数据/报告
  → source_tier: "official"
  → 关注可量化的数据和统计

第二层 — 主流媒体深度报道：
  • 中文：财新、36Kr、晚点、财经
  • 英文：NYT、WSJ、The Economist、Wired
  → source_tier: "media"
  → 关注案例、叙事、人物故事

第三层 — 学术研究：
  • Google Scholar / Semantic Scholar 搜索相关论文
  • 关注高引用论文和近期发表的系统性综述
  • 记录论文年份和样本量
  → source_tier: "media" | "academic"
  → 关注因果分析和方法论

第四层 — 社区与一线视角：
  • Hacker News / Reddit 深度讨论
  • 知乎高赞分析
  • 从业者博客/Newsletter
  → source_tier: "community"
  → 关注一线经验和替代视角

第五层 — 数据与统计：
  • Statista、Our World in Data
  • 行业数据库
  • 招聘平台数据（如 LinkedIn、猎聘）
  → 充实"关键数据"引用块

检索输出：每个来源 → content_item（type 归类 + 摘要 + url + source_platform）
```

### 步骤 3：素材组织与覆盖度检查

```
将检索到的素材按分析维度分组。检查覆盖度：

| 检查维度 | 标准 | 不满足时的处理 |
|---------|------|--------------|
| 多源覆盖 | 是否覆盖了至少 3 类来源（官方+媒体+学术+社区+数据）？ | 回到步骤 2 补充缺失的源类型 |
| 维度覆盖 | 是否至少有 2 个分析维度有充分素材？ | 调整选题角度或补充检索 |
| 观点交叉验证 | 核心观点是否至少 2 个独立来源支撑？ | 标注"单一来源，待交叉验证" |
| 数据充分性 | 是否有足够的量化数据支撑分析？ | 标注"数据局限"，不做无数据支撑的分析 |
| 时效性 | 关键信息是否反映了最新状态？ | 标注信息截止日期 |

覆盖度不足 → 回到步骤 2 补充检索
覆盖度满足 → 进入步骤 4
```

### 步骤 4：可选 — 加载分析框架

```
如果选题适合使用社会经济学分析框架辅助思考，加载对应框架文件。

示例：
  职业影响主题 → 可选加载劳动力市场分析相关框架
  教育影响主题 → 可选加载教育公平/数字鸿沟分析框架
  监管影响主题 → 可选加载比较政策分析框架

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
4. 核心观点必须有 ≥ 2 个独立来源交叉验证
5. 仅有社区来源的观点 → 标注"未经独立验证"

社会洞察特有：
- 学术论文：确认 preprint 版本和正式出版版本的一致性，标注年份+样本量
- 统计报告：确认数据来源方和统计口径（如"国家统计局，2025，全国抽样 n=120,000"）
- 付费墙内容（如 Economist、WSJ）：标注 access_status: "paywalled"
- 预印本论文：标注"预印本，未经同行评审"
```

---

## 确认 ①：信息确认（L1→L2）

```
加载：references/confirmation/information-confirmation.md

呈现：
1. 选题边界 — 技术主题 + 影响层面 + 分析角度
2. 素材覆盖度 — 源类型分布（官方 X/媒体 X/学术 X/社区 X/数据 X）
3. 维度覆盖 — 计划展开的分析维度 + 各维度素材充分度
4. 数据局限 — 哪些关键数据缺失、哪些观点仅有单一来源

回流：用户要求调整 → 回到步骤 1/2/3
通过：用户确认"继续" → 进入 L2
```

---

## L2 工作流：内容资产包生成

### 步骤 6：加载结构模板

```
加载：templates/content-structure/social-insight.md
确定：选题适用的章节结构 + 重点展开的分析维度
```

### 步骤 7：撰写 body.md

```
输入：selected items + 结构模板 + 编辑判断
输出：body.md（纯 Markdown，遵循 schemas/content-package.md → 四、body.md 规范）

章节（参考结构，按选题灵活调整）：
# <标题>
## 一、现象：正在发生什么
## 二、多角度分析（选择 2-3 个维度深度展开）
## 三、深层影响与连锁反应
## 四、讨论与开放问题
## 五、展望（可选）
## 参考来源

约束：
- ❌ 不含 HTML/CSS
- ❌ 不含评分信息
- ❌ 不含格式指令
- ✅ 纯 Markdown + [^ref-xxx] 脚注
- ✅ 核心观点至少 2 个独立来源
- ✅ 引用学术研究时标注年份和样本量
- ✅ 编辑观点用"编辑判断："前缀标注
- ✅ 不确定的影响标注"需更长期数据验证"
- ✅ 不做价值判断（"好/坏"），做影响分析（"可能导致…"）

质量要求：
- 现象先行：用具体案例和数据开场
- 逻辑链清晰：每个结论有事实支撑
- 多角度覆盖：至少 2 个分析维度
- 观点与事实分离：编辑观点明确标注
- 诚实标注：不确定的标注"数据局限""未经独立验证""需更长期数据验证"
- 开放结尾：承认分析局限，邀请讨论
```

### 步骤 8：组装 manifest.json

```
按 schemas/content-package.md 结构组装：

{
  schema_version: "1.0.0",
  content_package_id: "social-insight-<YYYY-MM-DD>-<uuid-short>",
  metadata: {
    title, subtitle?, date, created_at,
    content_type: "social-insight",
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
创建目录：outputs/social-insight-<YYYY-MM-DD>/
写入：manifest.json
写入：body.md
```

---

## 确认 ②：内容确认（L2→L3）

```
加载：references/confirmation/editorial-confirmation.md

呈现：
1. 正文结构审查 — 章节结构 + 分析维度覆盖
2. 观点与事实分离 — 编辑观点是否明确标注
3. 事实核查 — 核心观点交叉验证对数 / 未验证数
4. 数据标注 — 学术引用是否有年份+样本量、不确定性是否标注
5. 引用完整性 — body.md 脚注 ↔ references[] 匹配率
6. 局限声明 — 是否诚实标注了数据局限

回流：用户要求修改 → 回到步骤 7/8/9
通过：用户确认"继续" → L2 完成，进入 L3（format-consume）
```

---

## 社会洞察的差异特征

与通用专题（同为编辑判断驱动+多源深度检索+灵活结构）的不同之处：

| 维度 | 社会洞察 | 通用专题 |
|------|---------|---------|
| 分析焦点 | 技术的**外部影响**（对社会/职业/教育/文化/伦理/监管） | 技术/公司/产业**本身**（原理/业务/竞争/趋势） |
| 信息源要求 | 必须有学术/社会研究视角 + 宏观数据 | 以官方+媒体+技术分析为主 |
| 特有步骤 | 多源交叉（官方+媒体+学术+社区+数据，至少 3 类） | 多源深度检索（官方+媒体+学术+社区） |
| 学术引用 | 推荐（标注年份+样本量） | 可选，不强制 |
| 编辑立场 | 不做价值判断，做影响分析 | 可以有编辑观点，标注"编辑判断" |
| 特有约束 | 核心观点 ≥2 个独立来源、不确定标注"需更长期数据验证" | 核心事实 ≥2 个独立来源 |
| 来源类型 | 5 类（官方+媒体+学术+社区+数据报告） | 4 类（官方+媒体+学术+社区） |
| 开放问题 | 必须（讨论局限+开放问题） | 可选 |

与热点分析的不同之处：

| 维度 | 社会洞察 | 热点分析 |
|------|---------|---------|
| 驱动 | 选题先行（因"值得思考"而写） | 数据驱动（因"热度高"而写） |
| 时间窗口 | 不限定 | 固定周期（周/月） |
| 评分 | 无 | 有 |
| 深度 | 深度（2000-5000 字） | 中度（1500-4000 字） |
| 趋势分析 | 以社会影响趋势为主 | 以热度/行业趋势为主 |
| 周期对比 | 不需要 | 可选 |
