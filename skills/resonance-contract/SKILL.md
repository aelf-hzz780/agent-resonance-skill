---
name: resonance-contract
version: 2.1.0
description: Use when an agent needs to help a user participate in ResonanceContract through direct pair or automatic queue flows, route between EOA and AA/CA, create, confirm, join queue, leave queue, or diagnose pair, queue, warmup, or reward-balance state without handling admin operations.
---

# Resonance Contract

Use this directory as the canonical `resonance-contract` skill package.

## Skill Version

- Current skill version: `2.1.0`
- If behavior seems inconsistent, report the `version` field from this file first.

## Scope

Supported user-side paths:

- `EOA` create: `CreatePairRequest(Address counterparty)`
- `EOA` confirm: `ConfirmPairRequest(Address initiator)`
- `EOA` queue join: `JoinPairQueue(JoinPairQueueInput)`
- `EOA` queue leave: `LeavePairQueue()`
- `AA/CA` create: `manager signer -> CA.ManagerForwardCall -> resonanceContract.CreatePairRequest(Address counterparty)`
- `AA/CA` confirm: `manager signer -> CA.ManagerForwardCall -> resonanceContract.ConfirmPairRequest(Address initiator)`
- `AA/CA` queue join: `manager signer -> CA.ManagerForwardCall -> resonanceContract.JoinPairQueue(JoinPairQueueInput)`
- `AA/CA` queue leave: `manager signer -> CA.ManagerForwardCall -> resonanceContract.LeavePairQueue()`
- pair, queue, warmup, and reward-balance diagnostics through read methods

This skill does not implement:

- any admin write such as `Initialize`, `FinalizeParticipationUpgrade`, `SetResonanceEnabled`, `SetResonanceWindow`, `SetRewardAmounts`, `SetPairQueueCapacity`, `ChangeAdmin`, `AcceptAdmin`, `WithdrawRemaining`, or `ExecuteWithdrawRemaining`
- wallet custody, key storage, or recovery logic inside this package
- direct-mode counterparty `email` or `caHash` resolution
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
- `resonance_contract_address` when the caller needs to override the default deployment or when the runtime is not the validated `tDVV` environment below

Required for `AA/CA` paths:

- `portkey_ca_contract_address`

Built-in canonical deployment for the current validated runtime:

- `chain_id`: `tDVV`
- raw `resonance_contract_address`: `28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
- normalized full `resonance_contract_address`: `ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`

Preferred config source order:

1. explicit user-provided config
2. local workspace or environment config
3. built-in canonical deployment for the validated `tDVV` runtime
4. dependency skill defaults that are already validated for the current environment

If the required config cannot be resolved, stop and explain the blocker.

## Address Normalization

This skill must normalize the configured resonance contract address before any read or write.

Default deployment rule:

- for the current validated `tDVV` runtime, use the built-in canonical deployment by default:
  - raw: `28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
  - full: `ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`
- if the user or config explicitly supplies `resonance_contract_address`, normalize and use the supplied value instead
- if the runtime is not the validated `tDVV` environment and no explicit `resonance_contract_address` can be resolved, stop and explain the blocker

Accepted user or config inputs:

- `<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>_<chain_id>`

Normalization rule:

- display the normalized full address in replies
- use the normalized raw address for SDK and RPC calls
- if the user or config supplies a full address with a suffix that does not match the runtime `chain_id`, stop and explain the mismatch

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
- `CreatePairRequest` and `JoinPairQueue` both require resonance to be enabled, the window to be open, and current time to be at or after `new_participation_available_time`
- `CreatePairRequest` auto-clears an expired old pending pair for the same unordered pair and recreates a new pending pair in one transaction
- `ConfirmPairRequest` must be sent by the pending counterparty
- `ConfirmPairRequest` checks the raw reward pool before randomness is sampled
- pending pairs snapshot `success_amount` and `strong_bonus_amount`
- pending pairs also snapshot `window_end_time`
- `JoinPairQueue` defaults to `FIFO` when `selection_policy` is omitted or default input is used
- `JoinPairQueue` also supports explicit `RANDOM` selection
- queue entries expire after `request_expire_seconds` and are also bounded by the snapped `window_end_time`
- one address can hold only one active matchmaking state at a time: either one direct pending pair or one active queue entry
- new direct pending pairs reserve their maximum reward exposure up front
- `JoinPairQueue` has two balance paths:
  - immediate match checks raw `remaining balance`
  - new queue entry checks `available balance` after reservations
