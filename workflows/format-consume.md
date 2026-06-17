# 格式消费 — 工作流（L3 总入口）

> 路径：workflows/format-consume.md
> 职责：定义 L3 格式消费的完整流程——从 content-package 到 HTML 成品。是所有内容类型共用的格式消费入口。
> Phase：1
> 依赖：CONSTITUTION.md, schemas/content-package.md, templates/formats/tech-magazine/design.md, templates/formats/tech-magazine/REFERENCE.md, templates/publishing-advice.md, references/confirmation/format-confirmation.md
> 最后更新：2026-06-17

---

## 资源加载清单

Agent 激活本 workflow 时加载：

| 阶段 | 加载资源 |
|------|---------|
| 启动 | 对应的格式模板目录：`templates/formats/<format>/design.md` → `REFERENCE.md` |
| 确认 ③ | `references/confirmation/format-confirmation.md` |
| L4 | `templates/publishing-advice.md` |

---

## 前置条件

- L2 content-package 已通过内容确认（确认 ②）
- `outputs/<type>-<date>/manifest.json` 和 `body.md` 已就绪
- 格式模板（design.md）已存在

---

## 工作流步骤

### 步骤 1：读取 confirmed content-package

```
输入：outputs/<type>-<date>/manifest.json
      outputs/<type>-<date>/body.md

读取 manifest.json → 获取：
  - metadata.title / subtitle / date / keywords
  - content_type（决定使用哪种格式模板套件）
  - body_markdown

读取 body.md → 作为渲染的直接内容输入
```

### 步骤 2：匹配格式模板

```
当前 Phase 1 仅支持一种格式：
  content_type 任意 → tech-magazine

未来扩展方式：
  - 新增格式 = 在 templates/formats/ 下新增目录 + SKILL.md 路由表加一行
  - 用户指定格式 → 路由到对应目录
  - 未指定 → 默认 tech-magazine

加载：templates/formats/tech-magazine/design.md
      templates/formats/tech-magazine/REFERENCE.md（参考，不直接参与渲染）
```

### 步骤 3：渲染 output-review.html

```
样式合并（三层）：
  ① 基线：templates/formats/tech-magazine/design.md 定义的 CSS 变量和组件规范
  ② 覆盖层：outputs/<type>-<date>/overrides.md（如存在）— 仅覆盖需要调整的变量/规则
  ③ 渲染：Agent 将 body.md 的 Markdown 结构 → 对应的 HTML 组件

渲染规则：
  # <标题>        → <h1>
  ## <章节>       → <h2>
  ### <小节>      → <h3>
  #### <子小节>   → <h4>
  - 列表          → <ul>/<ol>
  > 引用/信号块    → <blockquote>（含 CSS class .signal 体现信号类型）
  [^ref-xxx]     → <a href="#ref-xxx">[^ref-xxx]</a>（文内）
                    <p id="ref-xxx">（文末脚注区）
  ![](assets/x)  → <img src="assets/x">（或占位符 [图片：xxx]）
  **粗体**       → <strong>
  *斜体*         → <em>
  [text](url)    → <a href="url">text</a>

特殊组件识别（Agent 根据 Markdown 上下文判断）：
  - 表格（有 | 分隔的行）→ <table class="comparison-table">
  - 项目卡片（#### 后紧跟 | 属性 | 值 | 的表）→ <div class="project-card">
  - 概览卡片（## 一、本期概览 后续的关键词行）→ <div class="overview-card">
  - 信号块（> 💡 / 📈 / 👀 / ⚠️ 开头）→ <blockquote class="signal-block">
  - 标签行（🏷️ 开头的行）→ <span class="tag">

生成文件：
  outputs/<type>-<date>/output-review.html
  （带完整内联 style，直接可在浏览器中打开审阅）
```

### 步骤 4：呈现 output-review.html 供审阅

```
Agent 在浏览器中打开 output-review.html。
邀请用户审阅排版效果。

如用户不在（如 cron 任务），呈现文字摘要：
  ✅ HTML 已生成：outputs/<type>-<date>/output-review.html
  📐 格式：tech-magazine
  📏 字数：约 X,XXX 字
  📱 目标平台：微信公众号
```

---

## 确认 ③：格式确认（L3→L4）

```
加载：references/confirmation/format-confirmation.md

呈现：
1. 排版样式检查摘要
2. 组件选择检查摘要（如有卡片/表格/信号块）
3. 输出规格 — 目标平台兼容性（微信公众号）

回流：用户要求调整 → 修改 design.md 覆盖参数或调整 body.md Markdown 结构 → 重新渲染
通过：用户确认"确认" → 进入 L4
```

---

## L4：发布建议

```
加载：templates/publishing-advice.md

基于内容类型 + 格式 + 时效性 → 生成发布建议卡

输出：
  建议发布日期 + 时间
  4 个标题变体（A/B/C/D）
  封面方向
  发布平台优先级
  传播建议（转发语 + 社群角度）
  注意事项
```

---

## 步骤 5：固化为 output.html

```
用户确认格式 + 发布建议后：
  - 将 output-review.html 复制为 output.html（最终版）
  - output-review.html 保留作为审阅记录

如果 output-review.html 和 output.html 相同：
  - 直接重命名或保留两个文件均可
```

---

## 最终交付清单

```
outputs/<type>-<date>/
├── manifest.json         ✅ 内容资产元数据
├── body.md               ✅ Markdown 成稿
├── editorial-notes.md    ❓ 编辑手记（如有长文版）
├── output-review.html    ✅ 审阅版 HTML
├── output.html           ✅ 最终版 HTML
├── overrides.md          ❓ 覆盖层（如使用了定制样式）
└── assets/               ❓ 图片资源
    └── ...
```

---

## 格式扩展规则

### 新增格式步骤

1. 复制 `templates/formats/_TEMPLATE/` → `templates/formats/<new-format>/`
2. 填入 `<new-format>/design.md` — 设计规范
3. 填入 `<new-format>/REFERENCE.md` — 参考来源
4. 填入 `<new-format>/example.html` — 样例
5. 在 SKILL.md 中补充新格式的名称映射
6. 完成。**不动任何已有文件。**
