# Tech-Magazine 设计规范（通用基线）

> 路径：templates/formats/tech-magazine/design.md
> 职责：定义"科技杂志"格式的视觉设计规范。当前为通用基线版本，用户提供参考文章链接后迭代固化。
> Phase：1
> 依赖：无（格式层独立于内容层）
> 被依赖：workflows/format-consume.md
> 最后更新：2026-06-17

---

## 一、设计思想

**定位：** 面向微信公众号的科技长文排版。干净、专业、有呼吸感。不花哨，不"营销号风"。读者读完感觉"这是一篇认真写的科技文章"，而非"一篇用模板快速生成的推送"。

**关键词：** 克制的排版 · 舒适的阅读节奏 · 信息层级清晰 · 暗色模式友好

---

## 二、配色方案

### 基础色板

| 用途 | 色值 | 说明 |
|------|------|------|
| 正文文字 | `#1a1a1a` | 深灰而非纯黑，减少视觉疲劳 |
| 次级文字 | `#666666` | 说明文字、日期、来源标注 |
| 链接色 | `#0066cc` | 可辨识的蓝色 |
| 强调色（主色） | `#1a56db` | 标题装饰线、卡片边框、标签 |
| 强调色（辅色） | `#047857` | 数据、增长类信息 |
| 页面背景 | `#ffffff` | 微信默认白色背景 |
| 卡片背景 | `#f8f9fa` | 浅灰卡片，稍带区分 |
| 分隔线 | `#e5e7eb` | 浅灰分隔线 |
| 引用块背景 | `#f0f4ff` | 浅蓝引用块背景 |
| 引用块左边框 | `#1a56db` | 与主色一致 |

### 暗色模式（CSS 变量适配）

```css
:root {
  --text-primary: #1a1a1a;
  --text-secondary: #666666;
  --link: #0066cc;
  --accent: #1a56db;
  --accent-green: #047857;
  --bg-page: #ffffff;
  --bg-card: #f8f9fa;
  --border: #e5e7eb;
  --quote-bg: #f0f4ff;
  --quote-border: #1a56db;
}

@media (prefers-color-scheme: dark) {
  :root {
    --text-primary: #e5e7eb;
    --text-secondary: #9ca3af;
    --link: #60a5fa;
    --accent: #3b82f6;
    --accent-green: #34d399;
    --bg-page: #111827;
    --bg-card: #1f2937;
    --border: #374151;
    --quote-bg: #1e3a5f;
    --quote-border: #3b82f6;
  }
}
```

---

## 三、字体与字号

### 字体栈

```css
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC",
               "Hiragino Sans GB", "Microsoft YaHei", "Helvetica Neue",
               Helvetica, Arial, sans-serif;
}
```

优先级：系统字体（macOS/iOS）→ 苹方 → 冬青黑体 → 微软雅黑 → 回退。

### 字号层级

| 层级 | 字号 | 行高 | 字重 | 用途 |
|------|------|------|------|------|
| H1 | 22px | 1.4 | 700 | 文章大标题 |
| H2 | 18px | 1.5 | 700 | 章节标题（如"一、本期概览"） |
| H3 | 16px | 1.5 | 600 | 小节标题（如热点条目标题） |
| H4 | 15px | 1.6 | 600 | 分组标题 |
| 正文 | 15px | 1.8 | 400 | 正文段落 |
| 小字 | 13px | 1.6 | 400 | 说明文字、来源标注、脚注 |
| 代码 | 14px | 1.6 | 400 | 行内代码和代码块 |

### 微信特殊考虑

- 微信最小可读正文字号：14px，推荐 15px。
- 微信不支持 `rem` 单位（部分 Android），使用 `px`。
- 微信不支持 web fonts，仅能使用系统字体。

---

## 四、间距系统

| 元素 | 间距 | 说明 |
|------|------|------|
| 段间距（正文段落之间） | 12px | `margin-bottom: 0.8em` |
| 标题上方间距 | 32px | `margin-top: 2em` |
| 标题下方间距 | 12px | `margin-bottom: 0.8em` |
| 列表项间距 | 6px | `margin-bottom: 0.4em` |
| 引用块上下间距 | 16px | `margin: 1em 0` |
| 卡片上下间距 | 20px | `margin: 1.2em 0` |
| 分隔线上下间距 | 24px | `margin: 1.5em 0` |
| 图片上下间距 | 20px | `margin: 1.2em 0` |
| 左右内边距 | 0 | 微信文章区已有 15px 边距 |

---

## 五、组件规范

### 5.1 章节标题

```
┌─────────────────────────────────────────┐
│  一、本期概览                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  （标题下方 2px 装饰线，主色，30% 宽度）      │
└─────────────────────────────────────────┘
```

