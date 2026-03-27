# Runtime Compatibility

Version: `3.0.1`

Use this file when the agent needs to normalize deployment config, reason about Portkey CA dependency versions, explain CA-only identity semantics, diagnose warmup or reward reservation, or apply known SDK fallbacks.

## Current Known Runtime Facts

Current validated `tDVV` runtime facts:

- `chain_id`: `tDVV`
- `rpc_url`: `https://tdvv-public-node.aelf.io`
- resonance contract raw address on `tDVV`: `RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
- resonance contract normalized full address on `tDVV`: `ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`
- current expected Portkey CA contract on `tDVV`: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`
- validated Portkey CA skill version: `2.3.0`

## Contract Address Normalization

Accepted inputs for the resonance contract:

- `<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>`
- `ELF_<raw_base58_contract_address>_<chain_id>`

Normalization rule:

- if no explicit resonance contract address is provided and the runtime is the validated `tDVV` environment, default to:
  - raw: `RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49`
  - full: `ELF_RXnedMaCt4QRJoSca9CG2Qf8Qt9e5prowzRHd6JJh9ue8AD49_tDVV`
- reply with the normalized full address
- execute SDK and RPC reads or writes with the normalized raw address
- if the incoming full address suffix does not match the runtime `chain_id`, stop and explain the mismatch
- if the incoming config differs from the normalized value, show both `input_contract_address` and `normalized_full_contract_address`
- if the runtime is not the validated `tDVV` environment and no explicit address is available, stop and explain the missing deployment config

## RPC Transport Semantics

Treat `rpc_url` as the AElf JSON-RPC base endpoint, not as a browser-only page URL.

Transport rule:

- AElf SDK reads and writes use HTTP `POST` against `rpc_url`
- the same base URL may also render a browser-facing page such as Swagger on `GET`
- a successful browser `GET` does not prove that JSON-RPC `POST` is healthy
- a successful TLS handshake also does not prove that the RPC method path, upstream gateway, proxy, WAF, CDN, or node-side request handling is healthy

Reply rule:

- if `GET` works but JSON-RPC `POST` times out, hangs, resets, returns an empty response, fails before a parseable JSON-RPC payload is returned, or otherwise never yields a usable JSON-RPC response, describe it as an RPC transport or endpoint availability problem in the current environment
- if JSON-RPC `POST` returns a structured JSON-RPC error, HTTP application error, exact contract error, parameter-validation error, or ABI/decode error, do not call it a transport problem
- do not claim a specific root cause such as `VPN`, `router`, `SSL`, `certificate`, or `firewall` unless the evidence clearly isolates that cause

## Call Path Semantics

Treat method type as the first routing decision.

Routing rule:

- all current user-side resonance writes target the CA-only business methods:
  - `CreatePairRequestByCa`
  - `ConfirmPairRequestByCa`
  - `JoinPairQueueByCa`
  - `LeavePairQueueByCa`
- all resonance `Get*` and other view-only methods must use the direct view path such as `contract.<Method>.call(...)`
- never present `ManagerForwardCall` as the business method or as a legacy-required route; when the validated Portkey CA dependency uses a relay transport underneath a current CA-only write, that transport detail is allowed
- do not treat forwarded or generic send receipts as substitutes for direct view responses
- if a prior agent invoked a resonance `Get*` method through legacy `CA.ManagerForwardCall` or a generic send path, diagnose the wrong call path first before interpreting contract state
- if a user pastes an old `EOA` or `ManagerForwardCall` write receipt, explain that it belongs to the pre-`v2.0.0` contract path and does not describe the current CA-only write model

## CA Identity And Relay Semantics

Treat business identity and relayer identity separately.

Meaning:

- current user-side writes identify participants by `ca_hash`
- the contract resolves every `ca_hash` into a `ca_address` by calling the configured Portkey CA contract
- direct, queue, stats, strong-record, and certificate state all run on resolved `ca_address`
- `Context.Sender` is only the relayer or operator wallet that paid for the transaction
- permissionless relay is accepted semantics in this contract version, not a bug

Practical consequence:

- direct create and confirm need `ca_hash` inputs for writes
- read-only pair and queue diagnosis still uses `ca_address`
- if the user provides `ca_hash` for a status lookup, resolve it into `ca_address` first through the configured Portkey CA contract, then perform the reads

## Portkey CA Dependency Mode

Validated local dependency version:

- Portkey CA skill: `2.3.0`

Dependency-version source rule:

- prefer the actually detected local dependency version for the current path
- prefer runtime metadata; if runtime metadata is `0.0.0` but the local package manifest has a real version, prefer the manifest version and report the runtime metadata bug in fallback evidence
- never copy the validated version into a user reply unless the local runtime or manifest actually confirms that version
- if the local dependency version still cannot be resolved reliably, omit `dependency_versions.portkey_ca` from the default visible layer and explain the omission only in technical details

Suggested reply metadata when the runtime is not on the validated dependency version:

- `dependency_versions.portkey_ca = <detected version>`
- `dependency_mode = compatibility`

