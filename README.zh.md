[English](README.md)

# Resonance Contract Skill

这个仓库提供一个面向多宿主的 `resonance-contract` skill，专门服务 `ResonanceContract` 的用户侧参与流程。

## 版本信息

- `resonance-contract` skill：`3.0.0`
- 已验证 Portkey CA skill：`2.3.0`
- 兼容合约版本：`2.0.0`

当前只覆盖：

- 本地 CA 上下文准备
- `CreatePairRequestByCa`
- `ConfirmPairRequestByCa`
- `JoinPairQueueByCa`
- `LeavePairQueueByCa`
- pair、queue、warmup 和 reward-balance 查询与诊断

不覆盖任何 admin 操作，例如初始化、启停、改奖励、改 Portkey CA contract、提现、管理员切换。

## Skill 能力

- 使用 CA-only 的用户参与模型
- 接受 `AA`、`CA`、`AA/CA` 作为输入别名，但 canonical 用户文案统一写 `CA`
- 如果本地有多个 CA 账号，会先确认当前要使用哪一个
- 在 `direct pair` 和 `queue` 两种参与模式之间做明确路由
- 显式依赖 Portkey CA skill 处理本地 CA 上下文准备
- 所有写操作都先做读校验
- 所有 resonance `Get*` 和其它 view-only 方法都强制走 direct view path
- 把 `ca_hash` 作为写侧身份输入，把 `ca_address` 作为读侧状态键
- 所有写操作都要求显式确认
- 默认先给普通用户摘要，并在写前和写后阶段给出关键锚点
- 默认层会展示 `skill_version` 和 `dependency_versions`
- `Technical Details` 只在用户明确说“展开详情 / debug / 看链上参数”时再完整展开
- 会把旧版 `EOA` 或 `ManagerForwardCall` 回执解释为 pre-`v2.0.0` 的遗留路径，而不是当前合约语义
- 会把排队超时、默认撮合规则、满队列处理方式、Portkey CA 配置阻塞和升级冷却期解释成普通用户能看懂的话
- 对明确完成结果追加 success CTA，只在用户确实卡住、agent 当前无法自动继续时追加 support CTA；如果只是无效输入、缺少必需输入，或 agent 还能立即修正的轻量路由问题，则不追加 CTA

## 支持的分支

- Account Choice And Participation Mode
- CA Create Pair Request
- CA Confirm Pair Request
- CA Join Pair Queue
- CA Leave Pair Queue
- Status Query And Diagnostics

## Canonical Package

- `skills/resonance-contract/SKILL.md`
- `skills/resonance-contract/references/flows/`
- `skills/resonance-contract/references/examples/`
- `skills/resonance-contract/references/output-contract.md`

旧版 pre-CA-only 材料保存在：

- `skills/resonance-contract/references/deprecated/v2-pre-ca-only/`

## 快速开始

这个 skill 现在默认使用当前 `tDVV` 的 canonical deployment：

- raw 地址：`RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
- full 地址：`ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`
- 预期 Portkey CA contract：`2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`

只有在你想覆盖这个默认部署时，才需要显式提供 `resonance_contract_address`。
当前写路径仍会硬检查 `GetConfig.portkey_ca_contract_address`；如果链上还没配置好，CA 写操作会在发送前直接停下。

在支持本地或 workspace skill 的 agent 里，可以直接使用：

```text
使用 $resonance-contract 帮我在 ResonanceContract 上发起共振。
我想用 CA，并给这个对手方 ca_hash 发起配对请求：<counterparty-ca-hash>。
```

如果是排队找人，可以直接用：

```text
使用 $resonance-contract 帮我加入 ResonanceContract 的共振队列。
我想用 CA，我现在没有对手方 ca_hash，并且想走默认排队模式。
```

如果本地机器上有多个 CA 账号，记得明确告诉 agent 这次要用哪个本地 CA。

如果是按 `ca_hash` 查状态，可以直接用：

```text
使用 $resonance-contract 帮我查询这个 ca_hash 在 ResonanceContract 上是否仍在队列里，或者是否仍有 active pending pair：<ca-hash>。
```

如果是按解析后的 `ca_address` 查状态，可以直接用：

```text
使用 $resonance-contract 帮我查询这个 ca_address，或者这组 ca_address，在 ResonanceContract 上的配对或排队状态：<ca-address>。
```

## 宿主布局

- Codex / OpenAI / OpenClaw: `skills/resonance-contract/`
- Claude Code: `.claude/skills/resonance-contract/SKILL.md`
- OpenCode: `.opencode/skills/resonance-contract/SKILL.md`
- Cursor: `.cursor/rules/resonance-contract.mdc`

## 仓库结构

- `skills/resonance-contract/`: canonical skill package
- `.claude/skills/resonance-contract/`: Claude wrapper
- `.opencode/skills/resonance-contract/`: OpenCode wrapper
- `.cursor/rules/resonance-contract.mdc`: Cursor rule wrapper
- `AGENTS.md`: workspace 路由说明
