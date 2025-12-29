---
# promptSemIR

> 本仓库主要用于分享 **PromptSemIR 的思路 / 方法论** 与 Dify 工作流实践，并非“开箱即用的生产级方案”。


## 这是什么？
目前大多数Prompt管理都将Prompt视为一段文本。
实际上，如果我们输入一个很烂的prompt，要求llm替我们优化/改写这个prompt时，llm需要先提取语义、设定目标输出格式然后才能输出优化后的prompt。
本项目的核心想法是，将prompt的语义抽取出来并保存，形成一个Prompt的语义中间件（我姑且称之为PromptSemIR）。

## 有什么用？
将Prompt的语义提取为PromptSemIR后，我们可以实现
- 现存prompt的语义资产固化；
- prompt结构及内容优化；
- 通过prompt语义，将prompt在dspy或其他各类prompt管理平台中迁移；

本仓库包含：
- PromptSemIR 的 **规范（Spec）与示例（Examples）**
- **Dify 工作流**（编译 / 解读 / 转译）
- 抽取规则、证据、锚点、门禁、人审点等 **配置与约定**

> ⚠️ **声明 / 免责**  
> - 本项目包含AI生成的代码。  
> - 本仓库以 **分享思路与参考实现** 为主。  
> - 如用于生产环境，请自行做充分验证与安全审查。

---

## 快速开始（5–10 分钟跑通）

### 0）准备

1. dify环境（可以申请官网个人版）
2. LLM API Key：推荐Gemini 3 pro

### 1）导入 Dify 工作流

请先阅读：[`workflows/dify/README.md`](./dify/README.md)

### 2）跑通示例
1. 在PromptSemIR编译的Dify工作流中，输入任意一个Prompt，运行得到PromptSemIR文件内容
2. 在PromptSemIR解读的Dify工作流中，输入PromptSemIR文件内容，运行得到PromptSemIR文件解读报告
3. 在 PromptSemIR 转译的 Dify 工作流中，输入 PromptSemIR 文件内容+需要适配的 LLM 名称，运行得到针对目标 LLM 优化的 Prompt

---

## 仓库结构

```text
promptSemIR/
├─ docs/            # 设计说明与解释文档
├─ spec/            # PromptSemIR 规范
├─ config/          # 示例配置
├─ dify/  					# Dify 工作流导出文件 + 导入指南
├─ examples/        # 案例
