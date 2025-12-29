# Dify Workflows（导入与运行指南｜v0.1）

本目录下，存放了目前我测试并跑通的 dify 工作流 demo。

注意：三个工作流均使用 Google Gemini 3 Pro Preview 模型，经验证 Gemini 3 Flash Preview 模型也可使用，gpt 的未做验证。deepseek v3.2 和 r1 效果均不理想。

注意再注意：如果使用特定LLM 模型进行了相关工作流，再通过 LLM 模型对结果进行评估时，务必选择不同的模型。否则就会出现“我评我自己”的盲目自信，怎么分析都是优秀。

---

## 0. Dify 工作流DSL 文件说明

- 使用方式：在 dify 工作空间，选择导入dsl 即可
- PromptSemIR 提取.yml ：输入原始 prompt，提取 promptSemIR
- PromptSemIR 解读.yml ：输入 pormptSemIR，生成解读报告
- PromptSemIR 转译.yml ：输入 promptSemIR，生成prompt

---

## 1. 前置条件（Prerequisites）

- 一套可用的 Dify 环境（自建或云端均可）
- 至少一个可用模型（用于 compile LLM 节点）
- 你已准备好本仓库的 `examples/`（用于验证输出）

> ⚠️ 安全提醒  
> - 不要把 API Key、内部 URL、私有 prompt 等敏感信息写死到工作流或仓库。  
> - 所有密钥/连接信息应通过 Dify 的“模型提供商配置 / Secret / 环境变量”方式注入。

---

## 2. 工作流职责边界（推荐口径）

为保证可复现与稳定性，建议将职责拆分为：

### LLM 节点（语义工作）
- 只做：语义理解、结构化抽取（PromptSem）、输出 evidence_snippets（逐字证据线索）
- 强制：JSON-only 输出（避免 markdown/code fence）

### 代码/工具节点（确定性工作）
- canonicalize（规范化）
- JSON 清洗与 schema 校验
- evidence → anchors 的确定性定位（anchor resolve）
- 门禁（gates）与打包（packaging）

### 人审（可选）
- 当出现 `needs_human_anchor=true` 或 gate fail 时，人工补锚点/打 patch，再重打包或生成 final_view

---

## 3. v0.1 约定的输入 / 输出（建议保持一致）

> 变量名你可以按实际工作流调整，但建议尽量与本文一致，便于 examples 对照与复用。

### Compile（编译）工作流输入（Input）
- `raw_prompt`（必填，文本）：原始 prompt
- `vars`（可选，文本/JSON）：变量表线索或上下文（仅作提示，不可引入新事实）
- `enhance_case`（可选，文本）：增强/覆盖提醒（仅作提示）
- `compile_strict`（可选，布尔）：严格模式（更少 inferred/derived，更严格门禁）
- `human_patches_json`（可选，文本/JSON）：JSON Patch 数组（用于二次打包/人审后合并）

### Compile 输出（Output）
- `promptsemir_json`（必填，文本）：PromptSemIR 制品（JSON）
- `validation_report`（建议，文本/JSON）：schema/anchors/gates 摘要（方便排错）

---

## 4. 导入工作流（Import）

> 不同版本 Dify 的 UI 可能略有差异，但流程基本一致。

1) 打开 Dify 控制台 → Workflows（工作流）
2) 选择 Import（导入）/ Upload（上传）
3) 上传本目录下的 `compile.json`（或 `compile.yml`）
4) 导入成功后，打开工作流检查节点连线是否完整
5) 如有缺失的“自定义代码节点/插件节点”，先按你的实现补齐（建议在 README 顶部标注依赖）

---

## 5. 模型与密钥配置（Models & Secrets）

### 5.1 LLM 节点需要的模型
- compile LLM 节点需要一个可用模型（例如 GPT 系列、Claude 系列、或你自建模型网关）
- 推荐开启：
  - JSON 输出约束（若模型/平台支持）
  - 足够的 max tokens（避免 JSON 被截断）

### 5.2 密钥注入建议
- **不要**把 Key 写在工作流常量里
- 优先使用：
  - Dify 的模型提供商配置
  - Secret/环境变量（若 Dify 支持）
- 如果你的代码节点需要外部服务（不推荐 v0.1 必须），也要走 Secret

---

## 6. 用 examples 验证（最重要）

### 6.1 成功样例（必测）
1) 打开 `examples/01_basic_success/input_prompt.md`
2) 复制内容到工作流输入 `raw_prompt`
3)（可选）保持 `vars/enhance_case` 为空
4) 运行工作流
5) 将输出的 `promptsemir_json` 与：
   - `examples/01_basic_success/output_promptsemir.md`
   对照（结构一致、关键字段存在、gates 合理即可；LLM 参与时细节可能略有差异）

### 6.2 失败/人审样例（强烈建议）
若仓库包含：
- `examples/02_needs_human_anchor/`
则用同样方式运行，期望看到：
- anchors unresolved 或 `needs_human_anchor=true`
- gates 将状态指向“需人工审核/不可发布”（按你的 config 设计）

---

## 7. 推荐的节点块结构（参考）

> 下面是“推荐结构”，你可对照自己的工作流检查是否具备同等能力。

### Compile（编译）工作流推荐节点块
1) Input：收集 `raw_prompt/vars/enhance_case/strict`
2) Canonicalize（代码节点）：生成 `canonical_prompt + hashes + compilation_context`
3) Compile LLM：输出 `promptSem + evidence_snippets`（JSON-only）
4) Sanitize & Validate（代码节点）：清洗/校验/规范化/稳定排序
5) Anchor Resolve（代码节点）：evidence → anchors；失败则 needs_human_anchor
6) Gates & Package（代码节点）：门禁判定 + 生成 PromptSemIR JSON
7) Output：`promptsemir_json + validation_report`

---

## 8. 常见问题与排错（Troubleshooting）

### 8.1 输出不是 JSON / 被 markdown 包住
- 解决：在 LLM 提示中强制 JSON-only；工具节点做 sanitize（去掉 ```json 等）
- 若仍失败：记录原始输出到 run logs，便于复现

### 8.2 JSON 被截断
- 增加 max tokens
- 简化输出字段（v0.1 先少抽一点）
- 把长文本放到 `source_prompt`，不要在语义结构里重复粘贴

### 8.3 anchors 无法定位 / 多处命中
- 这是设计内的正常失败模式：应输出 `needs_human_anchor=true`
- 优化建议：
  - 提高 evidence_snippet 的唯一性（更长、更精确）
  - 统一 canonicalization 口径（否则匹配会失败）
  - 增加人审补丁（patch）流程

### 8.4 门禁失败（gates fail）
- 检查：
  - required_sections 是否齐全
  - schema 是否通过
  - 是否存在 forbidden_keys
  - needs_human_anchor 是否触发“关键字段不可发布”

---

## 9. 开源发布注意事项（Must-do）

- 所有工作流导出文件应使用占位符，不包含：
  - API Key
  - 内部 URL
  - 私有 prompt/业务数据
- 若工作流依赖自定义节点/外部服务，请在本文件顶部明确写依赖（并提供替代方案或 stub）

---

## 10. English (Short Summary)

This folder contains Dify workflow exports (compile/interpret/translate) and an import/run guide.
- Import `compile.json|yml`
- Run with `examples/01_basic_success/input_prompt.md`
- Compare output with `examples/01_basic_success/output_promptsemir.md`
- Keep secrets out of workflows; inject via Dify provider config / secrets.
- LLM does semantics + evidence snippets; deterministic anchors & gates are handled by tool/code nodes.