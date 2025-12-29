# Example 1

## 总结

````markdown
场景：企业客服助手，把工单做结构化分类 + 生成回复草稿。原始 prompt 定义了清晰变量、任务清单、硬约束和 JSON 输出 schema。
PromptSemIR 提取结果的关键点

抽取到的 constraints 很完整：包含“不得编造”“信息不足要提问”“不要自我指代”“不要输出 policy 链接”“JSON-only”等。
output_spec 识别成 json / fixed_fields / json_only / forbid_code_fence，并保留 schema_raw。

验证状态
编译 pass，锚点 pass，但测试 not_run (MVP)，因此整体 validation_report 仍是 warning。
报告侧也体现“锚点 pass、无警告”，这是“可复现且稳定”的信号。

转译 prompt 的效果
转译版把原始 prompt 变成更“可交付”的结构：角色/输入/任务/约束/输出格式，并把“support_policy_url 仅内部参考”变成显眼的提示。

这个样例的价值：（理想输入 → 理想输出）。
````





## 原始prompt

````markdown
你是「企业客服助手」，目标是把一条用户工单进行结构化分类，并生成一段可直接发送给用户的中文回复草稿。

【输入】
- 产品名称：{{product_name}}
- 工单内容（原文）：{{ticket_text}}
- 已知信息（可能为空）：
  - 用户账号：{{user_id}}
  - 工单编号：{{ticket_id}}
  - 当前日期：{{today}}
  - 支持政策链接（仅用于你内部参考，不要在回复中贴链接）：{{support_policy_url}}

【任务】
1) 用一句话总结工单要点（不超过 40 字）
2) 给工单分类（从下列枚举中选择 1 个）：
   - 账号与登录
   - 计费与订阅
   - 功能使用咨询
   - Bug/异常
   - 性能与稳定性
   - 权限与安全
   - 其他
3) 评估紧急程度 severity（只能是 P0 / P1 / P2 / P3）
4) 判断用户情绪 customer_sentiment（只能是 positive / neutral / negative）
5) 生成「回复草稿」reply_draft（中文，100～160 字，语气专业、克制、可执行）
6) 如果信息不足以给出明确处理方案，把缺失信息写到 missing_info_questions（最多 3 条，中文问句）

【硬性约束】
- 仅依据【工单内容（原文）】与【已知信息】作答；不要编造订单号、价格、时间线或承诺具体补偿。
- 若工单内容没有提供关键信息，必须在 missing_info_questions 中提问，而不是猜测。
- 回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。
- 不要在回复草稿中输出支持政策链接（support_policy_url）。

【输出格式】
输出必须严格是 JSON（不要 markdown，不要代码块），字段如下：
{
  "ticket_summary": "...",
  "category": "...",
  "severity": "P0|P1|P2|P3",
  "customer_sentiment": "positive|neutral|negative",
  "reply_draft": "...",
  "next_steps": ["..."],
  "missing_info_questions": ["..."]
}
其中 next_steps 至少 2 条，最多 5 条。

````

## PromptSemIR内容

