---
name: resonance-contract
version: 4.0.0
description: Use when an agent needs to help a user participate in ResonanceContract through CA-only direct pair or automatic queue flows, create, confirm, join queue, leave queue, or diagnose pair, queue, warmup, or reward-balance state without handling admin operations.
---

# Resonance Contract

Use this directory as the canonical `resonance-contract` skill package.

## Skill Version

- Current skill version: `4.0.0`
- If behavior seems inconsistent, report the `version` field from this file first.

## Scope

Supported user-side paths on `ResonanceContract v2.0.0`:

- `CA` create: `CreatePairRequestByCa(CreatePairRequestByCaInput)`
- `CA` confirm: `ConfirmPairRequestByCa(ConfirmPairRequestByCaInput)`
- `CA` queue join: `JoinPairQueueByCa(JoinPairQueueByCaInput)`
- `CA` queue leave: `LeavePairQueueByCa(LeavePairQueueByCaInput)`
- pair, queue, warmup, and reward-balance diagnostics through direct view methods

This skill does not implement:

- any admin write such as `Initialize`, `FinalizeParticipationUpgrade`, `SetResonanceEnabled`, `SetResonanceWindow`, `SetRewardAmounts`, `SetPairQueueCapacity`, `SetPortkeyCaContract`, `ChangeAdmin`, `AcceptAdmin`, `WithdrawRemaining`, or `ExecuteWithdrawRemaining`
- wallet custody, key storage, or recovery logic inside this package
- any user-side route outside the current CA-only `*ByCa` contract methods
- blind retry loops after a failed write

## Required Dependency Skill

Use the Portkey CA skill explicitly instead of assuming the host will infer it:

- Portkey CA skill: `https://github.com/Portkey-Wallet/ca-agent-skills`

Routing rule:

- use the Portkey CA skill for local `CA` account readiness and local relayer/signer context when needed
- if multiple local `CA` accounts are available and the caller is not already implied, ask which local `CA` account to use before entering a write branch
- never present any underlying relay transport as the business method or as a user-facing route choice; when the validated Portkey CA dependency uses relay transport underneath a CA-only write, that transport detail is allowed but remains implementation detail

## Required Config Inputs

This skill is config-first.

Required runtime config:

- `chain_id`
- `rpc_url`
- `resonance_contract_address` when the caller needs to override the default deployment or when the runtime is not the validated `tDVV` environment below

Built-in canonical deployment for the current validated runtime:

- `chain_id`: `tDVV`
- raw `resonance_contract_address`: `RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
- normalized full `resonance_contract_address`: `ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`

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
  - raw: `RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
  - full: `ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`
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

- Portkey CA skill: `2.3.0`

Compatibility rule:

- if the local Portkey CA skill is exactly `2.3.0`, use normal mode
- if the local Portkey CA skill is newer than `2.3.0`, use normal mode unless local validation shows a regression
- if the local Portkey CA skill is older than `2.3.0`, continue only in compatibility mode and explain that the dependency line is not fully validated for the CA-only contract
- if dependency metadata reports `0.0.0`, treat it as a dependency runtime bug and do not claim that the skill version is unknown until the local package metadata has been checked

## Contract Facts

Treat these behaviors as canonical for `ResonanceContract v2.0.0`:

- the contract is now CA-only for user-side writes
- all user-side write methods accept `ca_hash`
- the contract resolves each `ca_hash` into a `ca_address` through the configured Portkey CA contract
- pair, queue, stats, strong record, and certificate placeholder state are all keyed by resolved `ca_address`
- `Context.Sender` is the relayer or operator that paid for the transaction, not the business identity itself
- a validated Portkey CA relay transport may still sit underneath the current CA-only write path; that transport detail does not change the business method, which remains `CreatePairRequestByCa`, `ConfirmPairRequestByCa`, `JoinPairQueueByCa`, or `LeavePairQueueByCa`
- the contract does not require manager visibility and does not validate `manager_infos`
- unordered pairs can resonate only once in the lifetime of the contract
- `CreatePairRequestByCa` fails for self-pairing after both sides resolve to the same `ca_address`
- `CreatePairRequestByCa` and `JoinPairQueueByCa` both require resonance to be enabled, the window to be open, and current time to be at or after `new_participation_available_time`
- `CreatePairRequestByCa` auto-clears an expired old pending pair for the same unordered pair and recreates a new pending pair in one transaction
- `ConfirmPairRequestByCa` checks the raw reward pool before randomness is sampled
- pending pairs snapshot `success_amount`, `strong_bonus_amount`, and `window_end_time`
- `JoinPairQueueByCa` defaults to `FIFO` when `selection_policy` is omitted or default input is used
- `JoinPairQueueByCa` also supports explicit `RANDOM` selection
- queue entries expire after `request_expire_seconds` and are also bounded by the snapped `window_end_time`
- one `ca_address` can hold only one active matchmaking state at a time: either one direct pending pair or one active queue entry
- new direct pending pairs reserve their maximum reward exposure up front
- `JoinPairQueueByCa` has two balance paths:
  - immediate match checks raw `remaining balance`
  - new queue entry checks `available balance` after reservations
