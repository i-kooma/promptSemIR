<!-- config/promptsemir_config.example.md -->

# PromptSemIR Config Example（v0.1, sanitized）

> **AI 辅助编码 + 人工审核**：本示例配置用于分享思路与口径约束。  
> 所有模型、endpoint、密钥均为占位符；请在你自己的环境中通过 Secret/环境变量注入。

下面给出一个**示例配置**（YAML 风格，便于阅读）。  
你可以按需改成 JSON，只要语义一致即可。

```yaml
meta:
  config_name: "promptsemir_config"
  config_version: "0.1.0"
  spec_version: "0.1.0"
  workflow_version: "0.1.0"
  notes: "Sanitized example for open-source sharing"

# 1) Canonicalization：用于生成 canonical_prompt（anchors 唯一基准）
canonicalization:
  normalize_newlines_to_lf: true
  trim_trailing_spaces: true
  unicode_nfkc: true
  collapse_multiple_blank_lines: false

# 2) Extraction：限制 LLM 可输出的结构与字段范围
extraction:
  mode: "json_only"              # 强制 JSON-only 输出契约（推荐）
  allow_unknown_fields: false    # 不允许输出未声明字段
  forbidden_keys:               # LLM 禁止输出（确定性字段必须工具端生成）
    - "anchors"
    - "start"
    - "end"
    - "context_window"
    - "context_hash"
    - "offset"
    - "line"
    - "column"

  # 允许的 section（你可以按你的 PromptSem 结构调整）
  allowed_sections:
    - "core"
    - "variables"
    - "constraints"
    - "tools"
    - "output"
    - "safety"
    - "metadata"
    - "evidence"

  # 必须存在的字段（缺失时 gate fail 或进入人审）
  required_fields:
    - "core.intent"
    - "output.format"

  # 可选：对一些字段限制最大长度/数量，降低被截断风险
  limits:
    max_list_items: 50
    max_string_length: 5000

# 3) Evidence policy：证据口径（v0.1 推荐：核心字段必须给 evidence_snippet）
evidence_policy:
  enabled: true
  snippet_max_chars: 160
  snippet_min_chars: 20
  must_come_from_canonical: true

  # 哪些字段必须提供 evidence（按你的 PromptSem 路径命名）
  required_evidence_paths:
    - "core.intent"
    - "constraints.must"
    - "output.format"

  # 匹配口径：exact 为主，必要时允许 normalized
  match_modes:
    - "exact"
    - "whitespace_normalized"
    - "nfkc_normalized"

# 4) Anchors policy：anchors 由工具端生成
anchors_policy:
  enabled: true
  resolver: "deterministic_match"
  on_no_match: "needs_human_anchor"
  on_multi_match: "needs_human_anchor"

  # unresolved 阈值（超过阈值则 gate fail / require human review）
  unresolved_threshold:
    max_unresolved_count: 0         # v0.1 推荐对关键字段为 0
    max_unresolved_ratio: 0.0

# 5) Inferred policy：是否允许“推断层”输出（建议默认更严格）
inferred_policy:
  enabled: false
  require_rationale: true
  require_confidence: true
  confidence_range: [0.0, 1.0]
  inferred_must_not_modify_core: true

# 6) Gates：门禁规则（决定是否可发布或进入人审）
gates:
  schema_validation:
    enabled: true
    fail_on_error: true
    warn_on_unknown: true

  forbidden_keys_check:
    enabled: true
    fail_on_detected: true

  required_fields_check:
    enabled: true
    fail_on_missing: true

  anchors_check:
    enabled: true
    fail_on_needs_human_anchor: true

  # 必须存在的顶层段（PromptSemIR 打包层面）
  required_sections:
    - "meta"
    - "source_prompt"
    - "hashes"
    - "compilation_context"
    - "promptSem"
    - "runs"
    - "validation_report"

# 7) Packaging：打包 PromptSemIR 制品的要求
packaging:
  include_final_view: false          # v0.1 可选（可用 base+patch 动态计算）
  include_compilation_context: true  # 必须快照 vars/enhance_case（哪怕为空）
  include_run_logs: true
  include_validation_report: true

  # 建议：把运行状态/时间/错误写到 runs.compile_run
  run_recording:
    record_started_at: true
    record_ended_at: true
    record_status: true
    record_errors: true