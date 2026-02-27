---
title: 20240629 - 梳理ChatDBA的结构 (第二版发版前)
confluence_page_id: 2949862
created_at: 2024-06-29T15:52:59+00:00
updated_at: 2024-06-29T15:52:59+00:00
---

# 知识点

  - chatdba/utils/component/pipelines/pipelinebuilder.py, pipeline的构建被移动到了这里

# 流程

  1. router_text_or_image: 判断文字或图片
     1. 不讨论图片
  2. 对问题进行重写 ?? (好像没有重写阶段)
  3. intend_input: 让LLM直接回答问题
  4. topic_manager: 使用上一步的LLM的答案, 进行意图识别 (而不是话题管理)
  5. 文档召回: fuzzy & openai & mcontriever
  6. jump_judge: 文档的有效性判断?? (名字需要调整)
     1. 如果无效, 则需要update_plan
     2. 否则, 使用当前plan
  7. update_plan: 更新plan (也没有使用新方案)
  8. error_manager: 话题管理 (话题限于 某种故障)
  9. final_refine
  10. merge_refines (实际是对refine的反思)
