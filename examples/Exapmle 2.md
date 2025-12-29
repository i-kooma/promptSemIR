# Example 2

## 总结

````markdown
场景：会议纪要结构化助手，从 meeting_notes 抽取 summary/decisions/action_items/risks/open_questions，并输出 JSON。

问题点
原始 prompt 里有重复句：“不得添加未提及的信息。”写了两次。
这导致验证器在 anchors 阶段报：evidence_snippet matched multiple locations (2)，也就是证据片段在 canonical 文本里能匹配到多处，锚点无法唯一定位。
在 PromptSemIR 资产里也明确标记了 needs_human_anchor: true（constraints 和 output_spec 都出现）。

转译 prompt 的效果
转译版把约束升级成“编号规则 + 解释”，包括 null 处理、日期格式、task 可执行句式、以及“不要输出代码块/解释”。这对下游模型执行其实更友好。

这个样例的价值：能诚实地暴露不稳定点（锚点歧义），并且给出 needs_human_anchor,要求人工修订。
````



## 原始prompt

````markdown
你是「会议纪要结构化助手」，请从会议记录中提取摘要、决策和行动项，帮助团队快速推进。

【输入】
- 会议主题：{{meeting_title}}
- 参会人：{{attendees}}
- 会议记录（逐字/要点混杂）：{{meeting_notes}}

【目标输出】
输出为 JSON（仅 JSON），用于后续进入工作流自动生成任务单。

【你需要提取】
1) summary：会议摘要（3～5 条要点）
2) decisions：已达成的决策（若无则空数组）
3) action_items：行动项列表（每项包含 owner、due_date、task）
4) risks：风险与阻塞点（若无则空数组）
5) open_questions：尚未解决的问题（若无则空数组）

【硬性约束】
- 不得添加未提及的信息。
- 不得添加未提及的信息。
- 若会议记录中没有明确 owner 或 due_date，请填 null，不要猜测。
- 行动项 task 必须是可执行句式（动词开头），长度不超过 30 字。

【字段规范】
- action_items 是数组，每个元素结构必须是：
  { "owner": "姓名或null", "due_date": "YYYY-MM-DD或null", "task": "..." }
- 所有日期必须统一为 YYYY-MM-DD；无法判断则为 null。
- summary 每条不超过 25 字。

【再次强调】
输出为 JSON（仅 JSON）。不要输出 markdown，不要输出解释，不要输出代码块。
如果你无法确定某个信息，请使用 null，并把对应问题写入 open_questions。

【输出 JSON 结构】
{
  "summary": ["..."],
  "decisions": ["..."],
  "action_items": [{"owner": null, "due_date": null, "task": "..."}],
  "risks": ["..."],
  "open_questions": ["..."]
}

````

## PromptSemIR内容

