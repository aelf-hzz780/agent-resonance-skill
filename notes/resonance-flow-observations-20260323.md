# Resonance Flow Observations (2026-03-23)

## Runtime observations

- `GetConfig` on `ResonanceContract` must use `contract.GetConfig.call()` with no `{}` argument for `Empty` input.
- `GetRemainingBalance` on `ResonanceContract` must use `contract.GetRemainingBalance.call()` with no `{}` argument for `Empty` input.
- `GetPairStatus` and `GetPendingPair` must use `camelCase` fields `addressA` and `addressB`. `snake_case` does not work in the current `aelf-sdk` path.
- The generic `portkey_query_skill.ts view-call` path currently hides the real issue and can surface `Unknown error` for valid `Empty` input view methods.
- `aelf-sdk` can return a misleading `Invalid signature` error for some view calls when the invocation shape is wrong, even though the method is view-only.
- After a pair reaches `EXECUTED`, `GetPairStatus.call(...)` can hit an `aelf-sdk` output transform error: `undefined is not an object (evaluating 'inputType.fieldsArray.length')`.
- `GetCertificateStatus.call(address)` can hit the same `aelf-sdk` output transform error in the current toolchain.
- `GetAddressStats.call(address)` and `GetStrongRecord.call(address)` did not show the same transform problem in this run.

## Local context observations

- `~/.portkey/skill-wallet/context.v1.json` contains a default CA profile pointing to the `clawson6` holder address and hash, but the linked `~/.portkey/ca/mainnet.keystore.json` file contains placeholder metadata (`hash_need_password`, `ELF_ca_need_password_tDVV`). This context looks stale or inconsistent.
- No local keystore profile was found for `clawson6@virgilian.com` or `clawson5@virgilian.com` before recovery.

## Contract state observations

- Contract address in this run: `ELF_GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj_tDVV`
- Raw contract address in this run: `GuERnXwhCo9Wcj33FbzQXY5mwitHpJTx5hTbsMzuaTr9J2uxj`
- `GetConfig` read on 2026-03-23:
  - `version`: `1.3.0`
  - `tokenSymbol`: `AIBOUNTY`
  - `successAmount`: `200000000`
  - `strongBonusAmount`: `500000000`
  - `resonanceEnabled`: `true`
  - `requestExpireSeconds`: `1800`
- Pair state for:
  - initiator candidate `2XhYNaVDL9rLuUc1pBBxSCLMrLvLju5R8qU2Sitc5ntxbydX7p`
  - counterparty candidate `Kh3EtoCi3jRae1yF4SV591D1kgmWwK2GejQ62B72TH5Y9UP8S`
- Current `GetPairStatus`: `NONE`
- Current `GetPendingPair`: `null`
- `GetRemainingBalance` after test funding: `1000000000` (`10 AIBOUNTY`)
- Confirm-path minimum pool check from current config: `1400000000` (`14 AIBOUNTY`)
- Result: create can proceed after local AA recovery, but confirm remains blocked until the contract balance is at least `14 AIBOUNTY` or the pair-specific snapshot requirement is lower.
- `GetRemainingBalance` after later top-up: `2000000000` (`20 AIBOUNTY`)
- With `20 AIBOUNTY`, the current baseline confirm-path pool check is satisfied.

## Recovery workflow observations

- `clawson6@virgilian.com` recovery is on `AELF` and currently requires 2 guardian approvals:
  - `clawson6@virgilian.com`
  - `clawson5@virgilian.com`
- `clawson5@virgilian.com` recovery is on `AELF` and currently requires 1 guardian approval:
  - `clawson5@virgilian.com`
- Recovery requests for both accounts returned `status = pass`, and the new manager addresses were visible in `GetHolderInfo` on `AELF`:
  - `clawson6@virgilian.com` -> manager `2nJPxqEHCiQNfaVFBfACi9DdYbLzhT8UGf4B2eEAJcKHeVDie1`
  - `clawson5@virgilian.com` -> manager `2W31jkF5xYEzAdx22mU6oumx7veua2rVjpRt6w5wHTxMscYMSH`
- The same new managers were not visible in `GetHolderInfo` on `tDVV` after repeated re-checks more than 20 seconds later.
- Result: `check-status = pass` is not sufficient to infer that the recovered manager is already usable for `tDVV` `CA.ManagerForwardCall`.
- Later re-checks showed both recovered managers on `tDVV` as well, so the propagation issue is delayed consistency rather than a permanent recovery failure:
  - `clawson6@virgilian.com` -> manager `2nJPxqEHCiQNfaVFBfACi9DdYbLzhT8UGf4B2eEAJcKHeVDie1`
  - `clawson5@virgilian.com` -> manager `2W31jkF5xYEzAdx22mU6oumx7veua2rVjpRt6w5wHTxMscYMSH`
- The current CLI `portkey_tx_skill.ts` only resolves a signer from in-process `unlock` state or `PORTKEY_PRIVATE_KEY`; it does not automatically decrypt the saved keystore profile in a new process, even though the lower-level signer-context path exists in `src/core/keystore.ts`.
- After upgrading the local Portkey CA skill from `2.1.0` to `2.2.0`, `portkey_tx_skill.ts` can resolve the signer through `--login-email` plus `--password` or `--keystore-file` in a fresh process, so the earlier limitation is specific to the older package.
- Even on local Portkey CA skill `2.2.0`, the runtime wallet context metadata still wrote `lastWriter.version = 0.0.0` before patching because the code depended on `process.env.npm_package_version` under `bun run <file>.ts`.
- The same runtime version metadata bug exists in the local Portkey EOA skill `1.2.4`.
- After patching the local runtime version source to read from `package.json`, `~/.portkey/skill-wallet/context.v1.json` now records `lastWriter.version = 2.2.0` after `portkey-ca` unlock.
- The local Portkey EOA skill had no `node_modules` installed before validation, so the CLI could not start with `Cannot find module '@portkey/aelf-signer'`.
- After running `bun install` in the local Portkey EOA skill directory, the EOA CLI help path started normally again.

## End-to-end run result

- `clawson6@virgilian.com` successfully sent `CreatePairRequest` for `clawson5@virgilian.com`.
- `clawson5@virgilian.com` successfully sent `ConfirmPairRequest` for `clawson6@virgilian.com`.
- The final `PairResonated` event decoded to:
  - `outcome`: `SUCCESS`
  - `randomNumber`: `6`
  - `rewardEach`: `200000000` (`2 AIBOUNTY`)
  - `executedTime`: `2026-03-23T09:44:21.1982626Z`
