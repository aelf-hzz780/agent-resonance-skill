# Resonance Contract Workspace Instructions

This repository packages the `resonance-contract` skill for multiple agent hosts.

## Canonical Sources

- `skills/resonance-contract/SKILL.md`: canonical skill entrypoint
- `skills/resonance-contract/references/flows/`: canonical workflow branches
- `skills/resonance-contract/references/examples/`: branch examples
- `skills/resonance-contract/references/output-contract.md`: canonical reply contract

## When To Apply

Apply this workflow when the user asks for:

- resonance pairing on `ResonanceContract`
- `EOA` or `AA/CA` account routing for user-side resonance participation
- `CreatePairRequest`
- `ConfirmPairRequest`
- `JoinPairQueue`
- `LeavePairQueue`
- pair status lookup, queue status lookup, pending or expired diagnosis, already-resonated diagnosis, warmup diagnosis, or reward-balance diagnosis

## Operating Rule

Load `skills/resonance-contract/SKILL.md` and follow the canonical package exactly.
Do not re-derive the workflow from this `AGENTS.md`.

## Compatibility Layout

- Codex / OpenAI / OpenClaw: `skills/resonance-contract/`
- Claude Code: `.claude/skills/resonance-contract/SKILL.md`
- OpenCode: `.opencode/skills/resonance-contract/SKILL.md`
- Cursor: `.cursor/rules/resonance-contract.mdc` and this root `AGENTS.md`
