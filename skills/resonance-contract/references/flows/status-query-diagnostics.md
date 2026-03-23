# Status Query And Diagnostics

Use this branch when the user wants a read-only status lookup or a precise diagnosis after a failure.

## When To Use

Use this flow when any of the following is true:

- the user wants to inspect a pair without sending a write
- a prior `CreatePairRequest` or `ConfirmPairRequest` failed
- the user wants to understand `pending`, `expired`, `already resonated`, or current contract-window state

## Required Reads

Read these first:

- `GetConfig()`
- `GetPairStatus()`
- `GetPendingPair()`

Read these when relevant:

- `GetRemainingBalance()` for confirm-path diagnosis
- `GetStrongRecord()` for executed strong-result diagnosis
- `GetCertificateStatus()` for executed strong-result diagnosis
- `GetAddressStats()` for executed-result diagnosis
- executed transaction result logs for event fallback when executed-state views fail

## Status Interpretation Rules

- if `GetConfig()` looks uninitialized, stop and say the contract is not ready
- if the pair query is invalid, stop and explain that both sides must be distinct on-chain addresses
- if `GetPairStatus().status == EXECUTED`, say the unordered pair has already resonated and cannot be retried
- if `GetPairStatus()` fails after an executed confirm because of a known SDK output-decode issue, fall back to the confirm transaction's `PairResonated` event plus `GetPendingPair()` and `GetAddressStats()`
- if `GetPairStatus().status == PENDING`, return the active pending pair fields and explain who must confirm
- if `GetPairStatus().status == NONE` and `GetPendingPair()` is empty:
  - the pair may never have been created
  - or an old pending pair may already be expired and cleared
  - the contract does not provide pending-history enumeration, so older expiration proof may require events or an indexer
- if a confirm-path diagnosis also shows remaining balance below the required minimum, explain that the reward pool is insufficient for the maximum tier

## Exact Error Mapping

When the chain returns these exact contract errors, preserve them:

- `Self pairing is not allowed.`
- `Pair has already resonated.`
- `Pair request is already pending.`
- `Pair request not found.`
- `Pair request has expired.`
- `Initiator does not match pending pair request.`
- `Resonance is disabled.`
- `Resonance has not started.`
- `Resonance has ended.`
- `Insufficient reward balance.`
- `Invalid pair query.`

## Must-Stop Conditions

Stop immediately if any of the following is true:

- the user asks this skill to perform an admin action
- required config is missing for the requested chain
- the pair input is not a pair of distinct on-chain addresses
- the local caller context is required for diagnosis but cannot be resolved

## Output Shape

The reply should contain:

- chosen flow: `Status Query And Diagnostics`
- current config summary
- current pair status summary
- pending-pair details when active
- executed outcome details when available
- fallback evidence used when executed-state views are unavailable
- precise diagnosis for expired, missing, already resonated, or insufficient reward balance cases
- no community CTA for hard-stop or failure diagnoses

## Example Reference

Read [../examples/status-query-diagnostics.md](../examples/status-query-diagnostics.md) before replying.