- `GetRewardBalance()` returns `total_balance`, `reserved_balance`, and `available_balance`
- `GetAvailableRewardBalance()` returns the currently available reward balance after reservations
- `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` are the authoritative address-scoped read models for direct-vs-queue exclusivity
- `GetPairQueueStats()` returns the current active queue length and current `queue_capacity`, but some runtimes may decode `queue_capacity` as `0`; when that happens and `GetConfig().queue_capacity` is positive, prefer `GetConfig().queue_capacity`
- when a new `JoinPairQueueByCa` call cannot find a match and the queue is already full after expired-entry cleanup, the earliest still-valid queue entry is removed before the new caller joins
- `queue_capacity` defaults to `1000` when initialization leaves it as `0`, but later reads must always trust the current on-chain config
- `GetPairStatus` returns `NONE`, `PENDING`, or `EXECUTED`
- `GetPendingPair` returns an empty object when there is no active pending pair
- `GetCertificateStatus(address)` is still a placeholder view with `status = COMING_SOON`, but when the address already has a strong-resonance record it also returns the current strong payload for display
- all writes require `GetConfig().portkey_ca_contract_address` to be configured; if it is unset, the write must stop before sending

## Required First Step

For generic resonance requests without an explicit participation mode, do not jump directly to a write method.

The agent must first explain:

- the current `ResonanceContract v2.0.0` user-side write path is `CA` only
- `AA`, `CA`, and `AA/CA` are accepted input aliases, but canonical wording in this skill is `CA`
- the current contract no longer supports user-side `EOA` writes
- `direct pair`: use this when the user already knows the other side's `ca_hash`
- `queue`: use this when the user does not know a counterparty and wants automatic matching instead
- once local `CA` readiness and a usable signer or relayer are available, `queue` remains the formal executable automatic-matching path rather than a fallback to community posting
- direct-mode counterparty input must be a `ca_hash`, not `email`, not an on-chain `Address`
- queue entries are not permanent; the timeout comes from `GetConfig.request_expire_seconds` and must be explained in plain language before sending
- queue default matching is `FIFO`; when the queue is full and the new caller still does not match immediately, the earliest still-valid queued user may be removed before the new caller joins

Then:

- if the user explicitly asks for `EOA` or another non-CA write route, explain that the current contract version only supports CA-side participation and guide them back to a local `CA` account
- if the user has not provided or implied a local `CA` context yet, explain that the local `CA` account must be ready first
- if multiple local `CA` accounts are available and the caller is not already implied, ask which local `CA` account to use
- if the user did not already provide a counterparty `ca_hash` or explicitly ask for queue, ask whether they want `direct pair` or `queue`

## Routing Rules

Choose one branch before asking for extra transaction inputs.

### Branch 1: Account Choice And Participation Mode

Read [references/flows/account-choice.md](./references/flows/account-choice.md) when:

- the user makes a generic resonance request without explicitly choosing `direct pair` or `queue`
- the local caller `CA` context is not ready yet
- the user only says `help me resonate`, `帮我共振`, or similar
- the user explicitly asks for `EOA` or another non-CA route and needs a CA-only correction

### Branch 2: CA Create Pair Request

Read [references/flows/ca-create-pair-request.md](./references/flows/ca-create-pair-request.md) when:

- the user wants to initiate a direct pair request
- a local `CA` context is already available
- the user provided or confirmed a counterparty `ca_hash`

### Branch 3: CA Confirm Pair Request

Read [references/flows/ca-confirm-pair-request.md](./references/flows/ca-confirm-pair-request.md) when:

- the user wants to confirm an existing pending pair
- a local `CA` context is already available
- the user provided or confirmed the initiator `ca_hash`

### Branch 4: CA Join Pair Queue

Read [references/flows/ca-join-pair-queue.md](./references/flows/ca-join-pair-queue.md) when:

- the user wants automatic matching or queue participation
- a local `CA` context is already available

### Branch 5: CA Leave Pair Queue

Read [references/flows/ca-leave-pair-queue.md](./references/flows/ca-leave-pair-queue.md) when:

- the user wants to leave the automatic matching queue
- a local `CA` context is already available

### Branch 6: Status Query And Diagnostics

Read [references/flows/status-query-diagnostics.md](./references/flows/status-query-diagnostics.md) when:

- the user wants to inspect a pair or queue state without sending a write
- the user wants to inspect whether a `ca_hash` or `ca_address` is in queue or still has an active pending pair
- a prior write failed
- the user needs to understand `pending`, `expired`, `already resonated`, `queue`, `warmup`, wrong-call-path mismatch, or reward-balance state

## Global Hard Rules

- Never handle admin-only methods in this skill.
- Never ask the user for a direct-mode counterparty `email` or on-chain `Address`; direct mode now accepts `counterparty_ca_hash`.
- Never continue if the caller `CA` context cannot be resolved from a local CA account.
- Never continue if the current write path has no usable local relayer or signer that can pay gas for the CA-only transaction.
- Never present any underlying relay transport as the business method, as a route choice, or as a reason to abandon an otherwise executable CA-only write.
- Never require manager visibility, manager sync, or holder `manager_infos` checks for the current CA-only contract.
- Never attempt or teach any non-CA user-side write path; explain that `ResonanceContract v2.0.0` is CA-only instead.
