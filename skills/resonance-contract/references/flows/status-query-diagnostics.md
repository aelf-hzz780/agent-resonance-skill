# Status Query And Diagnostics

Use this branch when the user wants a read-only lookup or a precise diagnosis after a failure in direct pair mode or queue mode.

## When To Use

Use this flow when any of the following is true:

- the user wants to inspect a pair without sending a write
- the user wants to inspect one address for active pending-pair or queue status
- a prior `CreatePairRequest`, `ConfirmPairRequest`, `JoinPairQueue`, or `LeavePairQueue` failed
- the user wants to understand `pending`, `expired`, `already resonated`, `already in queue`, warmup, queue capacity, or reward-balance state

## Required Reads

Read these first:

- `GetConfig()`

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

## Status Interpretation Rules

- if `GetConfig()` looks uninitialized, stop and say the contract is not ready
- if `new_participation_available_time` is missing or unset on an otherwise initialized contract, explain that this is an abnormal state or decode issue first; only use the "admin still needs to finalize the upgrade" explanation when the deployment is known to be an upgraded legacy instance
- if `new_participation_available_time` is still in the future, explain that new `CreatePairRequest` and `JoinPairQueue` actions are blocked during the upgrade warmup window
- if the pair query is invalid, stop and explain that both sides must be distinct on-chain addresses
- if the address-scoped reads show `GetActivePendingPair(address)` is active, explain that the address already occupies direct pending state
- if the address-scoped reads show `GetPairQueueStatus(address).in_queue == true`, explain that the address is already queued and cannot also hold a direct pending pair
- if `GetPairStatus().status == EXECUTED`, say the unordered pair has already resonated and cannot be retried through either direct mode or queue mode
- if `GetPairStatus()` fails after an executed confirm because of a known SDK output-decode issue, fall back to the confirm transaction's `PairResonated` event plus `GetPendingPair()` and `GetAddressStats()`
- if `GetPairStatus().status == PENDING`, return the active pending pair fields, including timeout and `window_end_time`, and explain who must confirm
- if `GetPairStatus().status == NONE` and `GetPendingPair()` is empty:
  - the pair may never have been created
  - or an old pending pair may already be expired and cleared
  - the contract does not provide pending-history enumeration, so older expiration proof may require events or an indexer
- if `GetPairQueueStatus(address).in_queue == false` but the user expected to be queued, explain that the queue entry may already have expired, been matched, been actively left, or been evicted when the queue was full
- if a create-path diagnosis shows low `GetAvailableRewardBalance()`, explain that direct create reserves maximum reward up front and therefore depends on available balance
- if a join-path diagnosis shows low `GetRemainingBalance()`, explain that neither immediate match nor queue entry can succeed
- if a join-path diagnosis shows sufficient `GetRemainingBalance()` but low `GetAvailableRewardBalance()`, explain that immediate match may still succeed while a pure queued result is not guaranteed
- if a confirm-path diagnosis shows low `GetRemainingBalance()`, explain that confirm uses the raw remaining reward pool check rather than the available-balance reservation check
- if `GetCertificateStatus()` reports `COMING_SOON` but also shows `has_strong == true`, explain that certificate issuance is not open yet while the strong-resonance record already exists

## Exact Error Mapping

When the chain returns these exact contract errors, preserve them:

- `Self pairing is not allowed.`
- `Pair has already resonated.`
- `Pair request is already pending.`
- `Pair request not found.`
- `Pair request has expired.`
- `Pair request is still active.`
- `Initiator does not match pending pair request.`
- `Resonance is disabled.`
- `Resonance has not started.`
- `Resonance has ended.`
- `Insufficient reward balance.`
- `Invalid pair query.`
- `Address is already in pair queue.`
- `Address already has an active pair request.`
- `Address is not in pair queue.`
- `New participation is blocked until admin finalizes the upgrade.`
- `New participation is warming up after upgrade.`

## Must-Stop Conditions

Stop immediately if any of the following is true:

- the user asks this skill to perform an admin action
- required config is missing for the requested chain
- the user is asking for pair-oriented diagnosis but the provided pair input is not a pair of distinct on-chain addresses
- the user is asking for address-oriented diagnosis but the provided address input is not a valid single on-chain address
- the local caller context is required for address-scoped diagnosis but cannot be resolved

## Output Shape

The reply should contain:

- chosen flow: `Status Query And Diagnostics`
- current config summary, including `new_participation_available_time` and `queue_capacity`
- current pair status summary when a pair query was provided
- current address-scoped pending or queue summary when an address query was provided
- pending-pair details when active
- queue details when active
- `GetPairQueueStats` when queue-capacity diagnosis is relevant
- `GetRewardBalance`, `GetAvailableRewardBalance`, or `GetRemainingBalance` when balance diagnosis is relevant
- executed outcome details when available
- `GetStrongRecord` and `GetCertificateStatus` when strong-result diagnosis is relevant
- fallback evidence used when executed-state views are unavailable
- `user_explanation` for queue timeout, queue-full eviction, direct-versus-queue exclusivity, warmup, and balance-model differences when relevant
- no community CTA for hard-stop or failure diagnoses

## Example Reference

Read [../examples/status-query-diagnostics.md](../examples/status-query-diagnostics.md) before replying.
