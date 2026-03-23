# Runtime Compatibility

Version: `1.2.0`

Use this file when the agent needs to normalize deployment config, reason about Portkey dependency versions, or apply known SDK fallbacks.

## Canonical Deployment

Current tDVV resonance deployment:

- `chain_id`: `tDVV`
- `rpc_url`: `https://tdvv-public-node.aelf.io`
- normalized full resonance address: `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj_tDVV`
- normalized raw resonance address: `GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`
- Portkey CA contract on `tDVV`: `2UthYi7AHRdfrqc1YCfeQnjdChDLaas65bW4WxESMGMojFiXj9`

## Contract Address Normalization

Accepted inputs for the resonance contract:

- `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj_tDVV`
- `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`
- `GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`

Normalization rule:

- reply with the normalized full address
- execute SDK and RPC reads or writes with the normalized raw address
- if the normalized full address and the incoming config differ, show both `input_contract_address` and `normalized_full_contract_address`

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
- `GetPairStatus` and `GetPendingPair` must use `camelCase` params: `addressA`, `addressB`
- `CreatePairRequest(Address)` and `ConfirmPairRequest(Address)` must encode the top-level `Address` input as a plain address string, not an object wrapper

## Known SDK Fallbacks

Known issues in the current toolchain:

- wrong invocation shape can surface misleading errors such as `Invalid signature`
- after a pair reaches `EXECUTED`, `GetPairStatus` can fail with an output-transform error in `aelf-sdk`
- `GetCertificateStatus(address)` can fail with the same output-transform error

Fallback order for executed-state confirmation:

1. decode the `PairResonated` event from the confirm transaction result
2. confirm that `GetPendingPair` is now empty
3. confirm that `GetAddressStats` changed as expected
4. read `GetStrongRecord` when available
5. if `GetCertificateStatus` still fails, report it as an SDK decode issue rather than a chain failure