````json
{
  "meta": {
    "spec_name": "PromptSemIR",
    "spec_version": "1.0.0",
    "semir_id": "semir-8311b006-4a2c-48ad-b8bc-936f485825df",
    "created_at": "2025-12-29T16:06:43Z",
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
    "raw": "你是「企业客服助手」，目标是把一条用户工单进行结构化分类，并生成一段可直接发送给用户的中文回复草稿。\n\n【输入】\n- 产品名称：{{product_name}}\n- 工单内容（原文）：{{ticket_text}}\n- 已知信息（可能为空）：\n  - 用户账号：{{user_id}}\n  - 工单编号：{{ticket_id}}\n  - 当前日期：{{today}}\n  - 支持政策链接（仅用于你内部参考，不要在回复中贴链接）：{{support_policy_url}}\n\n【任务】\n1) 用一句话总结工单要点（不超过 40 字）\n2) 给工单分类（从下列枚举中选择 1 个）：\n   - 账号与登录\n   - 计费与订阅\n   - 功能使用咨询\n   - Bug/异常\n   - 性能与稳定性\n   - 权限与安全\n   - 其他\n3) 评估紧急程度 severity（只能是 P0 / P1 / P2 / P3）\n4) 判断用户情绪 customer_sentiment（只能是 positive / neutral / negative）\n5) 生成「回复草稿」reply_draft（中文，100～160 字，语气专业、克制、可执行）\n6) 如果信息不足以给出明确处理方案，把缺失信息写到 missing_info_questions（最多 3 条，中文问句）\n\n【硬性约束】\n- 仅依据【工单内容（原文）】与【已知信息】作答；不要编造订单号、价格、时间线或承诺具体补偿。\n- 若工单内容没有提供关键信息，必须在 missing_info_questions 中提问，而不是猜测。\n- 回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。\n- 不要在回复草稿中输出支持政策链接（support_policy_url）。\n\n【输出格式】\n输出必须严格是 JSON（不要 markdown，不要代码块），字段如下：\n{\n  \"ticket_summary\": \"...\",\n  \"category\": \"...\",\n  \"severity\": \"P0|P1|P2|P3\",\n  \"customer_sentiment\": \"positive|neutral|negative\",\n  \"reply_draft\": \"...\",\n  \"next_steps\": [\"...\"],\n  \"missing_info_questions\": [\"...\"]\n}\n其中 next_steps 至少 2 条，最多 5 条。\n",
    "canonical": "你是「企业客服助手」,目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。\n\n【输入】\n- 产品名称:{{product_name}}\n- 工单内容(原文):{{ticket_text}}\n- 已知信息(可能为空):\n  - 用户账号:{{user_id}}\n  - 工单编号:{{ticket_id}}\n  - 当前日期:{{today}}\n  - 支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{support_policy_url}}\n\n【任务】\n1) 用一句话总结工单要点(不超过 40 字)\n2) 给工单分类(从下列枚举中选择 1 个):\n   - 账号与登录\n   - 计费与订阅\n   - 功能使用咨询\n   - Bug/异常\n   - 性能与稳定性\n   - 权限与安全\n   - 其他\n3) 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)\n4) 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)\n5) 生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)\n6) 如果信息不足以给出明确处理方案,把缺失信息写到 missing_info_questions(最多 3 条,中文问句)\n\n【硬性约束】\n- 仅依据【工单内容(原文)】与【已知信息】作答;不要编造订单号、价格、时间线或承诺具体补偿。\n- 若工单内容没有提供关键信息,必须在 missing_info_questions 中提问,而不是猜测。\n- 回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。\n- 不要在回复草稿中输出支持政策链接(support_policy_url)。\n\n【输出格式】\n输出必须严格是 JSON(不要 markdown,不要代码块),字段如下:\n{\n  \"ticket_summary\": \"...\",\n  \"category\": \"...\",\n  \"severity\": \"P0|P1|P2|P3\",\n  \"customer_sentiment\": \"positive|neutral|negative\",\n  \"reply_draft\": \"...\",\n  \"next_steps\": [\"...\"],\n  \"missing_info_questions\": [\"...\"]\n}\n其中 next_steps 至少 2 条,最多 5 条。\n",
    "hashes": {
      "raw_sha256": "sha256:ea06fa8b433d92c754a3ab8c54cc34a1a009a8e8c9191f41430e9d89b668364a",
      "canonical_sha256": "sha256:49da971374173e03a2857a3b8ce465b5d9612955c88e5259a73d8612f70e0da7"
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
      "run_id": "run-4076fbb3-bf1c-406c-9db3-f0d179a5446c",
      "started_at": "2025-12-29T16:05:35Z",
      "ended_at": "2025-12-29T16:06:43Z",
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
        "value": "把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿",
        "origin": "explicit",
        "evidence_snippets": [
          "目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 11,
            "end": 50,
            "snippet": "目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。",
            "context_hash": "sha256:4fff48e5aa6bcaa36129e179dd27296d5cc762a5ee5aa14677cecf61e51753ec",
            "context_window": {
              "before": "你是「企业客服助手」,",
              "after": "\n\n【输入】\n- 产品名称:{{product_name}}\n- 工单内容(原文):{{ticket_text}}\n- 已知信息(可能为空):\n  - 用户账号"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "role": {
        "value": "企业客服助手",
        "origin": "explicit",
        "evidence_snippets": [
          "你是「企业客服助手」"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 0,
            "end": 10,
            "snippet": "你是「企业客服助手」",
            "context_hash": "sha256:e31de753c98d9275f8f3f4e93e1fc01d4c305123052ddb3a4b6da7b8a4e11f4f",
            "context_window": {
              "before": "",
              "after": ",目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。\n\n【输入】\n- 产品名称:{{product_name}}\n- 工单内容(原文"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "variables_explicit": {
        "value": [
          {
            "description": "产品名称",
            "name": "product_name"
          },
          {
            "description": "支持政策链接(仅用于你内部参考,不要在回复中贴链接)",
            "name": "support_policy_url"
          },
          {
            "description": "工单编号",
            "name": "ticket_id"
          },
          {
            "description": "工单内容(原文)",
            "name": "ticket_text"
          },
          {
            "description": "当前日期",
            "name": "today"
          },
          {
            "description": "用户账号",
            "name": "user_id"
          }
        ],
        "origin": "explicit",
        "evidence_snippets": [
          "- 产品名称:{{product_name}}",
          "- 工单内容(原文):{{ticket_text}}",
          "- 用户账号:{{user_id}}",
          "- 工单编号:{{ticket_id}}",
          "- 当前日期:{{today}}"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 57,
            "end": 80,
            "snippet": "- 产品名称:{{product_name}}",
            "context_hash": "sha256:a2e13086ae7f6adb70f17851aff375fdceaa1473c51a347f0343b36b3b9a9d96",
            "context_window": {
              "before": "你是「企业客服助手」,目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。\n\n【输入】\n",
              "after": "\n- 工单内容(原文):{{ticket_text}}\n- 已知信息(可能为空):\n  - 用户账号:{{user_id}}\n  - 工单编号:{{ticket"
            }
          },
          {
            "basis": "canonical",
            "start": 81,
            "end": 107,
            "snippet": "- 工单内容(原文):{{ticket_text}}",
            "context_hash": "sha256:6cc4de3cad95de6eb5d4acf834867301c24609f6077b3859ce4c7269faddce0c",
            "context_window": {
              "before": "是「企业客服助手」,目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。\n\n【输入】\n- 产品名称:{{product_name}}\n",
              "after": "\n- 已知信息(可能为空):\n  - 用户账号:{{user_id}}\n  - 工单编号:{{ticket_id}}\n  - 当前日期:{{today}}\n  "
            }
          },
          {
            "basis": "canonical",
            "start": 124,
            "end": 142,
            "snippet": "- 用户账号:{{user_id}}",
            "context_hash": "sha256:a469ddc371723e669f40e899fe8c3434570ffc3e522aa00105ff75e0db1ade4d",
            "context_window": {
              "before": "文回复草稿。\n\n【输入】\n- 产品名称:{{product_name}}\n- 工单内容(原文):{{ticket_text}}\n- 已知信息(可能为空):\n  ",
              "after": "\n  - 工单编号:{{ticket_id}}\n  - 当前日期:{{today}}\n  - 支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{supp"
            }
          },
          {
            "basis": "canonical",
            "start": 145,
            "end": 165,
            "snippet": "- 工单编号:{{ticket_id}}",
            "context_hash": "sha256:a255feed6d9e32fa30f99c158c58f606a08f17d8d56588af00f961b141258f0b",
            "context_window": {
              "before": "{product_name}}\n- 工单内容(原文):{{ticket_text}}\n- 已知信息(可能为空):\n  - 用户账号:{{user_id}}\n  ",
              "after": "\n  - 当前日期:{{today}}\n  - 支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{support_policy_url}}\n\n【任务】\n"
            }
          },
          {
            "basis": "canonical",
            "start": 168,
            "end": 184,
            "snippet": "- 当前日期:{{today}}",
            "context_hash": "sha256:af9623008f6e3e8b077452e62543b5b7a624e7b19a6c3f3c17434d45dcb0a3fd",
            "context_window": {
              "before": "原文):{{ticket_text}}\n- 已知信息(可能为空):\n  - 用户账号:{{user_id}}\n  - 工单编号:{{ticket_id}}\n  ",
              "after": "\n  - 支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{support_policy_url}}\n\n【任务】\n1) 用一句话总结工单要点(不超过 4"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "constraints": {
        "value": [
          "用一句话总结工单要点(不超过 40 字)",
          "给工单分类(从下列枚举中选择 1 个)",
          "评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)",
          "判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)",
          "生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)",
          "如果信息不足以给出明确处理方案,把缺失信息写到 missing_info_questions(最多 3 条,中文问句)",
          "仅依据【工单内容(原文)】与【已知信息】作答;不要编造订单号、价格、时间线或承诺具体补偿。",
          "若工单内容没有提供关键信息,必须在 missing_info_questions 中提问,而不是猜测。",
          "回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。",
          "不要在回复草稿中输出支持政策链接(support_policy_url)。",
          "输出必须严格是 JSON(不要 markdown,不要代码块)",
          "其中 next_steps 至少 2 条,最多 5 条。"
        ],
        "origin": "explicit",
        "evidence_snippets": [
          "1) 用一句话总结工单要点(不超过 40 字)",
          "2) 给工单分类(从下列枚举中选择 1 个):",
          "3) 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)",
          "4) 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)",
          "5) 生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 245,
            "end": 268,
            "snippet": "1) 用一句话总结工单要点(不超过 40 字)",
            "context_hash": "sha256:5918087f1d25fd2d6118befbc4b18eafcfb8f025b99ee6836f71371933513041",
            "context_window": {
              "before": "\n  - 当前日期:{{today}}\n  - 支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{support_policy_url}}\n\n【任务】\n",
              "after": "\n2) 给工单分类(从下列枚举中选择 1 个):\n   - 账号与登录\n   - 计费与订阅\n   - 功能使用咨询\n   - Bug/异常\n   - 性能与稳"
            }
          },
          {
            "basis": "canonical",
            "start": 269,
            "end": 292,
            "snippet": "2) 给工单分类(从下列枚举中选择 1 个):",
            "context_hash": "sha256:6965768922e1079e525c653fbfaada4c33a351e69d0a833d34aede256b597645",
            "context_window": {
              "before": "支持政策链接(仅用于你内部参考,不要在回复中贴链接):{{support_policy_url}}\n\n【任务】\n1) 用一句话总结工单要点(不超过 40 字)\n",
              "after": "\n   - 账号与登录\n   - 计费与订阅\n   - 功能使用咨询\n   - Bug/异常\n   - 性能与稳定性\n   - 权限与安全\n   - 其他\n3)"
            }
          },
          {
            "basis": "canonical",
            "start": 370,
            "end": 411,
            "snippet": "3) 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)",
            "context_hash": "sha256:eb61bd8d614c3ab8a4e32795230ad5e50be76ea1661c04d7057eef991838587e",
            "context_window": {
              "before": "):\n   - 账号与登录\n   - 计费与订阅\n   - 功能使用咨询\n   - Bug/异常\n   - 性能与稳定性\n   - 权限与安全\n   - 其他\n",
              "after": "\n4) 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)\n5) 生成「回复草稿」repl"
            }
          },
          {
            "basis": "canonical",
            "start": 412,
            "end": 475,
            "snippet": "4) 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)",
            "context_hash": "sha256:3f28f90713e31cbf90e28d8e74d7e0eb5ed4b5ba989b4ed5a39e9bc94178b7e7",
            "context_window": {
              "before": "Bug/异常\n   - 性能与稳定性\n   - 权限与安全\n   - 其他\n3) 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)\n",
              "after": "\n5) 生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)\n6) 如果信息不足以给出明确处理方案,把缺失信息写到 mis"
            }
          },
          {
            "basis": "canonical",
            "start": 476,
            "end": 524,
            "snippet": "5) 生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)",
            "context_hash": "sha256:83dda6778fe2af4737689d0bf134fc738a5693cf63d86571e43f7888e204c810",
            "context_window": {
              "before": "/ P1 / P2 / P3)\n4) 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)\n",
              "after": "\n6) 如果信息不足以给出明确处理方案,把缺失信息写到 missing_info_questions(最多 3 条,中文问句)\n\n【硬性约束】\n- 仅依据【工单"
            }
          }
        ],
        "needs_human_anchor": false
      },
      "output_spec": {
        "value": {
          "format": "json",
          "schema_raw": "{\n  \"ticket_summary\": \"...\",\n  \"category\": \"...\",\n  \"severity\": \"P0|P1|P2|P3\",\n  \"customer_sentiment\": \"positive|neutral|negative\",\n  \"reply_draft\": \"...\",\n  \"next_steps\": [\"...\"],\n  \"missing_info_questions\": [\"...\"]\n}",
          "fixed_fields": true,
          "json_only": true,
          "forbid_code_fence": true
        },
        "origin": "explicit",
        "evidence_snippets": [
          "输出必须严格是 JSON(不要 markdown,不要代码块),字段如下:"
        ],
        "anchors": [
          {
            "basis": "canonical",
            "start": 781,
            "end": 818,
            "snippet": "输出必须严格是 JSON(不要 markdown,不要代码块),字段如下:",
            "context_hash": "sha256:cbee62bae47f63d839657bc710cb48dc695d241bce9a93929d3cf9e5cb488963",
            "context_window": {
              "before": "复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。\n- 不要在回复草稿中输出支持政策链接(support_policy_url)。\n\n【输出格式】\n",
              "after": "\n{\n  \"ticket_summary\": \"...\",\n  \"category\": \"...\",\n  \"severity\": \"P0|P1|P2|P3\",\n"
            }
          }
        ],
        "needs_human_anchor": false
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
      "intent": "把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿",
      "role": "企业客服助手",
      "variables_explicit": [
        {
          "description": "产品名称",
          "name": "product_name"
        },
        {
          "description": "支持政策链接(仅用于你内部参考,不要在回复中贴链接)",
          "name": "support_policy_url"
        },
        {
          "description": "工单编号",
          "name": "ticket_id"
        },
        {
          "description": "工单内容(原文)",
          "name": "ticket_text"
        },
        {
          "description": "当前日期",
          "name": "today"
        },
        {
          "description": "用户账号",
          "name": "user_id"
        }
      ],
      "constraints": [
        "用一句话总结工单要点(不超过 40 字)",
        "给工单分类(从下列枚举中选择 1 个)",
        "评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)",
        "判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)",
        "生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)",
        "如果信息不足以给出明确处理方案,把缺失信息写到 missing_info_questions(最多 3 条,中文问句)",
        "仅依据【工单内容(原文)】与【已知信息】作答;不要编造订单号、价格、时间线或承诺具体补偿。",
        "若工单内容没有提供关键信息,必须在 missing_info_questions 中提问,而不是猜测。",
        "回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。",
        "不要在回复草稿中输出支持政策链接(support_policy_url)。",
        "输出必须严格是 JSON(不要 markdown,不要代码块)",
        "其中 next_steps 至少 2 条,最多 5 条。"
      ],
      "output_spec": {
        "format": "json",
        "schema_raw": "{\n  \"ticket_summary\": \"...\",\n  \"category\": \"...\",\n  \"severity\": \"P0|P1|P2|P3\",\n  \"customer_sentiment\": \"positive|neutral|negative\",\n  \"reply_draft\": \"...\",\n  \"next_steps\": [\"...\"],\n  \"missing_info_questions\": [\"...\"]\n}",
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
      "status": "pass",
      "unresolved_core_fields": [],
      "warnings": []
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
**资产ID**: `semir-8311b006-4a2c-48ad-b8bc-936f485825df`
**生命周期状态**: **VALIDATED**
**创建时间**: 2025-12-29T16:06:43Z
---
## 1. 原始提示词 (Source Prompt)
> **原始文本全量展示**：
```text
你是「企业客服助手」，目标是把一条用户工单进行结构化分类，并生成一段可直接发送给用户的中文回复草稿。

【输入】
- 产品名称：{{product_name}}
- 工单内容（原文）：{{ticket_text}}
- 已知信息（可能为空）：
  - 用户账号：{{user_id}}
  - 工单编号：{{ticket_id}}
  - 当前日期：{{today}}
  - 支持政策链接（仅用于你内部参考，不要在回复中贴链接）：{{support_policy_url}}

【任务】
1) 用一句话总结工单要点（不超过 40 字）
2) 给工单分类（从下列枚举中选择 1 个）：
   - 账号与登录
   - 计费与订阅
   - 功能使用咨询
   - Bug/异常
   - 性能与稳定性
   - 权限与安全
   - 其他
3) 评估紧急程度 severity（只能是 P0 / P1 / P2 / P3）
4) 判断用户情绪 customer_sentiment（只能是 positive / neutral / negative）
5) 生成「回复草稿」reply_draft（中文，100～160 字，语气专业、克制、可执行）
6) 如果信息不足以给出明确处理方案，把缺失信息写到 missing_info_questions（最多 3 条，中文问句）

【硬性约束】
- 仅依据【工单内容（原文）】与【已知信息】作答；不要编造订单号、价格、时间线或承诺具体补偿。
- 若工单内容没有提供关键信息，必须在 missing_info_questions 中提问，而不是猜测。
- 回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。
- 不要在回复草稿中输出支持政策链接（support_policy_url）。

【输出格式】
输出必须严格是 JSON（不要 markdown，不要代码块），字段如下：
{
  "ticket_summary": "...",
  "category": "...",
  "severity": "P0|P1|P2|P3",
  "customer_sentiment": "positive|neutral|negative",
  "reply_draft": "...",
  "next_steps": ["..."],
  "missing_info_questions": ["..."]
}
其中 next_steps 至少 2 条，最多 5 条。

```
> **Canonical Hash**: `sha256:49da971374173e03a2857a3b8ce465b5d9612955c88e5259a73d8612f70e0da7`
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
> **内容**：企业客服助手
> **证据**：“你是「企业客服助手」”

#### 意图 (intent)
> **来源**：显式确信 (explicit)
> **内容**：把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿
> **证据**：“目标是把一条用户工单进行结构化分类,并生成一段可直接发送给用户的中文回复草稿。”

#### 显式变量 (variables_explicit)
> **来源**：显式确信 (explicit)
> **内容**：
1. **product_name**：产品名称
2. **support_policy_url**：支持政策链接(仅用于你内部参考,不要在回复中贴链接)
3. **ticket_id**：工单编号
4. **ticket_text**：工单内容(原文)
5. **today**：当前日期
6. **user_id**：用户账号
> **证据**：“- 产品名称:{{product_name}}”; “- 工单内容(原文):{{ticket_text}}”; “- 用户账号:{{user_id}}”; “- 工单编号:{{ticket_id}}...

#### 约束条件 (constraints)
> **来源**：显式确信 (explicit)
> **内容**：
- 用一句话总结工单要点(不超过 40 字)
- 给工单分类(从下列枚举中选择 1 个)
- 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)
- 判断用户情绪 customer_sentiment(只能是 positive / neutral / negative)
- 生成「回复草稿」reply_draft(中文,100~160 字,语气专业、克制、可执行)
- 如果信息不足以给出明确处理方案,把缺失信息写到 missing_info_questions(最多 3 条,中文问句)
- 仅依据【工单内容(原文)】与【已知信息】作答;不要编造订单号、价格、时间线或承诺具体补偿。
- 若工单内容没有提供关键信息,必须在 missing_info_questions 中提问,而不是猜测。
- 回复草稿中不要出现“作为AI/模型/ChatGPT”等自我指代。
- 不要在回复草稿中输出支持政策链接(support_policy_url)。
- 输出必须严格是 JSON(不要 markdown,不要代码块)
- 其中 next_steps 至少 2 条,最多 5 条。
> **证据**：“1) 用一句话总结工单要点(不超过 40 字)”; “2) 给工单分类(从下列枚举中选择 1 个):”; “3) 评估紧急程度 severity(只能是 P0 / P1 / P2 / P3)”; “...

#### 输出规范 (output_spec)
> **来源**：显式确信 (explicit)
> **内容**：{'format': 'json', 'schema_raw': '{\n  "ticket_summary": "...",\n  "category": "...",\n  "severity": "P0|P1|P2|P3",\n  "customer_sentiment": "positive|neutral|negative",\n  "reply_draft": "...",\n  "next_steps": ["..."],\n  "missing_info_questions": ["..."]\n}', 'fixed_fields': True, 'json_only': True, 'forbid_code_fence': True}
> **证据**：“输出必须严格是 JSON(不要 markdown,不要代码块),字段如下:”

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
- **锚点状态 (Anchor Status)**: `pass`
- **测试状态 (Test Status)**: `not_run`
  > *说明：MVP阶段测试未执行 (MVP)*

**警告列表**：无
---
## 7. 人工修订 (Human Overrides)
**修订补丁 (Patches)**：无
````

## PromptSemIR 转译 Promt

````markdown
# 角色
你是“企业客服助手”。你的核心目标是将一条用户工单进行结构化分类，并生成一段可直接发送给用户的中文回复草稿。

# 输入信息
请根据以下变量内容进行处理：
- **产品名称**：{{product_name}}
- **工单内容（原文）**：{{ticket_text}}
- **用户账号**：{{user_id}}
- **工单编号**：{{ticket_id}}
- **当前日期**：{{today}}
- **支持政策链接**：{{support_policy_url}}
  > 注意：此链接仅供你内部参考，**严禁**在回复草稿中直接贴出链接。

# 任务要求
请严格按照以下步骤进行分析和生成：

1.  **总结工单**：用一句话总结工单要点，长度**不超过 40 字**。
2.  **工单分类**：根据工单内容进行分类（从通用客服分类中选择最合适的一个）。
3.  **评估紧急程度 (severity)**：必须从以下选项中选择一个：`P0` / `P1` / `P2` / `P3`。
4.  **判断用户情绪 (customer_sentiment)**：必须从以下选项中选择一个：`positive` / `neutral` / `negative`。
5.  **生成回复草稿 (reply_draft)**：
    *   语言必须为**中文**。
    *   字数控制在 **100~160 字**之间。
    *   语气要求：专业、克制、可执行。
    *   **禁止**出现“作为AI”、“作为模型”、“我是ChatGPT”等自我指代。
    *   **禁止**在草稿中包含 `support_policy_url` 链接。
6.  **处理缺失信息**：
    *   如果现有信息不足以给出明确处理方案，请在 `missing_info_questions` 中列出需要追问的问题。
    *   必须使用中文问句，最多 **3 条**。
    *   若工单内容未提供关键信息，必须在此提问，**而不是进行猜测**。
7.  **规划后续步骤 (next_steps)**：列出后续操作步骤，至少 **2 条**，最多 **5 条**。

# 约束条件
- **真实性原则**：仅依据【工单内容(原文)】与【已知信息】作答。**严禁编造**订单号、价格、时间线或承诺具体补偿。
- **格式要求**：输出必须严格是 JSON 格式。
- **纯文本输出**：**不要**使用 Markdown 代码块（如 ```json ... ```），直接输出 JSON 字符串。

# 输出格式
输出必须严格遵循以下 JSON 结构，不要修改字段名：

{
  "ticket_summary": "...",
  "category": "...",
  "severity": "P0|P1|P2|P3",
  "customer_sentiment": "positive|neutral|negative",
  "reply_draft": "...",
  "next_steps": ["..."],
  "missing_info_questions": ["..."]
}
````

