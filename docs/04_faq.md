# FAQ / Troubleshooting（v0.1）

## Q0：这套方案的意义是什么？

当我在搭建 agent 时，发现每次更换 llm 模型，都需要重新调试 prompt，所以我想不同的模型对 pormpt 应该有不同的侧重，或者需要强调不同的约束。

于是，我开始尝试用 gemini 等模型，输入prompt，要求他替我优化成目标模型的 prompt，这个过程往往是炼丹一样的，不可控。

在于是，我想着把这一整个环节拆解开来，输入原始 prompt -> llm 提取语义 -> 确定目标模型偏好 -> 定向优化 prompt。

所以这套方案的核心意义，是将 prompt 的语义提取出来，并且基于语义实现 pormpt 管理、迁移、调优。

---

## Q1：为什么不让 LLM 直接输出 anchors（start/end）？
因为定位字段属于确定性工作，交给 LLM 会带来：
- 不可复现（同一输入可能输出不同索引）
- 漂移（模型升级后索引口径变化）
- 失败点增多（格式更复杂更易错）
v0.1 建议：LLM 只输出 evidence_snippets，工具端再做确定性 anchors。

---

## Q2：evidence_snippet 在 canonical prompt 里找不到怎么办？
输出 `needs_human_anchor=true`，并在 validation_report 写明原因与建议动作：
- 是否 snippet 被截断过短
- 是否 canonicalization 规则不一致
- 是否 prompt 文本发生了二次改写

---

## Q3：evidence_snippet 多处命中怎么办？
同样标记 `needs_human_anchor=true`。建议人审提供：
- 更长、更唯一的 snippet
- 或直接给出更明确的 anchor（由工具端写入 patches）

---

## Q4：compile LLM 输出不是严格 JSON 怎么办？
在 Sanitize 步骤做容错清洗（去 code fence、修复常见格式问题）。  
若仍无法解析，建议 fail fast，并把原始输出写入 runs.compile_run.errors 便于排查。

---

## Q5：strict 模式应该怎么用？
建议 strict 模式用于“只提取 core/显式信息”的场景：
- 禁止 inferred/derived（或强限制）
- 门禁更严格（needs_human_anchor 更容易触发人审）
适合你在面试/演示时展示“稳健可控”。

---

## Q6：如何新增一个字段/section？
推荐流程：
1) 先在 spec 里定义语义与类型
2) 在 config 中加入白名单/必填/证据要求
3) 更新 compile_config_model（让 LLM 知道输出结构）
4) 更新 examples（至少 1 个覆盖样例）
5) 更新 gates（是否要求 evidence/anchors）

---

## Q7：human_overrides / patches 推荐用什么格式？
推荐 JSON Patch（RFC 6902）或等价结构，关键是：
- 可审计（记录谁、何时、为什么改）
- 可重放（base + patches 可稳定复现 final_view）
- 可 diff（更利于 review）

---

## Q8：为什么要保存 compilation_context（vars/enhance 快照）？
为了审计与复现：
- 同一个 raw prompt，在不同 vars/enhance 下抽取结果可能不同
- 没有快照就无法解释“为什么这次编译这样输出”

---

## Q9：我跑出来的 PromptSemIR 和 examples 不一致正常吗？
可能正常，尤其当 LLM 参与抽取时。建议对照的重点是：
- 顶层结构是否一致（required_sections 是否齐）
- anchors/needs_human_anchor 的行为是否符合预期
- gates/validation_report 的状态是否合理

---

## 常见报错速查

- **schema fail**：检查 forbidden_keys、字段类型、必填字段
- **anchors unresolved 太多**：提高 snippet 唯一性 / 增加人审 patch
- **workflow 导入失败**：检查 Dify 版本与导出格式（json/yml）
- **输出为空/被截断**：检查模型 max tokens 与 JSON-only 约束