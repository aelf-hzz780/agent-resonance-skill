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

All reads in this branch must use direct view calls:

- for `AA/CA`, use `contract.<Method>.call(...)` or Portkey CA `view-call`
- for `EOA`, use `portkey_call_view_method`, CLI `contract view`, or an SDK read call
- never use `CA.ManagerForwardCall` or an `EOA` send path for resonance `Get*` or other view-only methods in this branch

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
- if the prior agent invoked a resonance `Get*` or other view-only method through `CA.ManagerForwardCall` or an `EOA` send path, first explain that the wrong call path was used and that the receipt cannot substitute for the direct view response
- if the available evidence includes `VirtualTransactionCreated`, explain that it only proves the forwarded inner call was created and does not by itself expose the inner view payload or final business result
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
- if the endpoint host is browser-reachable but SDK or JSON-RPC calls to `rpc_url` do not return a usable response, explain that `GET` reachability and JSON-RPC `POST` health are different checks
- if the available evidence is only `GET works`, `TLS works`, and JSON-RPC `POST` times out, hangs, resets, returns empty, or otherwise never yields a usable JSON-RPC response, report an RPC transport problem in the current environment without claiming a specific root cause such as `VPN`, `router`, `SSL`, `certificate`, or `firewall`
- if JSON-RPC `POST` returns a structured JSON-RPC error, exact contract error, parameter-validation error, or ABI/decode error, explain that this is a returned RPC response rather than a transport failure

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

- localized user-summary layer first with `skill_version`, `dependency_versions`, the current status conclusion, the most important time, status, or balance anchor, the practical reason for that state, and the next suggested action
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- surface `dependency_mode` in the default layer only when compatibility mode or runtime-metadata reliability materially affects the diagnosis
- localized technical-details layer on demand with chosen flow, current config summary, pair status summary, address-scoped pending or queue summary, pending-pair details, queue details, queue stats, balance reads, executed outcome details, `GetStrongRecord`, `GetCertificateStatus`, and fallback evidence
- if the diagnosis started from a forwarded or send receipt, say in the default layer first whether that receipt came from the wrong path for a view-only method, then put the exact replacement direct-view query in technical details
- when the diagnosis involves RPC reachability, keep the default layer wording bounded to observed facts and avoid over-attributing the root cause
- if the diagnosis ends in a real blocker or externally stalled state the agent cannot continue automatically, including missing chain/runtime config, append the support CTA in the default layer
- do not append any CTA for invalid input, missing required user input, or light routing corrections that still leave the agent able to continue in the same conversation

## Example Reference

Read [../examples/status-query-diagnostics.md](../examples/status-query-diagnostics.md) and [../examples/support-cta-diagnostics.md](../examples/support-cta-diagnostics.md) before replying.
