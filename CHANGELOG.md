# 更新日志（CHANGELOG）

> 说明：本仓库包含 **AI 辅助生成** 的部分内容（文档/结构/脚本等），并在发布前经过 **人工审核**。  
> 主要用于分享 PromptSemIR 的思路与参考实现模式，并非生产级开箱即用方案。

## v0.1.0 - 2025-12-29

### 新增
- PromptSemIR v0.1 规范草案：`spec/promptsemir_spec.md`
- 脱敏示例配置与说明：`config/`（含 `promptsemir_config.example.md`）
- 方案与流程文档：`docs/`（总览 / 编译流程 / Dify 工作流说明 / 配置说明 / FAQ）
- Dify 工作流导入与运行指南：`dify/README.md`
- 可复现样例：`examples/`
- MIT License

### 范围与说明
- v0.1 聚焦：**思路 + 规范 + 样例 + 工作流说明**，确保“能看懂、能跑通最小闭环”
- 不承诺：生产级部署、全 Dify/模型版本兼容、LLM 抽取细节完全一致（结构口径尽量稳定）

### 已知限制
- 暂无 JSON Schema 校验器/CLI、暂无 CI（计划 v0.2 补齐）
- 样例覆盖面有限，更多边界场景后续补充
- 工作流导出文件可能需按你的 Dify 版本做小幅调整

### 安全与隐私
- 示例与配置应为脱敏内容
- 请勿在 issue/PR 中提交密钥、内部 URL、敏感 prompt
