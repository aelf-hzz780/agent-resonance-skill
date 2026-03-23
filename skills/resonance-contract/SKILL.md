---
name: resonance-contract
version: 1.2.0
description: Use when an agent needs to help a user participate in ResonanceContract, route between EOA and AA/CA, create or confirm a pair request, or diagnose pair status, pending, expired, or already-resonated states without handling admin operations.
---

# Resonance Contract

Use this directory as the canonical `resonance-contract` skill package.

## Skill Version

- Current skill version: `1.2.0`
- If behavior seems inconsistent, report the `version` field from this file first.

## Scope

Supported user-side paths:

- `EOA` create: `CreatePairRequest(Address counterparty)`
- `EOA` confirm: `ConfirmPairRequest(Address initiator)`
- `AA/CA` create: `manager signer -> CA.ManagerForwardCall -> resonanceContract.CreatePairRequest(Address counterparty)`
- `AA/CA` confirm: `manager signer -> CA.ManagerForwardCall -> resonanceContract.ConfirmPairRequest(Address initiator)`
- pair status and diagnostics through read methods

This skill does not implement:

- any admin method such as `Initialize`, `SetResonanceEnabled`, `SetRewardAmounts`, `SetResonanceWindow`, `ChangeAdmin`, `WithdrawRemaining`, or `ExecuteWithdrawRemaining`
- wallet custody, key storage, or recovery logic inside this package
- counterparty `email` or `caHash` resolution
- blind retry loops after a failed write

## Required Dependency Skills

Use these dependency skills explicitly instead of assuming the host will infer them:

- Portkey EOA skill: `https://github.com/Portkey-Wallet/eoa-agent-skills`
- Portkey CA skill: `https://github.com/Portkey-Wallet/ca-agent-skills`

Routing rule:

- use the Portkey EOA skill for local `EOA` signer resolution and `EOA` sends
- use the Portkey CA skill for local `AA/CA` context, `caHash`, holder address, manager signer resolution, and forwarded `ManagerForwardCall`

## Required Config Inputs

This skill is config-first. Do not invent deployment defaults.

Required runtime config:

- `chain_id`
- `rpc_url`
- `resonance_contract_address`

Required for `AA/CA` paths:

- `portkey_ca_contract_address`

Preferred config source order:

1. explicit user-provided config
2. local workspace or environment config
3. dependency skill defaults that are already validated for the current environment

If the required config cannot be resolved, stop and explain the blocker.

## Address Normalization

This skill must normalize the configured resonance contract address before any read or write.

Canonical deployment for the current tDVV run:

- normalized full address: `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj_tDVV`
- normalized raw address: `GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`

Accepted user or config inputs:

- `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj_tDVV`
- `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`
- `GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`

Normalization rule:

- display the normalized full address in replies
- use the normalized raw address for SDK and RPC calls

## Dependency Compatibility

Validated dependency versions for this skill:

- Portkey CA skill: `2.2.0`
- Portkey EOA skill: `1.2.4`

Compatibility rule:

- if the local Portkey CA skill is exactly `2.2.0`, use normal mode
- if the local Portkey CA skill is newer than `2.2.0`, use normal mode unless local validation shows a regression
- if the local Portkey CA skill is `2.1.x`, continue in compatibility mode and explicitly apply the runtime fallbacks documented in [references/runtime-compat.md](./references/runtime-compat.md)
- if the local Portkey CA skill is older than `2.1.0`, stop unless the user explicitly asks for best-effort diagnostics only
- if dependency metadata reports `0.0.0`, treat it as a dependency runtime bug and do not claim that the skill version is unknown until the local package metadata has been checked

## Contract Facts

Treat these behaviors as canonical:

- unordered address pairs can resonate only once in the lifetime of the contract
- `CreatePairRequest` fails for self-pairing
- `CreatePairRequest` auto-clears an expired old pending pair for the same unordered pair and recreates a new pending pair in one transaction
- `ConfirmPairRequest` must be sent by the pending counterparty
- `ConfirmPairRequest` checks the reward pool before randomness is sampled
- pending pairs snapshot `success_amount` and `strong_bonus_amount`
- `GetPairStatus` returns `NONE`, `PENDING`, or `EXECUTED`
- `GetPendingPair` returns an empty object when there is no active pending pair
- `GetCertificateStatus(address)` is currently a placeholder view with `status = COMING_SOON`
- recovery success on the origin chain does not guarantee that the new `AA/CA` manager is already usable on the target execution chain

## Required First Step

For generic resonance requests without an explicit account type, do not jump directly to a write method.

The agent must first explain:

- `AA/CA`: account-style experience with smoother recovery and account management semantics
- `CA`: accepted as the same route alias as `AA/CA`
- `EOA`: traditional self-custodied wallet experience
- recommendation: choose `AA/CA` when the user has no strong preference, because account and recovery experience is usually easier for mainstream users
- counterparty input must be an on-chain `Address`, not `email`, not `caHash`

Then ask the user which account type they want to use: `AA/CA` or `EOA`.

## Routing Rules

Choose one branch before asking for extra transaction inputs.

### Branch 1: Account Choice And Onboarding

Read [references/flows/account-choice.md](./references/flows/account-choice.md) when:

