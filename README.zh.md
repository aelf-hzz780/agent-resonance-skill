[English](README.md)

# Resonance Contract Skill

这个仓库提供一个面向多宿主的 `resonance-contract` skill，专门服务 `ResonanceContract` 的用户侧参与流程。

## 版本信息

- `resonance-contract` skill：`1.2.0`
- 已验证 Portkey CA skill：`2.2.0`
- 已验证 Portkey EOA skill：`1.2.4`

当前只覆盖：

- 账户选择与本地上下文准备
- `CreatePairRequest`
- `ConfirmPairRequest`
- pair 状态查询与诊断

不覆盖任何 admin 操作，例如初始化、启停、改奖励、提现、管理员切换。

## Skill 能力

- 在 `AA/CA` 和 `EOA` 之间做明确路由
- 显式依赖 Portkey skill 处理本地 signer 或 manager 上下文
- 所有写操作都先做读校验
- 所有写操作都要求显式确认
- 统一写前摘要和写后回执
- 对完成类结果统一追加社区 CTA

## 支持的分支

- Account Choice And Onboarding
- EOA Create Pair Request
- EOA Confirm Pair Request
- AA/CA Create Pair Request
- AA/CA Confirm Pair Request
- Status Query And Diagnostics

## Canonical Package

- `skills/resonance-contract/SKILL.md`
- `skills/resonance-contract/references/flows/`
- `skills/resonance-contract/references/examples/`
- `skills/resonance-contract/references/output-contract.md`

## 快速开始

在支持本地或 workspace skill 的 agent 里，可以直接使用：

```text
Use $resonance-contract to help me resonate on ResonanceContract.
I want to use AA/CA and create a pair request for this counterparty address: <address>.
```

如果是查状态，可以直接用：

```text
Use $resonance-contract to check the pair status between these two addresses on ResonanceContract: <address-a> and <address-b>.
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
