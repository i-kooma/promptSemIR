# Config：PromptSemIR_config 说明（v0.1）

> Config 的作用：把“抽取范围、证据口径、锚点口径、门禁与人审策略”显式化，避免口径漂移。  

## 1. 设计目标

v0.1 的核心取向：
- 降低 LLM 失败概率：LLM 只做“语义理解 + 结构化抽取 + evidence（可定位线索）”
- 将确定性工作下沉：canonicalize / JSON 清洗校验 / schema 校验 / evidence→anchors 定位 / gates
- 避免双版本漂移：final_view 可由 base + patches 稳定复现

---

## 2. 配置结构（推荐分区）

下面是“建议你在 config 中固定具备”的分区（字段名可按你的实现调整，但语义要一致）：

### 2.1 meta
- config_name / config_version
- target_semir_spec（期望产出的 PromptSemIR spec 版本）
- canonicalization 策略（见下）

### 2.2 canonicalization（规范化）
用于将 raw_prompt 转成 canonical_prompt（anchors 唯一基准），建议包含：
- normalize_newlines_to_lf
- trim_trailing_spaces
- unicode_nfkc
- （可选）collapse_multiple_blank_lines 等

> 原则：canonicalization 必须是 **确定性** 的、可复现的。

### 2.3 extraction scope（抽取范围）
用于约束 LLM 输出的 PromptSem：
- 允许抽取的 section / field 白名单（避免“瞎扩展”）
- required_fields（缺失时 fail 或 warning）
- 禁止字段 forbidden_keys（尤其禁止 anchors、定位字段等）

### 2.4 evidence policy（证据口径）
建议明确：
- evidence_snippets 必须来自 canonical_prompt 的逐字片段（或明确允许的弱匹配口径）
- snippet 最大长度（例如 160 chars）
- 核心字段必须有证据（可配置：哪些字段必须 evidence）

### 2.5 anchors policy（锚点策略）
建议明确：
- anchors 由工具端生成（LLM 禁止输出）
- 匹配策略顺序（exact → whitespace_normalized → nfkc_normalized）
- 多处命中/无法命中时的统一标记：needs_human_anchor=true
- unresolved 阈值与 gate 规则（见下）

### 2.6 inferred/derived policy（推断层要求）
如果允许 inferred（推断/增强）层输出，建议加硬约束：
- require_rationale（必须给推断理由）
- require_confidence（必须给置信度范围）
- inferred 不得污染 core（事实层）

### 2.7 gates（门禁）
用于决定是否可发布 PromptSemIR：
- required_sections（必须存在的顶层段）
- schema_status 必须 pass（或允许 warning）
- needs_human_anchor 在关键字段出现则 fail（或进入“待人审”状态）
- unresolved 数量/比例阈值
- forbidden_keys 检测到则 fail

---

## 3. 修改指南（最常改的 5 个点）

1) **抽取白名单**：你希望 LLM 抽哪些字段  
2) **必填字段**：哪些字段缺了就不发布  
3) **证据策略**：哪些字段必须 evidence_snippet  
4) **锚点匹配策略**：是否允许 whitespace_normalized；snippet 长度上限  
5) **门禁阈值**：needs_human_anchor 是否直接 fail；unresolved 上限

---

## 4. 版本策略建议

- PromptSemIR spec 版本：描述制品结构与语义
- Config 版本：描述抽取与门禁策略
- Workflow 版本：描述运行链路与节点实现

建议在 meta 中显式记录：
- `spec_version`
- `config_version`
- `workflow_version`（或 compiler_ref/packager_ref）

并在 `CHANGELOG.md` 中说明 breaking change。

