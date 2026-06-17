# 分析框架知识库挂载点

> 路径：references/frameworks/_README.md
> 职责：分析框架知识库的挂载点说明。框架是编辑阶段的辅助工具，按需加载，不替代编辑判断。
> Phase：1
> 依赖：CONSTITUTION.md（第五节：知识库扩展规则）
> 最后更新：2026-06-17

---

## 使用方式

1. **新增框架** = 在此目录下新增一个 `.md` 文件。
2. **workflow 中加载** = 在对应 workflow 中添加一行加载声明：`加载 references/frameworks/<框架名>.md`。
3. **可选、不强制**：框架是辅助工具，不替代编辑判断。数据不足时宁可标"数据局限"也不强行套框架。
4. **不纳入 content-package schema**：框架是编辑阶段的工具，不是内容资产的字段。

---

## 待扩展框架（示例，按需添加）

| 框架 | 文件名 | 用途 | 适用内容类型 | 状态 |
|------|-------|------|------------|------|
| 波特五力模型 | `porter-five-forces.md` | 行业竞争格局分析 | topic-deep-dive | 待添加 |
| PEST 分析 | `pest-analysis.md` | 宏观环境分析 | social-insight | 待添加 |
| 技术成熟度曲线 | `hype-cycle.md` | 判断技术所处阶段 | topic-deep-dive, hotspot-analysis | 待添加 |
| SWOT 分析 | `swot-analysis.md` | 公司/产品竞争分析 | topic-deep-dive | 待添加 |
| 麦肯锡 MECE | `mece.md` | 结构化分解复杂议题 | topic-deep-dive | 待添加 |

---

## 命名规范

- 文件名：小写英文 + 连字符，如 `porter-five-forces.md`
- 文件内标题：中文名 + 英文名，如 `# 波特五力模型（Porter's Five Forces）`
- 元数据头部按 CONSTITUTION.md 第七节规范
