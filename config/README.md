# Config（PromptSemIR_config）使用说明（v0.1）

> **AI 辅助编码 + 人工审核**：本仓库包含 AI 辅助生成的文档/结构化内容，并在发布前经人工审核。  
> 本目录主要分享 PromptSemIR 的配置思路与口径约束（参考实现），并非生产环境开箱即用方案。

## 1. Config 的定位

`PromptSemIR_config` 用于把“抽取范围、证据策略、锚点定位策略、门禁与人审规则”显式化，避免：
- 同一套 prompt 在不同环境/不同人维护下口径漂移
- LLM 输出不受控（字段乱增、定位乱写、缺证据）
- 无法审计（不知道当时用的规则/参数/上下文）

**v0.1 核心约束：**
- LLM **只输出**：`promptSem`（语义结构）+ `evidence_snippets`（逐字证据线索）
- LLM **禁止输出** anchors（start/end 等定位字段）与任何 deterministic 字段
- anchors 由工具端 **Anchor Resolve** 从 canonical prompt 中确定性匹配得到
- `needs_human_anchor=true` 统一表示需要人工补锚点/补证据/打 patch
- gates（门禁）决定是否可发布 PromptSemIR 或进入人审

---

## 2. 文件说明

- `promptsemir_config.example.md`
  - 一个**脱敏**的示例配置（可直接复制修改）
  - 推荐把你实际使用的版本另存为：`promptsemir_config.local.md`（不提交仓库）

---

## 3. 你最常需要改的 7 个配置点（建议从这里入手）

1) **canonicalization**
- 规范化规则必须确定性，且与 anchors 匹配口径一致

2) **extraction.allowed_sections / allowed_fields**
- 限制 LLM 能输出的 section/字段范围，避免“编译时乱长字段”

3) **extraction.required_fields**
- 缺了哪些字段就不允许发布（或必须进入人审）

4) **evidence_policy**
- 哪些字段必须提供 evidence_snippet
- snippet 长度上限、允许的匹配模式（exact/normalized）

5) **anchors_policy**
- 匹配策略顺序
- 多处命中/无法命中时的统一处理（needs_human_anchor）
- unresolved 阈值

6) **gates**
- schema 是否必须 pass
- needs_human_anchor 是否直接 fail（建议对关键字段 fail）
- forbidden_keys 检测（建议直接 fail）

7) **packaging**
- PromptSemIR 产物里必须保留哪些运行记录与上下文快照（审计/复现）

---

## 4. 推荐的版本策略

建议在 config 中显式记录：
- `spec_version`：PromptSemIR 制品结构的版本
- `config_version`：抽取与门禁口径版本
- `workflow_version`：Dify 工作流/编译链路版本（或 compiler_ref）

并在仓库 `CHANGELOG.md` 里写明 breaking change