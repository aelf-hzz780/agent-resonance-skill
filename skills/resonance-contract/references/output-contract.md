# Resonance Contract Output Contract

Version: `1.1.0`

Use this file for reply formatting after the branch flow is chosen.

## Language Selection

Resolve `output_language` before rendering:

1. explicit user language instruction wins
2. if the latest user request is mainly Chinese, use `zh-CN`
3. otherwise use `en`

## Monolingual Rule

- Once `output_language` is selected, keep all visible fixed strings monolingual.
- Proper nouns, chain IDs, method names, URLs, and filesystem paths may remain as-is.

## Pre-Send Summary

Every write reply must include a pre-send summary before asking for confirmation.

Required fields for all write paths:

- `skill_version`
- `dependency_versions` when available
- `dependency_mode` when a fallback or compatibility path is in effect
- chosen flow
- caller identity
- input contract address when the incoming value differs from the normalized value
- target normalized full `resonance_contract_address`
- target raw `resonance_contract_address` used for execution
- method or method chain
- pair participants in caller and counterparty or initiator form
- current `GetConfig` summary:
  - `version`
  - `resonance_enabled`
  - `start_time`
  - `end_time`
  - `request_expire_seconds`
  - `success_amount`
  - `strong_bonus_amount`
- current pair state from `GetPairStatus`
- explicit confirmation request

Additional fields for `CreatePairRequest`:

- pending state from `GetPendingPair`
- whether an expired old pending pair may be auto-cleared on create

Additional fields for `ConfirmPairRequest`:

- active pending pair summary from `GetPendingPair`
- `GetRemainingBalance` result
- baseline minimum pool check
- effective pair-specific minimum pool check when snapshots are available

Additional fields for `AA/CA`:

- manager signer
- resolved `AA/CA` holder address when available
- `caHash` when available
- `portkey_ca_contract_address`
- sender semantics: forwarded caller is the `AA/CA` holder address, not the manager signer

## Post-Send Receipt

If the write returns `txId`, include:

- `txId`
- explorer link
- short note if the first lookup is still pending
- `used_fallbacks` when a compatibility path was needed

If the final chain result is available, include:

- final status
- exact chain error when failed

## Read-After-Write Summary

After `CreatePairRequest`, summarize:

- `GetPairStatus`
- `GetPendingPair`
- active pending `initiator`
- active pending `counterparty`
- `created_time`
- `expire_time`
- `success_amount_snapshot`
- `strong_bonus_amount_snapshot`

After `ConfirmPairRequest`, summarize:

- `GetPairStatus`
- `status`
- `outcome`
- `random_number`
- `reward_each`
- `executed_time`
- `GetAddressStats` for participant addresses when practical
- `GetStrongRecord` for participant addresses when practical
- `GetCertificateStatus` for participant addresses when practical
- if `GetPairStatus` or `GetCertificateStatus` failed because of an SDK decode issue, say so explicitly and cite the fallback source used instead

## Community CTA Policy

Append CTA blocks only when the reply is one of these:

- a pending pair was successfully created
- a pair confirmation returned a clear executed result
- a pure status query returned a clear non-error result such as active pending or executed

Do not append CTA blocks when the reply is a hard stop, missing-config stop, invalid-input stop, or failed precondition diagnosis.

## Community CTA Strings

### `zh-CN`

- `→ 想继续找更多共振对象？可以去 [X / Twitter](https://x.com/aelfblockchain) 留言发帖，同步你的配对状态。`
- `→ 也欢迎加入 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl)，带着你的当前结果和下一步需求继续找搭子。`

### `en`

- `→ Want to find more resonance partners? Post or reply on [X](https://x.com/aelfblockchain) and share your pairing status.`
- `→ You can also join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) with your current result and next step to match faster.`
