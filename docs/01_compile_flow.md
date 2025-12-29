# Compile Flow：PromptSemIR 编译流程（v0.1）

> 本文描述“从 raw prompt 到 PromptSemIR 制品”的编译流程口径。  
> **核心目标**：稳定、可审计、可复现、可门禁、可人审。

## 0. 输入与输出

### 输入（Input）
- `raw_prompt`：原始 prompt 文本
- `PromptSemIR_config`：抽取范围、规范化策略、门禁、人审规则等
- `compile_config_model`：驱动 compile LLM 的提示词与输出契约（JSON-only、字段结构、forbidden_keys）
- （可选）`vars`：变量表线索（可为空；只作为提醒，不可引入新事实）
- （可选）`enhance_case`：增强用例（可为空；只作为覆盖提醒，不可引入新事实）
- （可选）`human_patches`：已有补丁（若在二次编译/再打包场景使用）
- （可选）`test_case/test_config`：测试验证输入（v0.1 可不跑）

### 输出（Output）
- `PromptSemIR`（建议纯 JSON；示例可用 JSONC）
  - `meta`（含 spec/config 版本与 anchors 口径）
  - `compiler_ref` / `packager_ref`
  - `source_prompt.raw` / `source_prompt.canonical` / `hashes`
  - `compilation_context`（VARS/ENHANCE 快照；哪怕为空也必须存在）
  - `promptSem`（base，含 evidence_snippets；anchor resolve 后补全 anchors/needs_human_anchor）
  - `human_overrides`（patches + 审计）
  - `final_view`（可选物化；或由 base+patch 动态计算）
  - `runs.compile_run`（工具端记录 started_at/ended_at/status/errors）
  - `validation_report`（v0.1 可 `not_run`，但结构建议保留）
  - （可选）`runs.test_run` / `test_report`

---

## 1. 关键口径（必须全链路一致）

1) compile LLM **不输出 anchors**  
2) compile LLM **只输出**：PromptSem（语义结构）+ evidence_snippets（逐字来自 canonical prompt）  
3) anchors 由工具端 **Anchor Resolve** 生成  
4) 不确定统一标记：`needs_human_anchor = true`  
5) evidence/snippet 长度口径统一（例如建议 160 chars 上限；需要更大也要一致）  
6) 审计复现：必须快照保存 `vars/enhance_case` 到 `compilation_context`  
7) Patch 稳定性：工具端对关键数组字段进行确定性排序（例如按 name 升序）  
8) final_view 口径：`final_view = promptSem(base) + human_overrides(patches)`（单一真相）

---

## 2. 编译步骤（Step-by-step）

### Step 1) Canonicalize（工具端）
目的：生成 canonical prompt，作为 anchors 唯一基准。

建议规范化动作（示例）：
- 去除行尾空格
- 统一换行到 LF
- Unicode NFKC

同时计算哈希（建议 sha256）：
- `raw_sha256`
- `canonical_sha256`

产物：
- `source_prompt.raw`
- `source_prompt.canonical`
- `hashes.*`

---

### Step 2) Compile LLM（语义抽取）
输入：
- canonical prompt
- （可选）vars/enhance_case（仅作提示）
- compile_config_model（强制 JSON-only 输出契约）

输出（只允许）：
- `promptSem`（语义结构）
- `evidence_snippets`（逐字片段，来自 canonical prompt）
- （可选）`inferred` 层字段需带 rationale/confidence（按 config 要求）

禁止输出（示例）：
- `anchors`
- 任意 start/end/context_window/context_hash 等定位字段
- config 标记为 forbidden_keys 的字段

---

### Step 3) Sanitize & Schema Validate（工具端）
目的：把 LLM 的输出变成“可被稳定处理”的结构。

包含：
- JSON 清洗（去 markdown 代码块、修复尾逗号、容错解析等）
- 严格 schema 校验（缺字段/多字段/类型不对直接 fail 或 warning）
- 字段规范化：空数组/空对象补齐；字符串 trim；枚举值归一等
- 稳定排序：例如 variables_explicit 按 name 排序

产物：
- `promptSem_validated`
- `validation_report.schema`（pass/fail/warning）

---

### Step 4) Anchor Resolve（工具端）
目的：将 evidence_snippets 在 canonical prompt 中做确定性匹配，生成 anchors。

匹配结果：
- **唯一命中**：生成 `{ basis, start, end, snippet, context_hash }`
- **多处命中 / 无法命中**：写入 `needs_human_anchor=true`（统一标记）

建议匹配策略（按序尝试）：
1) exact
2) whitespace_normalized
3) nfkc_normalized（需与 canonicalization 一致）

产物：
- `promptSem_with_anchors`
- `validation_report.anchors`（unresolved / warnings / stats）

---

### Step 5) Gates（工具端）
目的：决定 PromptSemIR 是否可发布，或需要人工审核。

常见 gate 条件：
- required_sections 是否齐全
- schema 是否通过
- 是否存在关键字段 `needs_human_anchor=true`
- inferred 层是否满足 rationale/confidence 要求
- 是否存在 forbidden_keys
- anchors unresolved 数量是否超过阈值

产物：
- `validation_report.gates`（pass/fail + reasons）
- `runs.compile_run.status`（pass/fail）

---

### Step 6) Combine Patches → final_view（工具端）
目的：将 human_overrides 应用到 base，得到单一真相视图。

建议：
- patch 采用 JSON Patch 或等价结构
- 记录每条 patch 的来源、时间、作者（若可）
- 若不物化 final_view，也要保证可通过 base+patch 稳定计算

产物：
- `final_view`（可选）
- `human_overrides`（patches + audit）

---

### Step 7) Test（可选）
若存在 test_case 且 enabled，则执行并产出：
- `runs.test_run`
- `validation_report.test`（pass/fail/warning/not_run）
v0.1 可以 `not_run`。

---

### Step 8) Package（工具端）
目的：打包成可发布 PromptSemIR 制品。

最少应写入：
- meta / compiler_ref / packager_ref
- source_prompt（raw/canonical）与 hashes
- compilation_context（快照）
- promptSem（带 anchors/needs_human_anchor）
- runs.compile_run
- validation_report（至少占位 + not_run 可接受）

---

## 3. 常见失败与处理建议

- **LLM 输出非 JSON / 多余字段**：Sanitize 严格处理，必要时 fail fast
- **evidence_snippet 无法定位**：统一 needs_human_anchor=true，进入人审
- **多处命中**：优先引导人审提供更精确证据（更长 snippet / 更精确 anchors）
- **schema 不通过**：在 validation_report 写清原因，避免“吞错”