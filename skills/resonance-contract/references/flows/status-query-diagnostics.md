# Status Query And Diagnostics

Use this branch when the user wants a read-only lookup or a precise diagnosis after a failure in direct pair mode or queue mode.

## When To Use

Use this flow when any of the following is true:

- the user wants to inspect a pair without sending a write
- the user wants to inspect one `ca_hash` or `ca_address` for active pending-pair or queue status
- a prior `CreatePairRequestByCa`, `ConfirmPairRequestByCa`, `JoinPairQueueByCa`, or `LeavePairQueueByCa` failed
- the user wants to understand `pending`, `expired`, `already resonated`, `already in queue`, warmup, queue capacity, reward-balance state, or wrong-call-path mismatch

## Required Reads

Read these first:

- `GetConfig()`

All reads in this branch must use direct view calls:

- use `contract.<Method>.call(...)` or the dependency skill's direct view path
- never use a generic send path or any other non-view transport for resonance `Get*` or other view-only methods in this branch

Read these when relevant to the user input:

- `GetPairStatus()` for unordered-pair diagnosis
- `GetPendingPair()` for unordered-pair diagnosis
- `GetActivePendingPair(address)` for address-scoped direct-state diagnosis
- `GetPairQueueStatus(address)` for address-scoped queue diagnosis
- `GetPairQueueStats()` for queue-capacity diagnosis
- `GetRewardBalance()` for create/join balance diagnosis
- `GetAvailableRewardBalance()` for create/join balance diagnosis
- `GetRemainingBalance()` for confirm-path diagnosis
- `GetStrongRecord()` for executed strong-result diagnosis
- `GetCertificateStatus()` for executed strong-result diagnosis
- `GetAddressStats()` for executed-result diagnosis
- executed transaction result logs for event fallback when executed-state views fail

## Identity Normalization Rules

- if the user provides `ca_hash`, resolve it into `ca_address` through the configured Portkey CA contract first
- if the user provides `ca_address`, read the pair or queue state directly
- if the flow needs pair-oriented diagnosis and the user provided two `ca_hash` values, resolve both before calling `GetPairStatus()` or `GetPendingPair()`
- if `GetConfig().portkey_ca_contract_address` is missing, explain that `ca_hash` inputs cannot currently be resolved into `ca_address`

## Status Interpretation Rules

- if `GetConfig()` looks uninitialized, stop and say the contract is not ready
- if `GetConfig().portkey_ca_contract_address` is missing and the user input is `ca_hash`, stop and say the contract cannot currently resolve CA identities for status lookup
- if the pasted receipt references a method outside the current CA-only `*ByCa` write set and the direct-view `Get*` reads, treat it as a receipt from an older or unsupported route, say it does not belong to the current CA-only runbook, and do not expand the old workflow
- if the prior agent invoked a resonance `Get*` or other view-only method through a generic send path or another non-view transport, first explain that the wrong call path was used and that the receipt cannot substitute for the direct view response
- if the available evidence is only a send receipt, explain that it does not by itself expose the direct view payload or final business state
- if `new_participation_available_time` is missing or unset on an otherwise initialized contract, explain that this is an abnormal state or decode issue first
- if `new_participation_available_time` is still in the future, explain that new `CreatePairRequestByCa` and `JoinPairQueueByCa` actions are blocked during the upgrade warmup window
- if the pair query is invalid after identity normalization, stop and explain that both sides must resolve to distinct `ca_address` values
- if the address-scoped reads show `GetActivePendingPair(address)` is active, explain that the CA address already occupies direct pending state
- if the address-scoped reads show `GetPairQueueStatus(address).in_queue == true`, explain that the CA address is already queued and cannot also hold a direct pending pair
- if `GetPairStatus().status == EXECUTED`, say the unordered pair has already resonated and cannot be retried through either direct mode or queue mode
- if `GetPairStatus()` fails after an executed confirm because of a known SDK output-decode issue, fall back to the transaction's `PairResonated` event plus `GetPendingPair()` and `GetAddressStats()`
- if `GetPairStatus().status == PENDING`, return the active pending pair fields, including timeout and `window_end_time`, and explain who must confirm
- if `GetPairStatus().status == NONE` and `GetPendingPair()` is empty:
  - the pair may never have been created
  - or an old pending pair may already be expired and cleared
  - the contract does not provide pending-history enumeration, so older expiration proof may require events or an indexer
- if `GetPairQueueStatus(address).in_queue == false` but the user expected to be queued, explain that the queue entry may already have expired, been matched, been actively left, or been evicted when the queue was full
- if `GetPairQueueStats().queue_capacity == 0` but `GetConfig().queue_capacity > 0`, explain that queue stats appear stale or under-decoded and that the config capacity should be treated as source of truth
- if a create-path diagnosis shows low `GetAvailableRewardBalance()`, explain that direct create reserves maximum reward up front and therefore depends on available balance
- if a join-path diagnosis shows low `GetRemainingBalance()`, explain that neither immediate match nor queue entry can succeed
- if a join-path diagnosis shows sufficient `GetRemainingBalance()` but low `GetAvailableRewardBalance()`, explain that immediate match may still succeed while a pure queued result is not guaranteed
- if a confirm-path diagnosis shows low `GetRemainingBalance()`, explain that confirm uses the raw remaining reward pool check rather than the available-balance reservation check
- if `GetCertificateStatus()` reports `COMING_SOON` but also shows `has_strong == true`, explain that certificate issuance is not open yet while the strong-resonance record already exists
- if the endpoint host is browser-reachable but SDK or JSON-RPC calls to `rpc_url` do not return a usable response, explain that `GET` reachability and JSON-RPC `POST` health are different checks

## Must-Stop Conditions

Stop immediately if any of the following is true:

- the user asks this skill to perform an admin action
- required config is missing for the requested chain
- the user is asking for pair-oriented diagnosis but the provided pair input cannot be normalized into two distinct `ca_address` values
- the user is asking for address-oriented diagnosis but the provided identity input is neither a valid `ca_hash` nor a valid `ca_address`

## Output Shape

The reply should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions.portkey_ca`, the current status conclusion, the most important time, status, or balance anchor, the practical reason for that state, and the next suggested action
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- surface `dependency_mode` in the default layer only when compatibility mode or runtime-metadata reliability materially affects the diagnosis
- localized technical-details layer on demand with chosen flow, current config summary, identity normalization details, pair status summary, address-scoped pending or queue summary, pending-pair details, queue details, queue stats, balance reads, executed outcome details, `GetStrongRecord`, `GetCertificateStatus`, and fallback evidence
- if the diagnosis started from a send receipt, say in the default layer first whether that receipt came from the wrong path for a view-only method, then put the exact replacement direct-view query in technical details
- if the diagnosis ends in a clear non-error state such as active pending, executed, or still queued, append the success CTA in the default layer
- if the diagnosis ends in a real blocker or externally stalled state the agent cannot continue automatically, including missing chain/runtime config or missing `portkey_ca_contract_address`, append the support CTA in the default layer
- do not append any CTA for invalid input, missing required user input, or light routing corrections that still leave the agent able to continue in the same conversation

## Example Reference

Read [../examples/status-query-diagnostics.md](../examples/status-query-diagnostics.md), [../examples/support-cta-diagnostics.md](../examples/support-cta-diagnostics.md), and [../examples/wrong-call-path-vs-correct-call-path.md](../examples/wrong-call-path-vs-correct-call-path.md) before replying.
