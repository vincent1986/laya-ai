---
name: laya-ai
license: MIT
description: >
  Use local Laya AI to judge tool-call intent and route requests among configured
  tools or handlers. Apply when integrating tool-request decisions into application
  code or when the user requests Laya AI. Department and refund judgments are
  examples, not the scope boundary. Avoid extra calls for ordinary conversation.
---

# Laya AI 工具调用意图判断

已验证接口：`POST http://127.0.0.1:8000/predict`，JSON 请求体
`{"state":"用户文本"}`。用途是判断请求是否表达工具调用意图，以及应路由到哪个
已配置的工具或处理器。部门分类与退款意图是已验证示例，不是 skill 的业务边界。
无需在线查阅文档即可调用。

## 工具调用边界

先依据应用已有的工具清单和服务端问题配置确定可判定范围。语义意图由本地模型
提供，代码负责将结果映射到允许的工具、校验参数并执行既有业务规则。
需要区分执行请求、仅询问信息、否定请求和无匹配情形；是否支持这些区分必须
由服务配置或测试确认，不能从示例推断。部门标签不等同于工具名称。

当前只实测了 department/refund 输出，尚未证明该部署能路由任意工具。
接入其他工具时先查看已有实现或接口文档确认配置；仅修改本 skill 不会扩展模型
能力。已知请求体只有 state，不擅自添加 tools、instructions、criteria 等字段。
缺少匹配的判断或参数时保留未判定，不编造工具名、参数或执行结果。

## 调用与结果

```bash
curl --silent --show-error --fail --connect-timeout 1 --max-time 5 \
  http://127.0.0.1:8000/predict \
  -H 'Content-Type: application/json' \
  --data-binary '{"state":"I was billed twice. Please refund the duplicate today."}'
```

动态文本使用 JSON 序列化，不直接拼接 shell 命令。只发送当前意图判断所需文本。
`127.0.0.1` 指执行主机；已知沙箱无法访问时使用获准的宿主机执行环境，遵守权限机制。

已验证示例部署返回 `model`、`answers`、`usage`：
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
已支持且能完成的工具意图判断不再交给大模型复判。没有应用改造请求时不要创建额外系统。

用户明确要求分级处理时，才接入大模型或人工复核；用标注数据确定升级条件，
不要编造通用置信阈值。服务不可用仍遵守“跳过”，不自动升级云端。
未判定结果不能触发退款等业务动作。

若用户要求证明节省效果，用同一批代表性数据比较直接大模型与本地优先流程，
记录所有大模型调用的输入/输出 token、准确率、升级率、未判定率和端到端延迟；
单列本地算力成本。未测量前不宣称节省比例。
