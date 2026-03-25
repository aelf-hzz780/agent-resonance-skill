[中文](README.zh.md)

# Resonance Contract Skill

This repository packages a multi-host skill for user-side participation on `ResonanceContract`.

## Version

- `resonance-contract` skill: `2.1.1`
- validated Portkey CA skill: `2.2.0`
- validated Portkey EOA skill: `1.2.4`

It focuses only on:

- account choice and local context readiness
- `CreatePairRequest`
- `ConfirmPairRequest`
- `JoinPairQueue`
- `LeavePairQueue`
- pair, queue, warmup, and reward-balance diagnostics

It does not cover admin operations such as initialization, enablement changes, reward updates, withdrawals, or admin transfer.

## What This Skill Does

- routes the user between `AA/CA` and `EOA`
- routes the user between direct pair and automatic queue participation
- uses explicit Portkey dependency skills for local signer or manager resolution
- enforces preflight reads before writes
- routes all resonance `Get*` and other view-only methods through the appropriate view path rather than `AA/CA` forwarded writes or `EOA` send paths
- requires an explicit write confirmation before sending
- returns summary-first user-facing replies for both pre-send and post-send stages
- keeps `skill_version` and `dependency_versions` visible in the default user-facing layer
- moves `Technical Details` behind explicit requests such as `expand details`, `debug`, or `show raw data`
- explains `VirtualTransactionCreated` only as forwarded-write evidence rather than a view return payload or standalone business-success proof
- explains queue timeout, matching policy, queue-full behavior, and warmup windows in plain language
- appends community CTAs for completed user-side results

## Supported Branches

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

## Quick Start

This skill now defaults to the current canonical `tDVV` deployment:

- raw address: `28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
- full address: `ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`

Only provide `resonance_contract_address` explicitly when you want to override that default deployment.

Use this prompt with an agent that supports workspace or local skills:

```text
Use $resonance-contract to help me resonate on ResonanceContract.
I want to use AA/CA and create a pair request for this counterparty address: <address>.
```

For queue participation:

```text
Use $resonance-contract to help me join the resonance queue on ResonanceContract.
I want to use AA/CA, I do not have a counterparty address, and I want the default queue mode.
```

For a status lookup:

```text
Use $resonance-contract to check the pair status between these two addresses on ResonanceContract: <address-a> and <address-b>.
```

For queue or warmup diagnostics:

```text
Use $resonance-contract to check whether this address is still in the queue or blocked by warmup on ResonanceContract: <address>.
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