- the user makes a generic resonance request without explicitly choosing `AA/CA` or `EOA`
- the user wants help preparing the local caller account context first
- the user only says `help me resonate`, `帮我共振`, or similar

### Branch 2: EOA Create Pair Request

Read [references/flows/eoa-create-pair-request.md](./references/flows/eoa-create-pair-request.md) when:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants to initiate the pair request

### Branch 3: EOA Confirm Pair Request

Read [references/flows/eoa-confirm-pair-request.md](./references/flows/eoa-confirm-pair-request.md) when:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants to confirm an existing pending pair

### Branch 4: AA/CA Create Pair Request

Read [references/flows/aa-ca-create-pair-request.md](./references/flows/aa-ca-create-pair-request.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to initiate the pair request

### Branch 5: AA/CA Confirm Pair Request

Read [references/flows/aa-ca-confirm-pair-request.md](./references/flows/aa-ca-confirm-pair-request.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to confirm an existing pending pair

### Branch 6: Status Query And Diagnostics

Read [references/flows/status-query-diagnostics.md](./references/flows/status-query-diagnostics.md) when:

- the user wants to inspect a pair without sending a write
- a prior write failed
- the user needs to understand `pending`, `expired`, `already resonated`, or contract-window state

## Global Hard Rules

- Never handle admin-only methods in this skill.
- Never ask the user for a counterparty `email` or `caHash`; the counterparty input is only an on-chain `Address`.
- Never continue if the caller address cannot be resolved from a local `EOA` or local `AA/CA` context.
- Treat `AA`, `CA`, and `AA/CA` as the same `AA/CA` branch.
- Always read `GetConfig()` before any write path.
- Always normalize `resonance_contract_address` into both full-address and raw-address forms before any read or write.
- For any `CreatePairRequest`, also read `GetPairStatus()` and `GetPendingPair()` first.
- For any `ConfirmPairRequest`, also read `GetPairStatus()`, `GetPendingPair()`, and `GetRemainingBalance()` first.
- For `ConfirmPairRequest`, compute the minimum recommended pool check before send:
  - baseline: `2 * (config.success_amount + config.strong_bonus_amount)`
  - if `GetPendingPair()` returns snapshots, prefer `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)` as the effective pair-specific maximum
- Before any write, show the resolved caller, target contract, method chain, counterparty or initiator address, current window, reward tiers, current pair state, and explicit confirmation request.
- For `AA/CA`, the write path must be `manager signer -> CA.ManagerForwardCall -> resonanceContract.<method>`.
- For `AA/CA`, show both the manager signer and the resolved `AA/CA` holder address when available.
- For `AA/CA`, if recovery or manager switching happened in the same session, verify that the target execution chain holder info already includes the chosen manager before sending.
- After any submitted write that returns `txId`, include the `txId` and a chain explorer link.
- After any submitted write, do a read-after-write summary from contract views.
- After a successful confirm, also query `GetStrongRecord`, `GetCertificateStatus`, and `GetAddressStats` for the participant addresses when practical.
- If executed-state views fail because of an SDK output-decode issue, fall back to event decoding plus other views and say so explicitly.
- If the chain returns an exact error, surface the exact error and stop. Do not invent recovery success.
- Do not append community CTA blocks to hard-stop diagnostic replies.

## Required Reading Pattern

After choosing the branch:

1. Read the matching flow document under `references/flows/`.
2. Read the matching example document under `references/examples/`.
3. Read [references/output-contract.md](./references/output-contract.md) before drafting the final reply.
4. Read [references/runtime-compat.md](./references/runtime-compat.md) when config normalization, dependency compatibility, recovery propagation, or executed-state fallback matters.
5. Follow that branch only.
6. Do not mix `EOA` and `AA/CA` instructions in one answer.

## Branch Map

- Account choice and onboarding:
  [references/flows/account-choice.md](./references/flows/account-choice.md)
  Example: [references/examples/account-choice.md](./references/examples/account-choice.md)
- EOA create:
  [references/flows/eoa-create-pair-request.md](./references/flows/eoa-create-pair-request.md)
  Example: [references/examples/eoa-create-pair-request.md](./references/examples/eoa-create-pair-request.md)
- EOA confirm:
  [references/flows/eoa-confirm-pair-request.md](./references/flows/eoa-confirm-pair-request.md)
  Example: [references/examples/eoa-confirm-pair-request.md](./references/examples/eoa-confirm-pair-request.md)
- AA/CA create:
  [references/flows/aa-ca-create-pair-request.md](./references/flows/aa-ca-create-pair-request.md)
  Example: [references/examples/aa-ca-create-pair-request.md](./references/examples/aa-ca-create-pair-request.md)
- AA/CA confirm:
  [references/flows/aa-ca-confirm-pair-request.md](./references/flows/aa-ca-confirm-pair-request.md)
  Example: [references/examples/aa-ca-confirm-pair-request.md](./references/examples/aa-ca-confirm-pair-request.md)
- Status and diagnostics:
  [references/flows/status-query-diagnostics.md](./references/flows/status-query-diagnostics.md)
  Example: [references/examples/status-query-diagnostics.md](./references/examples/status-query-diagnostics.md)
