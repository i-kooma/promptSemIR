# Dify Workflows：工作流说明（v0.1）

> 本文描述仓库中 Dify 工作流的定位、输入输出约定，以及如何与 examples 对照跑通。  
> 工作流导出文件与导入步骤见：`workflows/dify/README.md`

## 1. 工作流清单

### 1.1 Compile（编译）
目标：把 raw prompt 编译为 PromptSemIR（最小闭环核心）。

典型输入变量（建议）
- `raw_prompt`（必填，Paragraph）
- `vars`（可选，Paragraph/Object；变量表线索）
- `enhance_case`（可选，Paragraph）
- `compile_strict`（可选，Checkbox；严格模式可限制 inferred/derived 输出）
- `human_patches_json`（可选，Paragraph；JSON Patch 数组字符串）

典型输出
- `promptsemir_json`（编译产物 JSON）
- `validation_report`（门禁/锚点/校验摘要）
- （可选）`interpret_markdown` / `translated_prompt`（若你把后续工作流串起来）

---

### 1.2 Interpret（解读，可选）
目标：把 PromptSemIR “解释成人能读的报告”，用于评审与沟通。

输入
- `promptsemir_json`（必填）

输出
- `interpret_markdown`（或等价文本）

---

### 1.3 Translate（转译，可选）
目标：把 PromptSemIR 渲染为目标 Prompt（可加入目标模型偏好）。

输入
- `promptsemir_json`（必填）
- `target_llm_preference`（可选，Paragraph；目标模型偏好/风格）
- `strict_mode`（可选，Checkbox；开启时只参考 core 层，不用 inferred/enhance）

输出
- `translated_prompt`（纯文本）

---

## 2. 工作流设计建议（职责边界）

为提高稳定性，建议按以下边界设计节点：

- **LLM 节点**：只负责语义理解与结构化抽取（PromptSem + evidence）
- **代码/工具节点**：负责 canonicalize / schema 校验 / anchor resolve / gate / packaging
- **人审节点（可选）**：用于处理 needs_human_anchor 与高风险场景的 patch 生成

> v0.1 不建议把“确定性定位字段”交给 LLM 输出（如 start/end），应下沉到工具节点。

---

## 3. 推荐的节点分层（参考）

### Compile 工作流（建议节点块）
1) User Input：raw_prompt / vars / enhance_case / strict 等
2) Canonicalize（代码节点）：生成 canonical_prompt + hashes + compilation_context 快照
3) Compile LLM：输出 PromptSem + evidence_snippets（JSON-only）
4) Sanitize & Validate（代码节点）：清洗/校验/规范化/稳定排序
5) Anchor Resolve（代码节点）：evidence → anchors；无法定位则 needs_human_anchor
6) Gates & Package（代码节点）：门禁判定 + 生成 PromptSemIR JSON
7) 输出：promptsemir_json + validation_report

Interpret / Translate 工作流可以独立运行，也可以与 Compile 串联。

---

## 4. 如何用 examples 验证（强烈建议）

1) 导入 compile 工作流  
2) 用 `examples/01_basic_success/input_prompt.md` 的内容作为 raw_prompt  
3) 运行后得到的 `promptsemir_json` 应在结构上接近 `examples/01_basic_success/output_promptsemir.md`  
4) 如有 `02_needs_human_anchor`，可验证门禁与人审触发是否符合预期

---

## 5. 安全实践

- 不要把 API Key 写死在工作流里（使用环境变量/连接器）
- 示例与截图均应脱敏
- issue/PR 中不要提交真实业务 prompt 与密钥