```css
h2 {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
  margin-top: 2em;
  margin-bottom: 0.8em;
  padding-bottom: 0.4em;
  border-bottom: 2px solid var(--accent);
}
```

### 5.2 引用块（编辑点评/信号块）

```
┌──────────────────────────────────────────┐
│ ┃ 💡 编辑视角：这是一条重要的趋势信号……     │
│ ┃                                         │
└──────────────────────────────────────────┘
```

```css
blockquote.signal {
  background: var(--quote-bg);
  border-left: 4px solid var(--quote-border);
  padding: 12px 16px;
  margin: 1em 0;
  font-size: 14px;
  color: var(--text-secondary);
  border-radius: 0 4px 4px 0;
}

blockquote.signal strong {
  color: var(--text-primary);
}
```

### 5.3 项目卡片（GitHub 项目推荐专用）

```
┌──────────────────────────────────────────┐
│  llama.cpp              ⭐ 78k  Python   │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │
│  通用本地大模型推理框架，支持 GGUF 格式，    │
│  在消费级硬件上高效运行。                   │
│                                           │
│  核心特性：                                │
│  • CPU/GPU 混合推理                       │
│  • 支持 50+ 模型架构                       │
│  • 内存占用极低                            │
│                                           │
│  🔗 github.com/ggerganov/llama.cpp        │
│  💬 编辑点评：本地推理的事实标准……           │
└──────────────────────────────────────────┘
```

```css
.project-card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 16px 20px;
  margin: 1.2em 0;
}

.project-card h4 {
  font-size: 16px;
  font-weight: 600;
  margin: 0 0 8px 0;
  color: var(--text-primary);
}

.project-card .meta {
  font-size: 13px;
  color: var(--text-secondary);
  margin-bottom: 12px;
}

.project-card .meta span {
  margin-right: 16px;
}

.project-card .features {
  margin: 8px 0;
  padding-left: 16px;
}

.project-card .features li {
  font-size: 14px;
  line-height: 1.6;
  color: var(--text-primary);
}

.project-card .comment {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid var(--border);
  font-size: 14px;
  color: var(--text-secondary);
}
```

### 5.4 对比表格

```css
.comparison-table {
  width: 100%;
  border-collapse: collapse;
  margin: 1.2em 0;
  font-size: 14px;
  overflow-x: auto;
  display: block;
}

.comparison-table th {
  background: var(--bg-card);
  color: var(--text-primary);
  font-weight: 600;
  padding: 10px 12px;
  text-align: left;
  border-bottom: 2px solid var(--border);
  white-space: nowrap;
}

.comparison-table td {
  padding: 8px 12px;
  border-bottom: 1px solid var(--border);
  color: var(--text-primary);
}

.comparison-table tr:hover td {
  background: var(--bg-card);
}
```

### 5.5 标签

```css
.tag {
  display: inline-block;
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 4px;
  background: rgba(26, 86, 219, 0.1);
  color: var(--accent);
  margin-right: 6px;
}
```

### 5.6 分隔线

```css
hr {
  border: none;
  border-top: 1px solid var(--border);
  margin: 1.5em 0;
}
```

---

## 六、图片规范

| 规则 | 说明 |
|------|------|
| 宽度 | 100%（微信文章区约 645px） |
| 格式 | PNG（截图）/ JPG（照片）/ SVG（图表） |
| 边距 | `margin: 1.2em 0` |
| 圆角 | `border-radius: 6px`（截图类图片） |
| 说明文字 | 图片下方灰色 13px 居中 |
| 微信图床 | 最终发布时需上传到微信素材库并替换 URL。模板阶段使用占位符 `[图片：xxx]` |

---

## 七、覆盖层（Overrides）

渲染时样式分三层：

1. **基线**：本文件（design.md）定义的通用样式。
2. **覆盖层**：`overrides.md`（如存在），为特定期次/特定客户定制的样式调整。格式与 design.md 相同，但只包含需要覆盖的变量和规则。
3. **渲染**：Agent 在生成 HTML 时合并基线 + 覆盖层。

**覆盖层命名：** `overrides-<type>-<date>.md`（如 `overrides-hotspot-analysis-2026-06-17.md`），放在对应 outputs 目录中。不影响其他期次的渲染。

---

## 八、待固化项

以下项等待用户提供参考文章链接后精调：

- [ ] 最终确认配色（主色饱和度、辅色选择）
- [ ] 标题装饰线样式（渐变/纯色/虚线？）
- [ ] 卡片圆角值（4px / 8px / 12px？）
- [ ] 朋友圈分享卡的设计
- [ ] 首图/引导图设计规范
- [ ] 文末"关于我们"/引导关注区块
