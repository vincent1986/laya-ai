# Laya AI Skill

面向 Codex 等支持 Agent Skills 的智能体：接入本地 Laya AI 服务，判断工具调用类请求的意图，并路由到已配置的工具或处理器。部门分类与退款意图是示例，不是用途边界。

## 安装

将本仓库克隆到全局 skills 目录（目标目录不存在时）：

```bash
git clone https://github.com/vincent1986/laya-ai.git "${CODEX_HOME:-$HOME/.codex}/skills/laya-ai"
```

在支持的客户端中通过 `$laya-ai` 显式调用，或在实现工具调用意图判断与路由时自动匹配。普通文本理解不会因此额外调用接口。

## 接口

需自行提供运行在本机的预测服务。本仓库只包含 skill，不包含模型权重或服务端实现。

```bash
curl --silent --show-error --fail --connect-timeout 1 --max-time 5 \
  http://127.0.0.1:8000/predict \
  -H 'Content-Type: application/json' \
  --data-binary '{"state":"I was billed twice. Please refund the duplicate today."}'
```

已验证的部署使用 `laya-rl-agent`，返回 `answers.department`（Choice）和 `answers.refund`（Noul），以及 `usage`。这仅验证了示例任务，未证明该部署支持任意工具路由。新工具需要相应的服务端判断配置与应用映射；修改 skill 文案不会扩展接口能力。预测本身不执行工具。

## 不可用时跳过

连接最多 1 秒，请求总计最多 5 秒。连接失败、超时、HTTP 错误或无法使用的响应均直接跳过，不重试、不启动服务、不切换云端，也不阻塞其他工作。未获得预测时保留“未判定”状态。

`127.0.0.1` 指请求执行主机。沙箱或容器可能无法访问宿主机服务；应使用获准且可访问服务的执行环境。

完整规则见 [SKILL.md](SKILL.md)。

## Token 使用边界

Skill 本身不保证节省 token。智能体读取 skill、发起工具调用和解释结果都有开销；本地接口的 `usage` 不包含这些开销，`output_tokens: 0` 也不代表推理免费。

推荐在应用中实现：`业务代码 → 本地工具意图判断 → JSON 与参数校验 → 已配置工具路由`。让已支持且可直接完成的工具意图判断绕过大模型，避免随后重复判断。只有用户明确要求分级处理时才接入大模型或人工复核，升级阈值须在目标数据上验证；接口不可用仍直接跳过，不自动切换云端。

效果需使用同一批标注数据对比：所有大模型调用的输入/输出 token、工具路由准确率、升级率、未判定率和端到端延迟，另外记录本地算力成本。目前只验证了示例接口调用，尚未进行节省效果基准测试。本仓库不包含应用级路由器或评测数据。

## 来源与许可证

基于 [TypeSafe AI skills](https://github.com/typesafe-ai/skills) 的设计原则改编，为本地接口增加失败跳过规则。独立改编项目，非官方 TypeSafe 发行版。

采用 [MIT License](LICENSE)，保留上游版权声明。
