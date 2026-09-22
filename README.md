# Laya AI Skill

面向 Codex 等支持 Agent Skills 的智能体：调用本地 Laya AI 服务，将文本转换为结构化语义判断。

## 安装

将本仓库克隆到全局 skills 目录（目标目录不存在时）：

```bash
git clone https://github.com/vincent1986/laya-ai.git "${CODEX_HOME:-$HOME/.codex}/skills/laya-ai"
```

在支持的客户端中通过 `$laya-ai` 调用，或根据任务自动匹配。

## 接口

需自行提供运行在本机的预测服务。本仓库只包含 skill，不包含模型权重或服务端实现。

```bash
curl --silent --show-error --fail --connect-timeout 1 --max-time 5 \
  http://127.0.0.1:8000/predict \
  -H 'Content-Type: application/json' \
  --data-binary '{"state":"I was billed twice. Please refund the duplicate today."}'
```

已验证的部署使用 `laya-rl-agent`，返回 `answers.department`（Choice）和 `answers.refund`（Noul），以及 `usage`。问题和响应取决于服务端配置；预测不执行退款等业务动作。

## 不可用时跳过

连接最多 1 秒，请求总计最多 5 秒。连接失败、超时、HTTP 错误或无法使用的响应均直接跳过，不重试、不启动服务、不切换云端，也不阻塞其他工作。未获得预测时保留“未判定”状态。

`127.0.0.1` 指请求执行主机。沙箱或容器可能无法访问宿主机服务；应使用获准且可访问服务的执行环境。

完整规则见 [SKILL.md](SKILL.md)。

## 来源与许可证

基于 [TypeSafe AI skills](https://github.com/typesafe-ai/skills) 的设计原则改编，为本地接口增加失败跳过规则。独立改编项目，非官方 TypeSafe 发行版。

采用 [MIT License](LICENSE)，保留上游版权声明。
