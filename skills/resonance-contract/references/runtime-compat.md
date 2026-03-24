# Runtime Compatibility

Version: `2.0.0`

Use this file when the agent needs to normalize deployment config, reason about Portkey dependency versions, explain queue semantics, reason about reward reservation, diagnose warmup, or apply known SDK fallbacks.

## Current Known Runtime Facts

Current validated `tDVV` runtime facts:

- `chain_id`: `tDVV`
- `rpc_url`: `https://tdvv-public-node.aelf.io`
- resonance contract raw address on `tDVV`: `28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
- resonance contract normalized full address on `tDVV`: `ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`
- Portkey CA contract on `tDVV`: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`

## Contract Address Normalization

Accepted inputs for the resonance contract:

- `<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>_<chain_id>`

Normalization rule:

- if no explicit resonance contract address is provided and the runtime is the validated `tDVV` environment, default to:
  - raw: `28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy`
  - full: `ELF_28Lot71VrWm1WxrEjuDqaepywi7gYyZwHysUcztjkHGFsPPrZy_tDVV`
- reply with the normalized full address
- execute SDK and RPC reads or writes with the normalized raw address
- if the incoming full address suffix does not match the runtime `chain_id`, stop and explain the mismatch
- if the normalized full address and the incoming config differ, show both `input_contract_address` and `normalized_full_contract_address`
- if the runtime is not the validated `tDVV` environment and no explicit address is available, stop and explain the missing deployment config

## Portkey Dependency Mode

Validated local Portkey dependency versions:

- Portkey CA skill: `2.2.0`
- Portkey EOA skill: `1.2.4`

Observed local compatibility mode:

- `2.1.x` still works for the resonance flow, but the agent must apply the fallbacks below

Suggested reply metadata when the runtime is not on the validated dependency version:

- `dependency_versions.portkey_ca = <detected version>`
- `dependency_mode = compatibility`
- if the dependency runtime metadata reports `0.0.0` but the local package manifest has a real version, prefer the manifest version and report the runtime metadata path as buggy

## Participation Gating

Create-side methods:

- `CreatePairRequest`
- `JoinPairQueue`

Both require all three conditions:

- resonance is enabled
- current time is inside the configured window
- `GetConfig().new_participation_available_time` is not in the future

Confirm-side note:

- `ConfirmPairRequest` is not gated by `new_participation_available_time`

Plain-language requirement:

- if `new_participation_available_time` is missing on an otherwise initialized contract, report it as an abnormal state or decode issue first; only explain it as "new participation is still blocked until admin finalizes the upgrade" when the deployment is known to be an upgraded legacy instance waiting for finalize
- if `new_participation_available_time` is still in the future, explain it as an upgrade warmup or cooling-off window rather than only repeating the raw field name

## Balance Model

Use these reads deliberately:

- `GetRemainingBalance()` => raw FT balance currently held by the contract
- `GetRewardBalance()` => `total_balance`, `reserved_balance`, and `available_balance`
- `GetAvailableRewardBalance()` => the currently available reward balance after subtracting active reservations

Write-path rule:

- `CreatePairRequest` reserves the maximum possible reward up front, so its diagnostics must use `available_balance`
- `JoinPairQueue` is dual-path:
  - immediate match uses raw `remaining balance`
  - new queue entry uses `available_balance`
- `ConfirmPairRequest` still checks the raw balance against the maximum reward of the active pending pair snapshot
- `WithdrawRemaining` and `ExecuteWithdrawRemaining` are admin-only and also use available balance, but this skill only mentions that in diagnostics

## Direct vs Queue Exclusivity

One address can hold only one active matchmaking state at a time:

- either one active direct pending pair
- or one active queue entry

Preflight implications:

- direct create must check `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` for caller and counterparty
- queue join must check `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` for the caller

Plain-language requirement:

- explain this as "one address can only occupy one participation state at a time" instead of only citing ABI names

## Queue Semantics

Queue behavior that the skill must explain in ordinary language:

- queue timeout comes from `GetConfig().request_expire_seconds`
- queue entries also stop being active after their snapped `window_end_time`
- empty/default `JoinPairQueueInput` means `FIFO`
- explicit `RANDOM` means choose from the currently eligible candidates without guaranteeing first-come-first-served order
- if the queue is full and the new caller still cannot match immediately, the earliest still-valid queue entry is removed before the new caller joins
- `queue_capacity` defaults to `1000` only when initialization passed `0`; user-facing replies must still treat the current on-chain `queue_capacity` as source of truth

## Recovery Propagation Check

For `AA/CA` create or confirm after recovery:

1. confirm that recovery status is already `pass`
2. query holder info on the target execution chain, not only the origin chain
3. verify the chosen manager address is already present in the holder's manager list
4. only then treat the manager signer as ready for `CA.ManagerForwardCall`

Reason:

- recovery can be visible on `AELF` before the same manager becomes visible on `tDVV`

## AElf SDK Invocation Rules

Use these rules exactly:

- `GetConfig` must be called as `call()` with no `{}` for `Empty` input
- `GetRemainingBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetRewardBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetAvailableRewardBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetPairQueueStats` must be called as `call()` with no `{}` for `Empty` input
- `GetPairStatus` and `GetPendingPair` must use `camelCase` params: `addressA`, `addressB`
- `CreatePairRequest(Address)`, `ConfirmPairRequest(Address)`, `GetActivePendingPair(address)`, `GetPairQueueStatus(address)`, `GetStrongRecord(address)`, `GetCertificateStatus(address)`, and `GetAddressStats(address)` must encode the top-level `Address` input as a plain address string, not an object wrapper
- `JoinPairQueue` default `FIFO` path should use empty/default input rather than inventing an unrelated wrapper shape

## Known SDK Fallbacks

Known issues in the current toolchain:

- wrong invocation shape can surface misleading errors such as `Invalid signature`
- after a pair reaches `EXECUTED`, `GetPairStatus` can fail with an output-transform error in `aelf-sdk`
- `GetCertificateStatus(address)` can fail with the same output-transform error
- queue joins that match immediately may need event decoding from `PairResonated` to identify the matched counterparty and outcome without guessing

Fallback order for executed-state confirmation:

1. decode the `PairResonated` event from the confirm transaction result
2. confirm that `GetPendingPair` is now empty
3. confirm that `GetAddressStats` changed as expected
4. read `GetStrongRecord` when available
5. if `GetCertificateStatus` still fails, report it as an SDK decode issue rather than a chain failure