````json
{
  "meta": {
    "spec_name": "PromptSemIR",
    "spec_version": "1.0.0",
    "semir_id": "semir-108399f3-7245-410b-ba70-916e37e3b27e",
    "created_at": "2025-12-29T16:09:01Z",
    "lifecycle_status": "validated",
    "anchor_indexing": {
      "basis": "canonical",
      "unit": "char"
    },
    "tags": [
      "mvp",
      "prompt-asset"
    ]
  },
  "compiler_ref": {
    "promptsemir_config_ref": {
      "name": "PromptSemIR_config",
      "version": "1.1.1",
      "config_hash": null
    },
    "compile_config_model_ref": {
      "name": "compile_config_model",
      "version": "2.1.0",
      "config_hash": null
    }
  },
  "packager_ref": {
    "name": "PromptSemIR_packager",
    "version": "1.0.2",
    "config_hash": "sha256:ad7c94c2d5b44d0a8b43324b1e3dfb24959bec34a31f5333b594139282c62561"
  },
  "source_prompt": {
    "raw": "你是「会议纪要结构化助手」，请从会议记录中提取摘要、决策和行动项，帮助团队快速推进。\n\n【输入】\n- 会议主题：{{meeting_title}}\n- 参会人：{{attendees}}\n- 会议记录（逐字/要点混杂）：{{meeting_notes}}\n\n【目标输出】\n输出为 JSON（仅 JSON），用于后续进入工作流自动生成任务单。\n\n【你需要提取】\n1) summary：会议摘要（3～5 条要点）\n2) decisions：已达成的决策（若无则空数组）\n3) action_items：行动项列表（每项包含 owner、due_date、task）\n4) risks：风险与阻塞点（若无则空数组）\n5) open_questions：尚未解决的问题（若无则空数组）\n\n【硬性约束】\n- 不得添加未提及的信息。\n- 不得添加未提及的信息。\n- 若会议记录中没有明确 owner 或 due_date，请填 null，不要猜测。\n- 行动项 task 必须是可执行句式（动词开头），长度不超过 30 字。\n\n【字段规范】\n- action_items 是数组，每个元素结构必须是：\n  { \"owner\": \"姓名或null\", \"due_date\": \"YYYY-MM-DD或null\", \"task\": \"...\" }\n- 所有日期必须统一为 YYYY-MM-DD；无法判断则为 null。\n- summary 每条不超过 25 字。\n\n【再次强调】\n输出为 JSON（仅 JSON）。不要输出 markdown，不要输出解释，不要输出代码块。\n如果你无法确定某个信息，请使用 null，并把对应问题写入 open_questions。\n\n【输出 JSON 结构】\n{\n  \"summary\": [\"...\"],\n  \"decisions\": [\"...\"],\n  \"action_items\": [{\"owner\": null, \"due_date\": null, \"task\": \"...\"}],\n  \"risks\": [\"...\"],\n  \"open_questions\": [\"...\"]\n}\n",
    "canonical": "你是「会议纪要结构化助手」,请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。\n\n【输入】\n- 会议主题:{{meeting_title}}\n- 参会人:{{attendees}}\n- 会议记录(逐字/要点混杂):{{meeting_notes}}\n\n【目标输出】\n输出为 JSON(仅 JSON),用于后续进入工作流自动生成任务单。\n\n【你需要提取】\n1) summary:会议摘要(3~5 条要点)\n2) decisions:已达成的决策(若无则空数组)\n3) action_items:行动项列表(每项包含 owner、due_date、task)\n4) risks:风险与阻塞点(若无则空数组)\n5) open_questions:尚未解决的问题(若无则空数组)\n\n【硬性约束】\n- 不得添加未提及的信息。\n- 不得添加未提及的信息。\n- 若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。\n- 行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。\n\n【字段规范】\n- action_items 是数组,每个元素结构必须是:\n  { \"owner\": \"姓名或null\", \"due_date\": \"YYYY-MM-DD或null\", \"task\": \"...\" }\n- 所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。\n- summary 每条不超过 25 字。\n\n【再次强调】\n输出为 JSON(仅 JSON)。不要输出 markdown,不要输出解释,不要输出代码块。\n如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。\n\n【输出 JSON 结构】\n{\n  \"summary\": [\"...\"],\n  \"decisions\": [\"...\"],\n  \"action_items\": [{\"owner\": null, \"due_date\": null, \"task\": \"...\"}],\n  \"risks\": [\"...\"],\n  \"open_questions\": [\"...\"]\n}\n",
    "hashes": {
      "raw_sha256": "sha256:39e2956a8586218758b8d4a08581684ae592f0d10b4d73116cfcd0d5e1aff02f",
      "canonical_sha256": "sha256:9aadbc9a4f5c94f7258c794cbb156b67122932c876d4161fb4665f8ec9edf728"
    },
    "canonicalization": {
      "normalization": [
        "trim_trailing_spaces",
        "normalize_newlines_to_lf",
        "unicode_nfkc"
      ]
    }
  },
  "compilation_context": {
    "postprocess_applied": [
      "deterministic_sort:variables_by_name",
      "packager:assemble_semir"
    ]
  },
  "origin_model_adaptation": {
    "provider": "google",
    "model": "gemini-3-flash-preview"
  },
  "runs": {
    "compile_run": {
      "run_id": "run-e6ac5c6f-38df-4a8c-8246-534205cec82d",
      "started_at": "2025-12-29T16:08:09Z",
      "ended_at": "2025-12-29T16:09:01Z",
      "status": "success",
      "errors": [],
      "llm": {
        "provider": "llm-node"
      },
      "security_applied": {
        "redact_secrets_before_llm": false
      }
    }
  },
  "promptSem": {
    "core_semantics": {
      "intent": {
        "value": "从会议记录中提取摘要、决策和行动项,帮助团队快速推进",
        "origin": "explicit",
        "evidence_snippets": [
          "请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 14,
            "end": 42,
            "snippet": "请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。",
            "context_hash": "sha256:996123c030bb79e4a3f30cdc3e7394dad68d65a95023dc7d641ffac7f77d64b5",
            "context_window": {
              "before": "你是「会议纪要结构化助手」,",
              "after": "\n\n【输入】\n- 会议主题:{{meeting_title}}\n- 参会人:{{attendees}}\n- 会议记录(逐字/要点混杂):{{meeting_no"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "role": {
        "value": "会议纪要结构化助手",
        "origin": "explicit",
        "evidence_snippets": [
          "你是「会议纪要结构化助手」"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 0,
            "end": 13,
            "snippet": "你是「会议纪要结构化助手」",
            "context_hash": "sha256:1181680e8cb9e300e72fd50a343c39e075974f478b5e39442993fbd26dfe692a",
            "context_window": {
              "before": "",
              "after": ",请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。\n\n【输入】\n- 会议主题:{{meeting_title}}\n- 参会人:{{attendees}}"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "variables_explicit": {
        "value": [
          {
            "name": "attendees",
            "description": "参会人"
          },
          {
            "name": "meeting_notes",
            "description": "会议记录(逐字/要点混杂)"
          },
          {
            "name": "meeting_title",
            "description": "会议主题"
          }
        ],
        "origin": "explicit",
        "evidence_snippets": [
          "- 会议主题:{{meeting_title}}",
          "- 参会人:{{attendees}}",
          "- 会议记录(逐字/要点混杂):{{meeting_notes}}"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 49,
            "end": 73,
            "snippet": "- 会议主题:{{meeting_title}}",
            "context_hash": "sha256:2f37304e175610e5458c10910b34822b3162bb9463f36a4beeaec23ea03c495f",
            "context_window": {
              "before": "你是「会议纪要结构化助手」,请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。\n\n【输入】\n",
              "after": "\n- 参会人:{{attendees}}\n- 会议记录(逐字/要点混杂):{{meeting_notes}}\n\n【目标输出】\n输出为 JSON(仅 JSON),"
            }
          },
          {
            "basis": "canonical",
            "start": 74,
            "end": 93,
            "snippet": "- 参会人:{{attendees}}",
            "context_hash": "sha256:d24cea18ed629127d9d6423f6647e2c5f47fd8bc408bac76dbc1b9f77306d0be",
            "context_window": {
              "before": "你是「会议纪要结构化助手」,请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。\n\n【输入】\n- 会议主题:{{meeting_title}}\n",
              "after": "\n- 会议记录(逐字/要点混杂):{{meeting_notes}}\n\n【目标输出】\n输出为 JSON(仅 JSON),用于后续进入工作流自动生成任务单。\n\n【"
            }
          },
          {
            "basis": "canonical",
            "start": 94,
            "end": 127,
            "snippet": "- 会议记录(逐字/要点混杂):{{meeting_notes}}",
            "context_hash": "sha256:e456854149410afde01aa10d5e8946cf400cf15b38ccfefab2b73f431515058c",
            "context_window": {
              "before": "请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。\n\n【输入】\n- 会议主题:{{meeting_title}}\n- 参会人:{{attendees}}\n",
              "after": "\n\n【目标输出】\n输出为 JSON(仅 JSON),用于后续进入工作流自动生成任务单。\n\n【你需要提取】\n1) summary:会议摘要(3~5 条要点)\n2)"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "constraints": {
        "value": [
          "输出为 JSON(仅 JSON)",
          "不得添加未提及的信息。",
          "若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。",
          "行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。",
          "所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。",
          "summary 每条不超过 25 字。",
          "不要输出 markdown,不要输出解释,不要输出代码块。",
          "如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。"
        ],
        "origin": "explicit",
        "evidence_snippets": [
          "输出为 JSON(仅 JSON)",
          "不得添加未提及的信息。",
          "若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。",
          "行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。",
          "所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 377,
            "end": 418,
            "snippet": "若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。",
            "context_hash": "sha256:d3c5179facd304e05dd7508515f1223aba4e876d09faf8fbcd9fddfc4dcaab1b",
            "context_window": {
              "before": "若无则空数组)\n5) open_questions:尚未解决的问题(若无则空数组)\n\n【硬性约束】\n- 不得添加未提及的信息。\n- 不得添加未提及的信息。\n- ",
              "after": "\n- 行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。\n\n【字段规范】\n- action_items 是数组,每个元素结构必须是:\n  {"
            }
          },
          {
            "basis": "canonical",
            "start": 421,
            "end": 456,
            "snippet": "行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。",
            "context_hash": "sha256:bf7eb98a06d97ecedaf316f32025409b549b847b07f4bff9fb46f9e646182a4d",
            "context_window": {
              "before": "硬性约束】\n- 不得添加未提及的信息。\n- 不得添加未提及的信息。\n- 若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。\n- ",
              "after": "\n\n【字段规范】\n- action_items 是数组,每个元素结构必须是:\n  { \"owner\": \"姓名或null\", \"due_date\": \"YYYY"
            }
          },
          {
            "basis": "canonical",
            "start": 568,
            "end": 601,
            "snippet": "所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。",
            "context_hash": "sha256:d3528ff8f6dd012d8a5034b28bd2a6bc7d780f50e585395e9c5f2e175b5864d6",
            "context_window": {
              "before": "结构必须是:\n  { \"owner\": \"姓名或null\", \"due_date\": \"YYYY-MM-DD或null\", \"task\": \"...\" }\n- ",
              "after": "\n- summary 每条不超过 25 字。\n\n【再次强调】\n输出为 JSON(仅 JSON)。不要输出 markdown,不要输出解释,不要输出代码块。\n如果"
            }
          }
        ],
        "needs_human_anchor": true
      },
      "output_spec": {
        "value": {
          "format": "json",
          "schema_raw": "{\n  \"summary\": [\"...\"],\n  \"decisions\": [\"...\"],\n  \"action_items\": [{\"owner\": null, \"due_date\": null, \"task\": \"...\"}],\n  \"risks\": [\"...\"],\n  \"open_questions\": [\"...\"]\n}",
          "fixed_fields": true,
          "json_only": true,
          "forbid_code_fence": true
        },
        "origin": "explicit",
        "evidence_snippets": [
          "【输出 JSON 结构】",
          "输出为 JSON(仅 JSON)",
          "不要输出代码块"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 726,
            "end": 738,
            "snippet": "【输出 JSON 结构】",
            "context_hash": "sha256:a4739dd5ca97e88367e06defbd8f3f7f2f842d53bdab702d1491cfcd9498f758",
            "context_window": {
              "before": "N)。不要输出 markdown,不要输出解释,不要输出代码块。\n如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。\n\n",
              "after": "\n{\n  \"summary\": [\"...\"],\n  \"decisions\": [\"...\"],\n  \"action_items\": [{\"owner\": nu"
            }
          },
          {
            "basis": "canonical",
            "start": 670,
            "end": 677,
            "snippet": "不要输出代码块",
            "context_hash": "sha256:819172bfa60c7dc5a407e8ebe188ea8d1350ed3bc5f3bf6724adeeecdfb5a83b",
            "context_window": {
              "before": "法判断则为 null。\n- summary 每条不超过 25 字。\n\n【再次强调】\n输出为 JSON(仅 JSON)。不要输出 markdown,不要输出解释,",
              "after": "。\n如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。\n\n【输出 JSON 结构】\n{\n  \"summary\": [\"."
            }
          }
        ],
        "needs_human_anchor": true
      }
    },
    "derived_semantics": {},
    "augmentation_plan": {}
  },
  "human_overrides": {
    "patches": [],
    "audit": null
  },
  "final_view": {
    "core_semantics": {
      "intent": "从会议记录中提取摘要、决策和行动项,帮助团队快速推进",
      "role": "会议纪要结构化助手",
      "variables_explicit": [
        {
          "name": "attendees",
          "description": "参会人"
        },
        {
          "name": "meeting_notes",
          "description": "会议记录(逐字/要点混杂)"
        },
        {
          "name": "meeting_title",
          "description": "会议主题"
        }
      ],
      "constraints": [
        "输出为 JSON(仅 JSON)",
        "不得添加未提及的信息。",
        "若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。",
        "行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。",
        "所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。",
        "summary 每条不超过 25 字。",
        "不要输出 markdown,不要输出解释,不要输出代码块。",
        "如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。"
      ],
      "output_spec": {
        "format": "json",
        "schema_raw": "{\n  \"summary\": [\"...\"],\n  \"decisions\": [\"...\"],\n  \"action_items\": [{\"owner\": null, \"due_date\": null, \"task\": \"...\"}],\n  \"risks\": [\"...\"],\n  \"open_questions\": [\"...\"]\n}",
        "fixed_fields": true,
        "json_only": true,
        "forbid_code_fence": true
      }
    },
    "derived_semantics": {},
    "augmentation_plan": {}
  },
  "validation_report": {
    "status": "warning",
    "compile": {
      "status": "pass",
      "errors": [],
      "warnings": []
    },
    "anchors": {
      "status": "warning",
      "unresolved_core_fields": [
        "/promptSem/core_semantics/constraints",
        "/promptSem/core_semantics/output_spec"
      ],
      "warnings": [
        {
          "path": "/promptSem/core_semantics/constraints",
          "message": "evidence_snippet matched multiple locations (2)"
        },
        {
          "path": "/promptSem/core_semantics/constraints",
          "message": "evidence_snippet matched multiple locations (2)"
        },
        {
          "path": "/promptSem/core_semantics/output_spec",
          "message": "evidence_snippet matched multiple locations (2)"
        }
      ]
    },
    "test": {
      "status": "not_run",
      "summary": "MVP"
    }
  }
}
````

## PromptSemIR报告

````markdown
# PromptSemIR 资产解读报告
**资产ID**: `semir-108399f3-7245-410b-ba70-916e37e3b27e`
**生命周期状态**: **VALIDATED**
**创建时间**: 2025-12-29T16:09:01Z
---
## 1. 原始提示词 (Source Prompt)
> **原始文本全量展示**：
```text
你是「会议纪要结构化助手」，请从会议记录中提取摘要、决策和行动项，帮助团队快速推进。

【输入】
- 会议主题：{{meeting_title}}
- 参会人：{{attendees}}
- 会议记录（逐字/要点混杂）：{{meeting_notes}}

【目标输出】
输出为 JSON（仅 JSON），用于后续进入工作流自动生成任务单。

【你需要提取】
1) summary：会议摘要（3～5 条要点）
2) decisions：已达成的决策（若无则空数组）
3) action_items：行动项列表（每项包含 owner、due_date、task）
4) risks：风险与阻塞点（若无则空数组）
5) open_questions：尚未解决的问题（若无则空数组）

【硬性约束】
- 不得添加未提及的信息。
- 不得添加未提及的信息。
- 若会议记录中没有明确 owner 或 due_date，请填 null，不要猜测。
- 行动项 task 必须是可执行句式（动词开头），长度不超过 30 字。

【字段规范】
- action_items 是数组，每个元素结构必须是：
  { "owner": "姓名或null", "due_date": "YYYY-MM-DD或null", "task": "..." }
- 所有日期必须统一为 YYYY-MM-DD；无法判断则为 null。
- summary 每条不超过 25 字。

【再次强调】
输出为 JSON（仅 JSON）。不要输出 markdown，不要输出解释，不要输出代码块。
如果你无法确定某个信息，请使用 null，并把对应问题写入 open_questions。

【输出 JSON 结构】
{
  "summary": ["..."],
  "decisions": ["..."],
  "action_items": [{"owner": null, "due_date": null, "task": "..."}],
  "risks": ["..."],
  "open_questions": ["..."]
}

```
> **Canonical Hash**: `sha256:9aadbc9a4f5c94f7258c794cbb156b67122932c876d4161fb4665f8ec9edf728`
---
## 2. 编译上下文 (Compilation Context)
- **变量快照 (Vars Snapshot)**: 无
- **增强用例快照 (Enhance Case)**: 无
---
## 3. 核心语义层 (Core Semantics)
> **💡 数据可信度图例**：
> * **显式确信 (explicit)**：原文直接写明的文字。
> * **隐式确信 (entailed)**：原文未直接写明，但逻辑上必然包含的事实。
> * **推断 (inferred)**：LLM基于概率挖掘的信息（附置信度）。
> * **规划建议 (projected)**：LLM基于专家知识对未来的优化建议（非原文内容）。

#### 角色 (role)
> **来源**：显式确信 (explicit)
> **内容**：会议纪要结构化助手
> **证据**：“你是「会议纪要结构化助手」”

#### 意图 (intent)
> **来源**：显式确信 (explicit)
> **内容**：从会议记录中提取摘要、决策和行动项,帮助团队快速推进
> **证据**：“请从会议记录中提取摘要、决策和行动项,帮助团队快速推进。”

#### 显式变量 (variables_explicit)
> **来源**：显式确信 (explicit)
> **内容**：
1. **attendees**：参会人
2. **meeting_notes**：会议记录(逐字/要点混杂)
3. **meeting_title**：会议主题
> **证据**：“- 会议主题:{{meeting_title}}”; “- 参会人:{{attendees}}”; “- 会议记录(逐字/要点混杂):{{meeting_notes}}”

#### 约束条件 (constraints)
> **来源**：显式确信 (explicit)
> **内容**：
- 输出为 JSON(仅 JSON)
- 不得添加未提及的信息。
- 若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。
- 行动项 task 必须是可执行句式(动词开头),长度不超过 30 字。
- 所有日期必须统一为 YYYY-MM-DD;无法判断则为 null。
- summary 每条不超过 25 字。
- 不要输出 markdown,不要输出解释,不要输出代码块。
- 如果你无法确定某个信息,请使用 null,并把对应问题写入 open_questions。
> **证据**：“输出为 JSON(仅 JSON)”; “不得添加未提及的信息。”; “若会议记录中没有明确 owner 或 due_date,请填 null,不要猜测。”; “行动项 task 必须是可执行句式(动...

#### 输出规范 (output_spec)
> **来源**：显式确信 (explicit)
> **内容**：{'format': 'json', 'schema_raw': '{\n  "summary": ["..."],\n  "decisions": ["..."],\n  "action_items": [{"owner": null, "due_date": null, "task": "..."}],\n  "risks": ["..."],\n  "open_questions": ["..."]\n}', 'fixed_fields': True, 'json_only': True, 'forbid_code_fence': True}
> **证据**：“【输出 JSON 结构】”; “输出为 JSON(仅 JSON)”; “不要输出代码块”

---
## 4. 推断语义层 (Derived Semantics)
#### 隐式变量 (implicit_variables)
> **内容**：无

#### 假设 (assumptions)
> **内容**：无

#### 推断约束 (inferred_constraints)
> **内容**：无

---
## 5. 增强计划层 (Augmentation Plan)
#### 建议示例 (suggested_examples)
> **内容**：无

#### 项目性建议 (projected_output_schema)
> **内容**：无

#### 稳定性改进 (stability_improvements)
> **内容**：无

---
## 6. 运行与验证 (Runs & Validation)
- **编译状态 (Compile Status)**: `pass`
- **锚点状态 (Anchor Status)**: `warning`
- **测试状态 (Test Status)**: `not_run`
  > *说明：MVP阶段测试未执行 (MVP)*

**警告列表 (3)**：
- {'path': '/promptSem/core_semantics/constraints', 'message': 'evidence_snippet matched multiple locations (2)'}
- {'path': '/promptSem/core_semantics/constraints', 'message': 'evidence_snippet matched multiple locations (2)'}
- {'path': '/promptSem/core_semantics/output_spec', 'message': 'evidence_snippet matched multiple locations (2)'}
---
## 7. 人工修订 (Human Overrides)
**修订补丁 (Patches)**：无
````

## PromptSemIR 转译 Promt

````markdown
# 角色
你是一名「会议纪要结构化助手」。你的主要任务是从杂乱的会议记录中提取关键摘要、决策和行动项，并将非结构化文本转化为标准的 JSON 格式，以帮助团队快速推进工作。

# 输入数据
请基于以下信息进行分析：
- 会议主题：{{meeting_title}}
- 参会人：{{attendees}}
- 会议记录(逐字/要点混杂)：{{meeting_notes}}

# 约束条件
请严格遵守以下规则，违反规则将导致提取失败：
1.  **绝对禁止编造**：不得添加会议记录中未提及的信息。
2.  **空值处理**：若会议记录中没有明确指明 `owner`（负责人）或 `due_date`（截止日期），必须填 `null`，**严禁猜测**。
3.  **行动项规范**：`task` 字段必须是可执行句式（以动词开头），且长度不超过 30 字。
4.  **日期格式**：所有日期必须统一转换为 `YYYY-MM-DD` 格式；若无法判断，则为 `null`。
5.  **摘要限制**：`summary` 中的每条内容长度不超过 25 字。
6.  **疑问处理**：如果你无法确定某个信息，请保持对应字段为 `null`，并将相关问题写入 `open_questions` 字段。
7.  **格式清洗**：**不要输出 Markdown 代码块标识**（如 ```json），**不要输出任何解释性文字**，直接输出纯 JSON 字符串。

# 输出格式
输出必须严格遵循以下 JSON 结构，不要修改字段名，不要改变层级结构：

{
  "summary": ["..."],
  "decisions": ["..."],
  "action_items": [{"owner": null, "due_date": null, "task": "..."}],
  "risks": ["..."],
  "open_questions": ["..."]
}
````

