# 数据源清单（Data Sources）

> 路径：references/data-sources.md
> 职责：定义能力→工具降级链，Agent 运行时查此表决定用哪个工具获取数据
> Phase：1
> 依赖：CONSTITUTION.md（禁止模式：不硬编码工具名）
> 被依赖：workflows/*.md, SKILL.md
> 最后更新：2026-06-17

---

## 一、核心原则

1. **workflow 声明能力，不声明工具**：workflow 写"我需要 `trending_discovery` 能力"，不写"调用 TrendRadar"。
2. **按优先级降级**：Agent 按本表从 ① → ② → ③ 依次尝试，上层不可用时自动降级。
3. **全部不可用时标记**：写入 `data_limitations` 字段，能力标记为 `unavailable`，不阻塞生产。
4. **通用工具优先**：通用工具（web_search / browser / gh CLI）优先于专用工具（TrendRadar），减少单点依赖。
5. **能力定义稳定，工具可替换**：换工具只改本表优先级链，不动任何 workflow。

---

## 二、能力 → 工具降级链

| 能力 | 工具（按优先级） | 说明 |
|------|-----------------|------|
| `date_resolution` | ① LLM 直接计算<br>② TrendRadar `resolve_date_range` | LLM 直接计算覆盖 95% 场景。当用户使用模糊表达（"上周""本月"）且需要和其他 TrendRadar 工具共用日期范围时，优先用 ② 保证一致性。 |
| `trending_discovery` | ① web_search（site:36kr.com OR site:jiqizhixin.com 等）<br>② browser 打开 HN / ProductHunt / 少数派<br>③ TrendRadar `get_latest_news` + `get_trending_topics` | 热点发现的核心是覆盖多个平台。web_search 覆盖中文科技媒体，browser 补充英文源，TrendRadar 作为聚合加速器减少逐站检索。门槛：至少覆盖 3 个平台 / 20 条候选。 |
| `trend_signal` | ① LLM 从热点结果归纳趋势<br>② TrendRadar `analyze_topic_trend`(mode=trend)<br>③ TrendRadar `analyze_sentiment` | 趋势判断本质是编辑判断——LLM 归纳优先。数据不足时不输出趋势信号块，不强行编造。 |
| `period_comparison` | ① 两次 web_search 结果对比<br>② TrendRadar `compare_periods` | 将本周/本月热点与上周/上月对比。web_search 可通过切换时间范围实现。TrendRadar 提供结构化对比结果。 |
| `github_discovery` | ① `gh search repos --sort stars` CLI<br>② browser github.com/trending<br>③ web_search "github 开源 AI 项目 推荐" | 不需要 TrendRadar。gh CLI 最直接，browser 补充发现性项目，web_search 用于中文推荐列表交叉验证。 |
| `web_verification` | ① browser 打开 URL 确认页面存在<br>② web_search 交叉验证<br>③ 标记 `link_accessible: false` | 通用验证能力。关键事实至少交叉验证 2 个独立来源。链接失效时标记，不阻塞引用但需注明。 |

---

## 三、运行时决策链

Agent 在每个数据获取步骤中的决策逻辑：

```
1. 识别 workflow 中声明的能力需求
   ↓
2. 查本表，获取该能力的优先级工具链
   ↓
3. 从优先级 ① 开始尝试：
   ├── 工具可用 → 使用该工具获取数据 → 进入下一步
   ├── 工具不可用（未安装/无权限/网络不通）→ 降级到下一优先级
   └── 全部不可用 → 能力标记 unavailable，写入 data_limitations
   ↓
4. 数据获取结果：
   ├── 满足门槛（如 20 条候选）→ 继续
   ├── 不满足门槛 → 继续降级尝试下一优先级补充
   └── 全部渠道用尽仍不满足 → 标记数据局限，不阻塞生产
```

### 降级触发条件

| 条件 | 动作 |
|------|------|
| 工具 CLI 未安装 | 跳过该优先级 |
| 工具返回空结果 | 降级到下一优先级 |
| 工具返回结果不足门槛（如仅 5 条候选） | 保留已有结果，继续尝试下一优先级补充 |
| 工具返回错误/超时 | 降级到下一优先级 |
| 全部优先级用尽 | 能力标记 `unavailable`，写入 `data_limitations` |

### 数据局限标记模板

当全部优先级用尽时，在 `manifest.json → metadata.data_limitations` 中写入：

```
"<capability> 不可用：<原因>。已降级尝试 <tried_tools>，全部失败。影响：<对产出的具体影响>。"
```

---

## 四、平台覆盖建议

为保证来源多样性，每种内容类型建议覆盖的平台类别：

| 内容类型 | 建议平台覆盖 |
|---------|------------|
| `daily-news` | 科技媒体（≥3） + 官方博客（≥1） + 社区（≥1） |
| `hotspot-analysis` | 科技媒体（≥5） + 官方博客（≥2） + 社区（≥2） |
| `topic-deep-dive` | 官方 + 学术 + 媒体 + 社区，以主题覆盖为准 |
| `github-project` | GitHub（≥5 个项目） + 社区讨论（≥2 篇评测/讨论） |
| `product-recommendation` | 官方页面（每个产品 ≥1） + 评测来源（每个产品 ≥1） |
| `social-insight` | 学术/政策 + 媒体 + 社会讨论，多角度覆盖 |
| `content-advice` | 轻量扫描，≥3 个平台即可 |

---

## 五、新增能力规则

1. 在能力表中新增一行：能力名 + 优先级工具链。
2. 在对应 workflow 中添加能力声明。
3. 能力名用 snake_case，语义自描述（如 `github_discovery` 而非 `data_source_5`）。
4. 不新增工具适配层/接口/抽象——直接在本表中维护优先级链。