- `GetRewardBalance()` returns `total_balance`, `reserved_balance`, and `available_balance`
- `GetAvailableRewardBalance()` returns the currently available reward balance after reservations
- `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` are the authoritative address-scoped read models for direct-vs-queue exclusivity
- `GetPairQueueStats()` returns the current active queue length and current `queue_capacity`
- when a new `JoinPairQueue` call cannot find a match and the queue is already full after expired-entry cleanup, the earliest still-valid queue entry is removed before the new caller joins
- `queue_capacity` defaults to `1000` when initialization leaves it as `0`, but later reads must always trust the current on-chain config
- `GetPairStatus` returns `NONE`, `PENDING`, or `EXECUTED`
- `GetPendingPair` returns an empty object when there is no active pending pair
- `GetCertificateStatus(address)` is still a placeholder view with `status = COMING_SOON`, but when the address already has a strong-resonance record it also returns the current strong payload for display
- recovery success on the origin chain does not guarantee that the new `AA/CA` manager is already usable on the target execution chain

## Required First Step

For generic resonance requests without an explicit account type or participation mode, do not jump directly to a write method.

The agent must first explain:

- `AA/CA`: account-style experience with smoother recovery and account management semantics
- `CA`: accepted as the same route alias as `AA/CA`
- `EOA`: traditional self-custodied wallet experience
- recommendation: choose `AA/CA` when the user has no strong preference, because account and recovery experience is usually easier for mainstream users
- `direct pair`: use this when the user already knows the other side's on-chain `Address`
- `queue`: use this when the user does not know a counterparty and wants automatic matching instead
- direct-mode counterparty input must be an on-chain `Address`, not `email`, not `caHash`
- queue entries are not permanent; the timeout comes from `GetConfig.request_expire_seconds` and must be explained in plain language before sending
- queue default matching is `FIFO`; when the queue is full and the new caller still does not match immediately, the earliest still-valid queued address may be removed before the new caller joins

Then:

- ask the user which account type they want to use: `AA/CA` or `EOA`
- if the user did not already provide a direct counterparty address or explicitly ask for queue, also ask whether they want `direct pair` or `queue`

## Routing Rules

Choose one branch before asking for extra transaction inputs.

### Branch 1: Account Choice And Participation Mode

Read [references/flows/account-choice.md](./references/flows/account-choice.md) when:

- the user makes a generic resonance request without explicitly choosing `AA/CA` or `EOA`
- the user has not yet chosen between `direct pair` and `queue`
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

### Branch 4: EOA Join Pair Queue

Read [references/flows/eoa-join-pair-queue.md](./references/flows/eoa-join-pair-queue.md) when:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants automatic matching or queue participation

### Branch 5: EOA Leave Pair Queue

Read [references/flows/eoa-leave-pair-queue.md](./references/flows/eoa-leave-pair-queue.md) when:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants to leave the automatic matching queue

### Branch 6: AA/CA Create Pair Request

