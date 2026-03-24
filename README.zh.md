[English](README.md)

# Resonance Contract Skill

这个仓库提供一个面向多宿主的 `resonance-contract` skill，专门服务 `ResonanceContract` 的用户侧参与流程。

## 版本信息

- `resonance-contract` skill：`2.1.0`
- 已验证 Portkey CA skill：`2.2.0`
- 已验证 Portkey EOA skill：`1.2.4`

当前只覆盖：

- 账户选择与本地上下文准备
- `CreatePairRequest`
- `ConfirmPairRequest`
- `JoinPairQueue`
- `LeavePairQueue`
- pair、queue、warmup 和 reward-balance 查询与诊断

不覆盖任何 admin 操作，例如初始化、启停、改奖励、提现、管理员切换。

## Skill 能力

- 在 `AA/CA` 和 `EOA` 之间做明确路由
- 在 `direct pair` 和 `queue` 两种参与模式之间做明确路由
- 显式依赖 Portkey skill 处理本地 signer 或 manager 上下文
- 所有写操作都先做读校验
- 所有写操作都要求显式确认
- 默认先给普通用户摘要，再给写前摘要和写后回执的关键锚点
- 默认层会展示 `skill_version` 和 `dependency_versions`
- `Technical Details` 只在用户明确说“展开详情 / debug / 看链上参数”时再完整展开
- 会把排队超时、默认撮合规则、满队列处理方式和升级冷却期解释成普通用户能看懂的话
- 对完成类结果统一追加社区 CTA

## 支持的分支

- Account Choice And Participation Mode
- EOA Create Pair Request
- EOA Confirm Pair Request
- EOA Join Pair Queue
- EOA Leave Pair Queue
- AA/CA Create Pair Request
- AA/CA Confirm Pair Request
- AA/CA Join Pair Queue
- AA/CA Leave Pair Queue
- Status Query And Diagnostics

## Canonical Package

- `skills/resonance-contract/SKILL.md`
- `skills/resonance-contract/references/flows/`
- `skills/resonance-contract/references/examples/`
- `skills/resonance-contract/references/output-contract.md`

## 快速开始

这个 skill 现在默认使用当前 `tDVV` 的 canonical deployment：

- raw 地址：`28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
- full 地址：`ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`

只有在你想覆盖这个默认部署时，才需要显式提供 `resonance_contract_address`。

在支持本地或 workspace skill 的 agent 里，可以直接使用：

```text
使用 $resonance-contract 帮我在 ResonanceContract 上发起共振。
我想用 AA/CA，并给这个对手地址发起配对请求：<address>。
```

如果是排队找人，可以直接用：

```text
使用 $resonance-contract 帮我加入 ResonanceContract 的共振队列。
我想用 AA/CA，我现在没有对手地址，并且想走默认排队模式。
```

如果是查状态，可以直接用：

```text
使用 $resonance-contract 帮我查询 ResonanceContract 上这两个地址之间的配对状态：<address-a> 和 <address-b>。
```

如果是查排队或冷却期状态，可以直接用：

```text
使用 $resonance-contract 帮我查询这个地址在 ResonanceContract 上是否仍在队列里，或者是否还被升级冷却期（warmup）拦住：<address>。
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
