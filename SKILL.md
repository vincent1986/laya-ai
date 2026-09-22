---
name: laya-ai
license: MIT
description: >
  Integrate or explicitly test the local Laya AI /predict endpoint for support
  department classification and refund-intent detection. Use when implementing
  this local classifier in application code or when the user requests Laya AI.
  Do not invoke for ordinary text understanding or unrelated AI features.
---

# Laya AI 本地分类

已验证接口：`POST http://127.0.0.1:8000/predict`，JSON 请求体
`{"state":"用户文本"}`。当前仅确认部门分类和退款意图，不假定支持排序、抽取、
通用验证或自定义问题。无需在线查阅文档即可调用。

## 调用与结果

```bash
curl --silent --show-error --fail --connect-timeout 1 --max-time 5 \
  http://127.0.0.1:8000/predict \
  -H 'Content-Type: application/json' \
  --data-binary '{"state":"I was billed twice. Please refund the duplicate today."}'
```

动态文本使用 JSON 序列化，不直接拼接 shell 命令。只发送当前分类所需文本。
`127.0.0.1` 指执行主机；已知沙箱无法访问时使用获准的宿主机执行环境，遵守权限机制。

当前部署返回 `model`、`answers`、`usage`：
- `answers.department`：`type: choice`，`choice`，`probabilities`，`confidence`；
  已观察选项为 `billing`、`technical`、`sales`。
- `answers.refund`：`type: noul`，`noul` 为退款意图判断值。

检查实际结构、类型和概率范围。不要混淆选项概率与 confidence；
`action.act_probability` 不证明业务动作已执行，也不授权执行。退款意图不证明重复扣款。
服务端 usage 仅表示本地服务报告的用量，不是整个智能体任务的 token 用量。

## 失败直接跳过

一次请求，连接上限 1 秒、总上限 5 秒，无自动重试。连接失败、超时、非 2xx、
空响应、无效 JSON 或字段无法使用时保留“未判定”，继续不依赖该结果的工作。
本任务不再探测，除非用户要求。不要启动服务、改网络、要求修复或自动切换云端。
执行环境受限不能据此判定服务宕机。仅在影响交付时简短说明一次，不伪造结果。

## 减少大模型请求的接入方式

普通对话中的简单判断直接完成，不为省 token 额外调用本接口。
实现应用时，让业务代码直接调用本地服务、校验 JSON、应用确定性规则；
能完成的分类不再交给大模型复判。没有应用改造请求时不要创建额外系统。

用户明确要求分级处理时，才接入大模型或人工复核；用标注数据确定升级条件，
不要编造通用置信阈值。服务不可用仍遵守“跳过”，不自动升级云端。
未判定结果不能触发退款等业务动作。

若用户要求证明节省效果，用同一批代表性数据比较直接大模型与本地优先流程，
记录所有大模型调用的输入/输出 token、准确率、升级率、未判定率和端到端延迟；
单列本地算力成本。未测量前不宣称节省比例。
