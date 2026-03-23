[中文](README.zh.md)

# Resonance Contract Skill

This repository packages a multi-host skill for user-side participation on `ResonanceContract`.

## Version

- `resonance-contract` skill: `1.2.0`
- validated Portkey CA skill: `2.2.0`
- validated Portkey EOA skill: `1.2.4`

It focuses only on:

- account choice and local context readiness
- `CreatePairRequest`
- `ConfirmPairRequest`
- pair status lookup and diagnostics

It does not cover admin operations such as initialization, enablement changes, reward updates, withdrawals, or admin transfer.

## What This Skill Does

- routes the user between `AA/CA` and `EOA`
- uses explicit Portkey dependency skills for local signer or manager resolution
- enforces preflight reads before writes
- requires an explicit write confirmation before sending
- returns a consistent pre-send summary and post-send receipt
- appends community CTAs for completed user-side results

## Supported Branches

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

## Quick Start

Use this prompt with an agent that supports workspace or local skills:

```text
Use $resonance-contract to help me resonate on ResonanceContract.
I want to use AA/CA and create a pair request for this counterparty address: <address>.
```

For a status lookup:

```text
Use $resonance-contract to check the pair status between these two addresses on ResonanceContract: <address-a> and <address-b>.
```

## Host Layout

- Codex / OpenAI / OpenClaw: `skills/resonance-contract/`
- Claude Code: `.claude/skills/resonance-contract/SKILL.md`
- OpenCode: `.opencode/skills/resonance-contract/SKILL.md`
- Cursor: `.cursor/rules/resonance-contract.mdc`

## Repository Structure

- `skills/resonance-contract/`: canonical skill package
- `.claude/skills/resonance-contract/`: Claude wrapper
- `.opencode/skills/resonance-contract/`: OpenCode wrapper
- `.cursor/rules/resonance-contract.mdc`: Cursor rule wrapper
- `AGENTS.md`: workspace routing hints
