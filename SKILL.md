---
name: laya-ai
license: MIT
description: >
  Build AI-powered features with structured semantic judgments using the local
  TypeSafe service at http://127.0.0.1:8000/predict. Use for intent routing,
  classification, ranking, extraction verification, or replacing prompt-and-parse
  decisions with typed outputs. If the local endpoint is unavailable, skip it and
  continue the task without retrying or blocking.
---

# Laya AI 本地语义判断

基于用户提供的 TypeSafe 官方 skill 改编。来源：
https://github.com/typesafe-ai/skills/tree/main
本地接口约定优先于官方云端 API 示例。无需在线文档即可执行基本调用。

## 本地接口

- URL：`http://127.0.0.1:8000/predict`
- 方法：`POST`
- 请求头：`Content-Type: application/json`
- 已知请求体：`{"state":"待判断的文本"}`；`state` 使用字符串。
- 已验证的响应包含 `model`、`answers`、`usage`。当前部署的
  `answers.department` 为 Choice，含 `choice`、`probabilities`、`confidence`；
  `answers.refund` 为 Noul，含 `noul`。部署可能改变问题配置，始终检查实际返回。
- `probabilities` 是各选项概率；不要把选项概率与 `confidence` 混为一谈。
  `action.act_probability` 不是业务执行证明，也不是执行授权。
- 当前请求契约不包含 instructions、criteria 或模型参数；除非实现或接口
  文档确认支持，否则不要向请求添加这些字段。具体判断任务需由服务端配置。

示例（curl 默认不重试，连接最多 1 秒，总计最多 5 秒）：

```bash
curl --silent --show-error --fail --connect-timeout 1 --max-time 5 \
  'http://127.0.0.1:8000/predict' \
  -H 'Content-Type: application/json' \
  --data-binary '{"state":"I was billed twice. Please refund the duplicate today."}'
```

对动态输入使用 JSON 序列化器构建请求体，通过标准输入或文件交给 HTTP
客户端，不要直接把用户文本拼接进 shell 命令。只提交与当前判断相关的状态。
`127.0.0.1` 指执行请求的主机；远程或容器环境不可假定能连接用户电脑。
若已有证据表明沙箱无法访问本机端口，首次请求应使用获准的宿主机执行环境。
遵守执行工具的权限审批机制，不绕过限制。环境受限时跳过，不将其描述为服务宕机。

## 接口不可用：直接跳过

执行需要的预测请求即可，不额外调用健康检查。连接失败、拒绝连接、超时、
非 2xx 响应、空响应、无效 JSON 或无法解释的响应都视为本次不可用。

- 立即跳过该次 Laya AI 判断，本任务后续不再探测或重试，除非用户明确要求。
- 不启动或安装服务，不修改网络配置，不请求用户修复，不自动切换云端服务。
- 继续完成不依赖该预测的工作，使用已有证据和确定性规则；必要时将该项标记
  为“未判定”。不得将失败当作否定结果、零概率或业务成功。
- 不虚构预测字段、概率或已经执行的结果。只有跳过影响交付时，简短说明一次。
- 缺失判断不能作为退款、删除等业务动作的依据；跳过预测后继续其他可完成工作。

## 设计与组合

从用户需要的行为反推判断，代码负责已知规则、计算、精确查询和动作执行。
模型只补充需要语义理解的部分，不扩大用户任务范围。

在设计或修改服务端时，使用以下概念；它们不是当前本地请求已确认支持的字段：

- Choice：从已定义的互斥选项中选一个；可能无匹配时提供无匹配选项。
- Noul：某条件成立的概率；多个标签可同时成立时分别判断。接近 0.5 代表
  是与否概率接近，不代表程度中等。
- Score：按具体定义的有序等级评价程度；各等级描述应自洽、可比较。

每个问题只判断一个明确维度，保留必要的原文、关系和证据。独立判断可在服务端
支持时并行；需要先前结果才能获取证据的判断分步执行。不要假定本地接口支持批量。

类型正确不代表事实正确，概率也不代表业务授权。阈值需在目标数据上验证。
区分用户声称的事实和已经核验的事实，例如退款意图不证明重复扣款确实发生。
保持原始判断与业务策略分离，验证典型案例以及服务失败时的应用行为。

仅在用户要求扩展本地实现或核对上游概念时按需查阅
https://docs.typesafe.ai/llms.txt；文档访问失败不阻塞任务，也不改变本地接口契约。