Read [references/flows/aa-ca-create-pair-request.md](./references/flows/aa-ca-create-pair-request.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to initiate the pair request

### Branch 7: AA/CA Confirm Pair Request

Read [references/flows/aa-ca-confirm-pair-request.md](./references/flows/aa-ca-confirm-pair-request.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to confirm an existing pending pair

### Branch 8: AA/CA Join Pair Queue

Read [references/flows/aa-ca-join-pair-queue.md](./references/flows/aa-ca-join-pair-queue.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants automatic matching or queue participation

### Branch 9: AA/CA Leave Pair Queue

Read [references/flows/aa-ca-leave-pair-queue.md](./references/flows/aa-ca-leave-pair-queue.md) when:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to leave the automatic matching queue

### Branch 10: Status Query And Diagnostics

Read [references/flows/status-query-diagnostics.md](./references/flows/status-query-diagnostics.md) when:

- the user wants to inspect a pair without sending a write
- the user wants to inspect whether an address is in queue or still has an active pending pair
- a prior write failed
- the user needs to understand `pending`, `expired`, `already resonated`, `queue`, `warmup`, or reward-balance state

## Global Hard Rules

- Never handle admin-only methods in this skill.
- Never ask the user for a direct-mode counterparty `email` or `caHash`; direct mode only accepts an on-chain `Address`.
- Never continue if the caller address cannot be resolved from a local `EOA` or local `AA/CA` context.
- Treat `AA`, `CA`, and `AA/CA` as the same `AA/CA` branch.
- Do not treat a successful browser `GET` to `rpc_url` as proof that JSON-RPC is healthy; AElf SDK reads and writes still use HTTP `POST` against the same base endpoint.
- When RPC requests hang or fail, do not jump directly to a root-cause claim such as `VPN`, `router`, `SSL`, or `certificate` unless the evidence clearly isolates that cause.
- For any write reply or diagnostics-only reply, render a user-summary layer first and keep `Technical Details` behind explicit user intent such as `展开详情`, `debug`, `看链上参数`, `technical details`, or `show raw data`.
- Keep `skill_version` and `dependency_versions` visible in the default user-facing layer.
- Keep `dependency_mode` in `Technical Details` unless the dependency is in compatibility mode or dependency runtime metadata itself is unreliable.
- Do not expose internal branch names such as `AA/CA Join Pair Queue` or `Status Query And Diagnostics` in the default user-facing layer unless the user explicitly asks for debug or Technical Details.
- If preflight says a write is blocked, stop with a plain-language blocker summary and next step; do not ask the user to confirm a send that cannot proceed.
- Always read `GetConfig()` before any write path.
- Always normalize `resonance_contract_address` into both full-address and raw-address forms before any read or write.
- For any `CreatePairRequest`, also read `GetPairStatus()`, `GetPendingPair()`, `GetActivePendingPair()` for caller and counterparty, `GetPairQueueStatus()` for caller and counterparty, and `GetRewardBalance()` or `GetAvailableRewardBalance()` first.
- For any `ConfirmPairRequest`, also read `GetPairStatus()`, `GetPendingPair()`, and `GetRemainingBalance()` first.
- For any `JoinPairQueue`, also read `GetActivePendingPair()` for the caller, `GetPairQueueStatus()` for the caller, `GetRemainingBalance()`, `GetRewardBalance()` or `GetAvailableRewardBalance()`, and `GetPairQueueStats()` when practical.
- For any `LeavePairQueue`, also read `GetPairQueueStatus()` for the caller first.
- For `ConfirmPairRequest`, compute the minimum recommended pool check before send:
  - baseline: `2 * (config.success_amount + config.strong_bonus_amount)`
  - if `GetPendingPair()` returns snapshots, prefer `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)` as the effective pair-specific maximum
- For `CreatePairRequest` and `JoinPairQueue`, if `GetConfig().new_participation_available_time` is missing on an otherwise initialized contract, treat it as an abnormal state or decode issue first; only explain it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
- For `CreatePairRequest` and `JoinPairQueue`, if `GetConfig().new_participation_available_time` is in the future, stop and explain the warmup window in plain language.
- Before any write, show a summary-first pre-send reply:
  - localized user-summary layer: operation, caller, target contract address, whether the write can proceed, expiry or timeout, the most important success condition or blocker, and an explicit confirmation request
  - localized technical-details layer: resolved caller, target contract, method chain, current window, queue timeout, reward tiers, current pair or queue state, and the relevant balance model
- For `AA/CA`, the write path must be `manager signer -> CA.ManagerForwardCall -> resonanceContract.<method>`.
- For `AA/CA`, show both the manager signer and the resolved `AA/CA` holder address when available.
- For `AA/CA`, if recovery or manager switching happened in the same session, verify that the target execution chain holder info already includes the chosen manager before sending.
- After any submitted write that returns `txId`, include the `txId` and a chain explorer link.
- After any submitted write, do a summary-first read-after-write reply:
  - keep the default layer focused on result, `txId`, explorer link, key outcome fields, and the next practical meaning for the user
  - keep the full contract-view snapshot in `Technical Details`
- After any queue write, include plain-language explanation for timeout, default `FIFO`, explicit `RANDOM`, queue-full eviction behavior, and the direct-vs-queue exclusivity rule when relevant.
- After a successful confirm, also query `GetStrongRecord`, `GetCertificateStatus`, and `GetAddressStats` for the participant addresses when practical.
- When `GetCertificateStatus` shows `COMING_SOON` but also returns strong payload, explain that certificate issuance is not open yet but the strong record already exists.
- If executed-state views fail because of an SDK output-decode issue, fall back to event decoding plus other views and say so explicitly.
- If the chain returns an exact error, surface the exact error and stop. Do not invent recovery success.
- Do not append community CTA blocks to hard-stop diagnostic replies.

## Required Reading Pattern

After choosing the branch:

1. Read the matching flow document under `references/flows/`.
2. Read the matching example document under `references/examples/`.
3. Read [references/output-contract.md](./references/output-contract.md) before drafting the final reply.
4. Read [references/runtime-compat.md](./references/runtime-compat.md) when config normalization, dependency compatibility, queue semantics, balance reservation, recovery propagation, warmup, or executed-state fallback matters.
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
- EOA queue join:
  [references/flows/eoa-join-pair-queue.md](./references/flows/eoa-join-pair-queue.md)
  Example: [references/examples/eoa-join-pair-queue.md](./references/examples/eoa-join-pair-queue.md)
- EOA queue leave:
  [references/flows/eoa-leave-pair-queue.md](./references/flows/eoa-leave-pair-queue.md)
  Example: [references/examples/eoa-leave-pair-queue.md](./references/examples/eoa-leave-pair-queue.md)
- AA/CA create:
  [references/flows/aa-ca-create-pair-request.md](./references/flows/aa-ca-create-pair-request.md)
  Example: [references/examples/aa-ca-create-pair-request.md](./references/examples/aa-ca-create-pair-request.md)
- AA/CA confirm:
  [references/flows/aa-ca-confirm-pair-request.md](./references/flows/aa-ca-confirm-pair-request.md)
  Example: [references/examples/aa-ca-confirm-pair-request.md](./references/examples/aa-ca-confirm-pair-request.md)
- AA/CA queue join:
  [references/flows/aa-ca-join-pair-queue.md](./references/flows/aa-ca-join-pair-queue.md)
  Example: [references/examples/aa-ca-join-pair-queue.md](./references/examples/aa-ca-join-pair-queue.md)
- AA/CA queue leave:
  [references/flows/aa-ca-leave-pair-queue.md](./references/flows/aa-ca-leave-pair-queue.md)
  Example: [references/examples/aa-ca-leave-pair-queue.md](./references/examples/aa-ca-leave-pair-queue.md)
- Status and diagnostics:
  [references/flows/status-query-diagnostics.md](./references/flows/status-query-diagnostics.md)
  Example: [references/examples/status-query-diagnostics.md](./references/examples/status-query-diagnostics.md)
