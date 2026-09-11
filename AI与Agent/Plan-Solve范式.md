---
title: "Plan-Solve范式"
date: 2026-07-17
section: "Agent"
source: http://120.77.152.123:8088/posts/Agent-framework-plan&Solve
tags:
  - "Agent"
  - "博客迁移"
---
Plan&Solve范式

适合需要`长远规划`的复杂任务，避免智能体在ReAct这种步进式的执行中远离目标的情况。

两个阶段：

- Planning Phase: 对复杂任务先思考，得出清晰完整的执行计划。
- Solving Phase：严格按照计划执行动作，最终的结果是根据原始问题、每一步执行过程及其前一步的执行结果而来。

规划器封装`Planner`：

提示词工程体现：

- 明确LLM的角色
- 明确LLM的任务：复杂任务->一步步独立进行的任务
- 规范LLM的格式：输出字符串存在python列表里，比解析自然语言更省事

执行器封装`Executor`：

这个范式重要的是把`历史记录作为上下文传递`。

重心在维护状态（历史执行与其结果，这是每一个步骤的直接输入）：

与Planner的Prompt不同，Executor的专注于：根据上下文信息，专注解决当下步骤。

传递包含：

- 原始问题
- 完整计划
- 历史步骤与对应的结果
- 当前步骤
