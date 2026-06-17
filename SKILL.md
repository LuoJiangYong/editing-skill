---
name: editing-skill
description: "When the user asks to edit AI/tech news, write daily/weekly/monthly reports, analyze topics, recommend GitHub projects or products, generate social insights, or create WeChat HTML articles — use this skill for structured news editing and format consumption."
version: 0.2.0
---

# Editing Skill — 路由中枢

> AI 科技新闻编辑：内容采编 → 内容资产包 → 格式消费 → 发布建议。

## 路由决策树

```
用户需求
  │
  ├─ "今天AI有什么新闻" / 日报 / 速览
  │    → workflows/daily-news.md
  │
  ├─ "本周/本月热点" / 趋势复盘 / 排行
  │    → workflows/hotspot-analysis.md
  │
  ├─ "分析XX公司/技术/行业" / 深度专题
  │    → workflows/topic-deep-dive.md
  │
  ├─ "推荐AI工具/产品" / 好用的工具
  │    → workflows/product-recommendation.md
  │
  ├─ "推荐GitHub项目" / 开源/AI库
  │    → workflows/github-project.md
  │
  ├─ "AI对社会的影响" / 职业/教育/监管
  │    → workflows/social-insight.md
  │
  └─ "有什么选题" / 标题方向 / 传播角度
       → workflows/content-advice.md

格式消费（独立入口）:
  "生成微信公众号HTML" / 格式排版 / 科技杂志风
    → workflows/format-consume.md
```

## 管线激活（4 层 Pipeline + 3 层确认）

```
L1 采编 → L2 资产包 → L3 格式消费 → L4 发布建议
 ①确认      ②确认        ③确认
```

### 按需加载清单

| 阶段 | 加载资源 |
|------|---------|
| **所有类型** | `CONSTITUTION.md` → `references/data-sources.md` → `references/source-and-verification.md` |
| **按内容类型** | `references/content-types-guide.md` + `templates/content-structure/<type>.md` |
| **格式消费** | `templates/formats/<format>/design.md` + `REFERENCE.md` |
| **确认关卡** | `references/confirmation/information-confirmation.md` / `editorial-confirmation.md` / `format-confirmation.md` |
| **分析框架** | `references/frameworks/` 下按需加载（可选，不强制） |

### 两层内容模型

- `content_items[]` — 素材层（id, type, title, summary, url, source, status, verification）
- `body_markdown` — 成稿层（纯 Markdown，格式无关）
- **不做** `content_blocks` 中间层

### 3 层确认不可跳过

① **信息确认**（L1→L2）：选题范围、候选素材、来源覆盖 → 可回流 L1
② **内容确认**（L2→L3）：正文结构、事实准确性、引用完整性 → 可回流 L1/L2
③ **格式确认**（L3→L4）：排版样式、组件选择、输出规格 → 可回流 L3

## 禁止事项

- ❌ 不硬编码 TrendRadar 工具名，走 data-sources.md 降级链
- ❌ 不杜撰新闻/链接/数据，所有内容须有可追溯来源
- ❌ 不跳过确认关卡（除非用户明确说"跳过"）
- ❌ 不把 HTML 当源文件——修改内容必须回写 body.md + manifest.json 再重渲染
- ❌ 不把格式信息（颜色/字号/组件）写入 content-package
- ❌ content-package 不存评分，评分在各 workflow 层
- ❌ <3 个消费者不抽象——不做 provider 契约/adapter/中间映射层
- ❌ 不自动推送/发送到外部平台，需用户明确确认

## 下游对接

| 产出 | 下游 Skill / 工具 |
|------|------------------|
| HTML 成品 | 直接复制到微信公众号后台 |
| 小红书 Brief | → `infographic-carousel` Skill 生成信息图 |
| 音频脚本 | → `songwriting-and-ai-music` / TTS 生成播客 |
| 视频脚本 | → `hyperframes` / `muse-video` Skill |
| 发布建议卡 | 用户自行用于排期和分发决策 |

## 产出验证

完成内容生产后自检：
1. **来源可追溯**：每条入选素材有 url + source_platform，三项验证字段（source_verified / link_accessible / source_tier）已填写
2. **结构完整**：manifest.json + body.md + editorial_notes 齐全，body.md 纯 Markdown 无 HTML
3. **确认通过**：三层确认关卡均经用户确认（或用户明确说跳过）
4. **格式解耦**：content-package 不含任何格式信息，换格式不需要改内容
5. **持久化**：产出已写入 `outputs/<type>-<date>/` 目录
