[中文](README.zh.md)

# Resonance Contract Skill

This repository packages a multi-host skill for user-side participation on `ResonanceContract`.

## Version

- `resonance-contract` skill: `3.0.1`
- validated Portkey CA skill: `2.3.0`
- compatible contract version: `2.0.0`

It focuses only on:

- local CA context readiness
- `CreatePairRequestByCa`
- `ConfirmPairRequestByCa`
- `JoinPairQueueByCa`
- `LeavePairQueueByCa`
- pair, queue, warmup, and reward-balance diagnostics

It does not cover admin operations such as initialization, enablement changes, reward updates, Portkey CA contract updates, withdrawals, or admin transfer.

## What This Skill Does

- routes the user through the CA-only participation model
- accepts `AA`, `CA`, and `AA/CA` as input aliases but renders `CA` as the canonical user-facing term
- asks which local CA account to use when multiple CA accounts are available
- routes the user between direct pair and automatic queue participation
- uses explicit Portkey CA dependency skill for local CA readiness
- treats missing local sign-in as an onboarding step first, using first-time setup or returning-user recovery sign-in before queue is considered blocked
- treats queue as the formal executable automatic-matching path once the local CA account and dependency signer or relayer are ready
- enforces preflight reads before writes
- routes all resonance `Get*` and other view-only methods through the direct view path
- treats `ca_hash` as the write-side identity input and `ca_address` as the read-side state key
- requires an explicit write confirmation before sending
- returns summary-first user-facing replies for both pre-send and post-send stages
- keeps `skill_version` and `dependency_versions` visible in the default user-facing layer
- moves `Technical Details` behind explicit requests such as `expand details`, `debug`, or `show raw data`
- explains that legacy `EOA` or `ManagerForwardCall` receipts belong to the pre-`v2.0.0` contract path
- does not treat the current validated Portkey relay transport as a reason to abandon an otherwise executable CA-only queue write
- explains queue timeout, matching policy, queue-full behavior, Portkey CA config blockers, and warmup windows in plain language
- appends success CTAs for clear completed results, support CTAs only when the user is genuinely stuck and the agent cannot continue automatically, and no CTA when the issue is only invalid input, missing required input, or a light route correction the agent can fix immediately

## Supported Branches

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

Legacy pre-CA-only material is preserved under:

- `skills/resonance-contract/references/deprecated/v2-pre-ca-only/`

## Quick Start

This skill now defaults to the current canonical `tDVV` deployment:

- raw address: `RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
- full address: `ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`
- expected Portkey CA contract on `tDVV`: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`

Only provide `resonance_contract_address` explicitly when you want to override that default deployment.
Current writes still hard-check `GetConfig.portkey_ca_contract_address`; if the live contract has not configured it, CA writes will stop before send.

Use this prompt with an agent that supports workspace or local skills:

```text
Use $resonance-contract to help me resonate on ResonanceContract.
I want to use CA and create a pair request for this counterparty ca_hash: <counterparty-ca-hash>.
```

For queue participation:

```text
Use $resonance-contract to help me join the resonance queue on ResonanceContract.
I want to use CA, I do not have a counterparty ca_hash, and I want the default queue mode.
```

## Migration Note

- old wording such as `there is no direct tool, so go find someone on X or Telegram first` is now deprecated
- new wording is `queue is the primary path when Portkey CA preflight passes; social channels are only fallback for real blockers`

If the local machine has multiple CA accounts, tell the agent which local CA account should be used for this run.

For a status lookup by `ca_hash`:

```text
Use $resonance-contract to check whether this ca_hash is still queued or has an active pending pair on ResonanceContract: <ca-hash>.
```

For a status lookup by resolved `ca_address`:

```text
Use $resonance-contract to check the pair or queue status on ResonanceContract for this ca_address or pair of ca_address values: <ca-address>.
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
