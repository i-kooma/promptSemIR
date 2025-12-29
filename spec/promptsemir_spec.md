

# PromptSemIR Spec（v0.1）

> **AI 辅助编码 + 人工审核**：本规范文档可能包含 AI 辅助生成内容，并在发布前经人工审核。  
> 本 spec 用于分享 PromptSemIR 的结构化思路（参考实现），并不承诺与任何现有标准完全兼容。

## 1. 目标与非目标

### 目标（Goals）
PromptSemIR 旨在把 prompt 工程“制品化”，提供：
- **结构化**：便于编排、检索、比对、治理
- **可审计**：可回溯 raw/canonical 输入与编译上下文
- **可复现**：确定性步骤可重复执行得到一致结果
- **可门禁**：明确是否可发布/是否需人审
- **可迁移**：可从 IR 渲染到不同 prompt 形态/平台

### 非目标（Non-goals）
- 不追求“完全自动无人工”：遇到不确定场景要显式触发人审
- 不强行绑定某个 LLM / 某个 Dify 版本：以结构与口径为主

## 2. 规范性语言（Normative Keywords）

- **MUST**：必须满足
- **SHOULD**：推荐满足
- **MAY**：可选

---

## 3. 顶层结构（Top-level Object）

PromptSemIR 是一个 JSON 对象（建议 UTF-8），顶层字段如下：

### 3.1 必须字段（MUST）
- `meta`
- `source_prompt`
- `hashes`
- `compilation_context`
- `promptSem`
- `runs`
- `validation_report`

### 3.2 可选字段（MAY）
- `human_overrides`
- `final_view`
- `tests` / `test_report`（如果你有测试阶段）

---

## 4. 字段定义

### 4.1 meta（MUST）
记录 spec/config/workflow 版本与基本信息。

推荐字段：
- `spec_version`（string）MUST
- `config_version`（string）MUST
- `workflow_version`（string）SHOULD
- `created_at`（string, ISO8601）SHOULD
- `compiler_ref`（object/string）MAY：编译器标识（例如 workflow 名称、commit hash 等）
- `packager_ref`（object/string）MAY：打包器标识

### 4.2 source_prompt（MUST）
保存原始输入与规范化输入，用于审计与 anchors 匹配基准。

- `raw`（string）MUST
- `canonical`（string）MUST
- `canonicalization`（object）SHOULD：记录 canonicalization 策略（与 config 对齐）

### 4.3 hashes（MUST）
用于一致性校验与去重。

- `raw_sha256`（string）MUST
- `canonical_sha256`（string）MUST

> hash 算法可替换，但 MUST 在 meta 或字段名体现。

### 4.4 compilation_context（MUST）
用于复现与解释当次编译输入上下文（哪怕为空也要保留结构）。

推荐结构：
- `vars_snapshot`（object/string/null）MUST（允许为空）
- `enhance_case_snapshot`（object/string/null）MUST（允许为空）
- `notes`（string）MAY

### 4.5 promptSem（MUST）
语义结构（base）。v0.1 推荐分层：`core`（事实/显式）与 `inferred`（推断/增强，可选）。

**v0.1 关键约束：**
- LLM 输出 **MUST NOT** 直接写 deterministic anchors（start/end 等）
- LLM 输出 **SHOULD** 为关键字段提供 `evidence_snippets`
- 工具端 anchor resolve 之后，`promptSem` MAY 包含 `anchors` 或 `needs_human_anchor`

推荐结构（示意）：
- `core`（object）MUST：核心语义（意图、约束、输出格式等）
- `inferred`（object）MAY：推断层（如启用必须带 rationale/confidence）
- `evidence`（array/object）SHOULD：证据线索集合（逐字片段）
- `anchors`（array/object）MAY：工具端生成的定位信息
- `needs_human_anchor`（boolean or list）MAY：标记无法确定定位的字段/项

> 允许你按项目需要扩展 `core` 子结构，但建议受 config 白名单约束。

### 4.6 human_overrides（MAY）
人工审核的补丁与审计信息。

建议包含：
- `patches`（array）SHOULD：推荐 JSON Patch（RFC 6902）或等价结构
- `review_notes`（string）MAY
- `audit`（object）MAY：who/when/why（注意脱敏）

### 4.7 final_view（MAY）
单一真相视图。  
**SHOULD** 保证：`final_view = apply(patches, promptSem)`（或等价规则）。  
v0.1 可不物化，只要可稳定计算即可。

### 4.8 runs（MUST）
运行记录（用于复现与排错）。至少包含 compile。

- `compile_run`（object）MUST
  - `started_at`（ISO8601）SHOULD
  - `ended_at`（ISO8601）SHOULD
  - `status`（string: pass/fail/needs_review）MUST
  - `errors`（array）MAY
  - `warnings`（array）MAY

（可选）
- `interpret_run`（object）MAY
- `translate_run`（object）MAY
- `test_run`（object）MAY

### 4.9 validation_report（MUST）
校验与门禁摘要，用于判断是否可发布/是否需人审。

建议结构：
- `schema`：pass/fail/warn + details
- `anchors`：resolved/unresolved stats + unresolved paths
- `gates`：pass/fail/needs_review + reasons
- `notes`：补充说明

---

## 5. v0.1 行为约束（关键）

### 5.1 LLM 输出约束（Compile LLM MUST）
- MUST 输出 JSON（JSON-only）
- MUST NOT 输出 anchors/deterministic 定位字段
- SHOULD 为关键字段提供 evidence_snippets（逐字来自 canonical）

### 5.2 工具端约束（Tooling MUST）
- MUST 提供 canonicalization（确定性、可复现）
- MUST 对 LLM 输出进行 sanitize + schema validate
- MUST 执行 evidence → anchors 的确定性定位（成功则写 anchors；失败则 needs_human_anchor）
- MUST 执行 gates，并写入 validation_report

---

## 6. 最小示例（Skeleton）

下面是一个“结构骨架示例”（字段可按你的 PromptSem 细化）：

```json
{
  "meta": {
    "spec_version": "0.1.0",
    "config_version": "0.1.0",
    "workflow_version": "0.1.0",
    "created_at": "2025-01-01T00:00:00Z"
  },
  "source_prompt": {
    "raw": "…",
    "canonical": "…",
    "canonicalization": { "normalize_newlines_to_lf": true }
  },
  "hashes": {
    "raw_sha256": "…",
    "canonical_sha256": "…"
  },
  "compilation_context": {
    "vars_snapshot": null,
    "enhance_case_snapshot": null,
    "notes": ""
  },
  "promptSem": {
    "core": { "intent": "…", "constraints": [], "output": { "format": "…" } },
    "evidence": [
      { "path": "core.intent", "snippet": "…", "match_mode": "exact" }
    ],
    "anchors": [],
    "needs_human_anchor": false
  },
  "human_overrides": {
    "patches": [],
    "review_notes": ""
  },
  "runs": {
    "compile_run": { "status": "pass", "errors": [], "warnings": [] }
  },
  "validation_report": {
    "schema": { "status": "pass", "details": [] },
    "anchors": { "unresolved_count": 0, "unresolved_paths": [] },
    "gates": { "status": "pass", "reasons": [] }
  }
}