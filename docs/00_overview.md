# Overview：PromptSemIR 方案总览（v0.1）

> **AI 辅助编码 + 人工审核**：本仓库包含 AI 辅助生成的文档/脚本/结构化内容，并在发布前经人工审核。  
> **定位**：主要分享 PromptSemIR 的思路与工作流落地方式（参考实现），不承诺生产环境开箱即用。

## 1. PromptSemIR 是什么

**PromptSemIR** 是一种面向 Prompt 的“语义中间表示（IR）”发布制品，用于把“不可控的自然语言 Prompt”转化为：
- **结构化**：可被机器读取与治理
- **可审计**：能回溯到原始输入
- **可追溯**：语义结论尽量有证据线索（evidence）与定位（anchors）
- **可迁移**：可从 IR 渲染/转译到不同提示词模板或不同 LLM 偏好
- **可工作流化**：可在 Dify 等编排平台上稳定跑通

本仓库提供：
- PromptSemIR **规范（Spec）与示例（Examples）**
- PromptSemIR **配置（Config）**：抽取范围、证据策略、锚点策略、门禁、人审触发等
- **Dify 工作流**（compile / interpret / translate）导入说明与导出文件
- 最小闭环：raw prompt → 编译 → PromptSemIR →（可选）解读/转译

---

## 2. 核心术语（最小集合）

- **raw prompt**：用户输入原文，用于审计/回溯。
- **canonical prompt**：工具端对 raw prompt 做确定性规范化后的文本（作为 anchors 唯一基准）。
- **PromptSem**：语义抽取草稿（工作产物），可迭代、可人审、可打补丁。
- **PromptSemIR**：可发布/可治理的 IR（制品），包含上下文快照、运行记录、门禁与人审信息。
- **evidence_snippet（证据线索）**：LLM 输出的“可定位线索”，通常是 canonical prompt 中的逐字片段。
- **anchors（锚点定位）**：工具端将 evidence_snippet 在 canonical prompt 中做确定性匹配后得到的定位信息（start/end 等）。
- **needs_human_anchor**：统一标记“不确定、无法唯一定位”的字段，需要人工审核/补充锚点。
- **patch / human_overrides**：人工审核后的补丁（建议用 JSON Patch 或等价结构），与 base 组合得到最终视图。
- **final_view**：单一真相（可物化或动态计算）= base（PromptSem）+ patches（human_overrides）。

---

## 3. 设计原则（v0.1 强约束）

### 3.1 LLM 只做“语义 + 结构化 + 证据线索”
为了降低失败概率与漂移：
- compile LLM **不输出 anchors**（不输出 start/end/hash/context_window 等确定性字段）
- compile LLM **只输出 PromptSem（语义结构）+ evidence_snippets（逐字片段）**
- anchors 由工具端在 **Anchor Resolve** 步骤确定性生成

### 3.2 确定性工作下沉到工具端
包括但不限于：
- canonicalization（规范化）
- JSON 清洗 / schema 校验
- evidence → anchors 的匹配定位
- gates（门禁）判定
- 数组字段的稳定排序（保证可重复）

### 3.3 显式的人审闭环
当出现：
- 证据不足 / 多处命中 / 无法命中
- 关键字段缺失或 schema 不通过
- release gate 不通过  
必须显式输出状态，触发人工审核与 patch。

---

## 4. 最小闭环（MVP 流程）

1) 输入 raw prompt  
2) 工具端 canonicalize → 得到 canonical prompt（anchors 基准）  
3) compile LLM 抽取 PromptSem + evidence_snippets  
4) 工具端 schema 校验 & 清洗  
5) 工具端 anchor resolve：evidence_snippets → anchors（唯一命中则落位；否则 needs_human_anchor）  
6) gates：按 config 判定是否可发布（或进入人审）  
7) 打包 PromptSemIR：写入 runs、validation_report、compilation_context（快照）等  
8)（可选）interpret / translate

---

## 5. 本仓库与 v0.1 交付范围

建议 v0.1 聚焦：
- spec：字段语义与版本策略（能读懂）
- workflows：能导入并跑出示例（能跑通）
- examples：至少 1 个成功样例（能对照）
- config：示例配置（可改、可扩展）
- docs：解释“为什么这么做、怎么做”

非 v0.1 必需（但 v0.2 推荐）：
- schema validator CLI / CI
- 更完整的可视化 diff / editor
- 更大规模的测试集与指标体系

---

## 6. 安全与隐私

- 本仓库应只包含**脱敏示例**，请勿提交密钥、内部 URL、客户信息、内部业务数据。
- issue/PR 中也不要粘贴敏感 prompt 或凭证。