## Write Preflight Hard Checks

Apply these checks before any current user-side write:

1. `GetConfig()` must succeed
2. `GetConfig().portkey_ca_contract_address` must be configured
3. the contract must be initialized
4. the local caller `CA` context must be available

If `GetConfig().portkey_ca_contract_address` is missing:

- stop before sending
- explain that the current contract cannot resolve `ca_hash` into `ca_address`
- do not attempt best-effort write retries

## Participation Gating

Create-side methods:

- `CreatePairRequestByCa`
- `JoinPairQueueByCa`

Both require all three conditions:

- resonance is enabled
- current time is inside the configured window
- `GetConfig().new_participation_available_time` is not in the future

Confirm-side note:

- `ConfirmPairRequestByCa` is not gated by `new_participation_available_time`

Plain-language requirement:

- if `new_participation_available_time` is missing on an otherwise initialized contract, report it as an abnormal state or decode issue first
- if `new_participation_available_time` is still in the future, explain it as an upgrade warmup or cooling-off window rather than only repeating the raw field name

## Balance Model

Use these reads deliberately:

- `GetRemainingBalance()` => raw FT balance currently held by the contract
- `GetRewardBalance()` => `total_balance`, `reserved_balance`, and `available_balance`
- `GetAvailableRewardBalance()` => the currently available reward balance after subtracting active reservations

Write-path rule:

- `CreatePairRequestByCa` reserves the maximum possible reward up front, so its diagnostics must use `available_balance`
- `JoinPairQueueByCa` is dual-path:
  - immediate match uses raw `remaining balance`
  - new queue entry uses `available_balance`
- `ConfirmPairRequestByCa` still checks the raw balance against the maximum reward of the active pending pair snapshot

## Direct vs Queue Exclusivity

One `ca_address` can hold only one active matchmaking state at a time:

- either one active direct pending pair
- or one active queue entry

Preflight implications:

- direct create must check `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` for caller and counterparty
- queue join must check `GetActivePendingPair(address)` and `GetPairQueueStatus(address)` for the caller

Plain-language requirement:

- explain this as "one CA address can only occupy one participation state at a time" instead of only citing ABI names

## Queue Semantics

Queue behavior that the skill must explain in ordinary language:

- queue timeout comes from `GetConfig().request_expire_seconds`
- queue entries also stop being active after their snapped `window_end_time`
- empty/default `JoinPairQueueByCaInput` means `FIFO`
- explicit `RANDOM` means choose from the currently eligible candidates without guaranteeing first-come-first-served order
- if the queue is full and the new caller still cannot match immediately, the earliest still-valid queue entry is removed before the new caller joins
- `queue_capacity` defaults to `1000` only when initialization passed `0`
- if `GetPairQueueStats().queue_capacity` is `0` but `GetConfig().queue_capacity` is positive, prefer `GetConfig().queue_capacity` in user-facing explanation and diagnostics

## AElf SDK Invocation Rules

Use these rules exactly:

- `GetConfig` must be called as `call()` with no `{}` for `Empty` input
- `GetRemainingBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetRewardBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetAvailableRewardBalance` must be called as `call()` with no `{}` for `Empty` input
- `GetPairQueueStats` must be called as `call()` with no `{}` for `Empty` input
- `GetPairStatus` and `GetPendingPair` must use `camelCase` params: `addressA`, `addressB`
- `GetActivePendingPair(address)`, `GetPairQueueStatus(address)`, `GetStrongRecord(address)`, `GetCertificateStatus(address)`, and `GetAddressStats(address)` must encode the top-level `Address` input as a plain address string, not an object wrapper
- `CreatePairRequestByCa` input uses `camelCase` fields: `caHash`, `counterpartyCaHash`
- `ConfirmPairRequestByCa` input uses `camelCase` fields: `initiatorCaHash`, `counterpartyCaHash`
- `JoinPairQueueByCa` input uses `camelCase` fields: `caHash`, optional `selectionPolicy`
- `LeavePairQueueByCa` input uses `camelCase` fields: `caHash`
- when using the default `FIFO` queue path, omit `selectionPolicy` unless the current SDK requires an explicit default enum

## Known SDK Fallbacks

Known issues in the current toolchain:

- wrong invocation shape can surface misleading errors such as `Invalid signature`
- after a pair reaches `EXECUTED`, `GetPairStatus` can still fail with an output-transform error in some `aelf-sdk` environments
- `GetCertificateStatus(address)` can fail with the same output-transform error
- queue joins that match immediately may need event decoding from `PairResonated` to identify the matched counterparty and outcome without guessing
- some environments decode `GetPairQueueStats().queue_capacity` as `0`; prefer `GetConfig().queue_capacity` when it is positive

Fallback order for executed-state confirmation:

1. decode the `PairResonated` event from the transaction result
2. confirm that `GetPendingPair` is now empty
3. confirm that `GetAddressStats` changed as expected
4. read `GetStrongRecord` when available
5. if `GetCertificateStatus` still fails, report it as an SDK decode issue rather than a chain failure
