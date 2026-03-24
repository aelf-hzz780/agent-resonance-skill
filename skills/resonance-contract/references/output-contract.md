# Resonance Contract Output Contract

Version: `2.0.0`

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
- current `GetConfig` summary:
  - `version`
  - `resonance_enabled`
  - `start_time`
  - `end_time`
  - `request_expire_seconds`
  - `new_participation_available_time`
  - `queue_capacity`
  - `success_amount`
  - `strong_bonus_amount`
- current participation-state summary from the relevant pair or queue reads
- `user_explanation` when the reply needs plain-language guidance for ordinary users
- explicit confirmation request

Additional fields for `CreatePairRequest`:

- pair participants in caller and counterparty form
- direct-mode counterparty
- pending state from `GetPendingPair`
- `GetActivePendingPair` for caller and counterparty when practical
- `GetPairQueueStatus` for caller and counterparty when practical
- `GetRewardBalance` or `GetAvailableRewardBalance`
- create-side maximum reservation check
- whether an expired old pending pair may be auto-cleared on create

Additional fields for `ConfirmPairRequest`:

- pair participants in caller and initiator form
- active pending pair summary from `GetPendingPair`
- `GetRemainingBalance` result
- `GetRewardBalance` when practical for diagnostics
- baseline minimum pool check
- effective pair-specific minimum pool check when snapshots are available

Additional fields for `JoinPairQueue`:

- queue selection policy
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetPairQueueStatus` for the caller
- `GetActivePendingPair` for the caller
- `GetPairQueueStats` when available
- `GetRemainingBalance`
- `GetRewardBalance` or `GetAvailableRewardBalance`
- join-side maximum reward check
- note whether the current balances guarantee both outcomes or only leave open the immediate-match path
- whether the result may be immediate match or queued entry

Additional fields for `LeavePairQueue`:

- current `GetPairQueueStatus` for the caller
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetPairQueueStats` when available

Additional fields for `AA/CA`:

- manager signer
- resolved `AA/CA` holder address when available
- `caHash` when available
- `portkey_ca_contract_address`
- sender semantics: forwarded caller is the `AA/CA` holder address, not the manager signer

## User Explanation Block

When `user_explanation` is present, it must be written for ordinary users rather than contract engineers.

It must not replace the technical fields above. It should summarize the practical meaning of those fields in short plain-language sentences.

Queue-related replies must cover these topics when relevant:

- how long the queue or pending request stays valid
- what default `FIFO` means in user language
- what explicit `RANDOM` means in user language
- what happens when the queue is full
- why direct pending and queue states are mutually exclusive
- how queue-join balance checks split between immediate-match and queued outcomes
- whether new participation is still blocked until the admin finalizes the upgrade on a known upgraded legacy deployment
- whether an upgrade warmup window is blocking new participation

Suggested plain-language content for `zh-CN`:

- queue timeout: `这次排队不是永久有效的，会在 <queue_timeout_humanized> 后失效。`
- default FIFO: `如果你不指定策略，系统会默认按先进入队列、且仍然符合条件的人优先尝试匹配。`
- explicit RANDOM: `如果你选择随机模式，系统会在当前可匹配的人里随机选一个，不保证先来先配。`
- queue full: `如果队列已经满了，而且这次也没有马上匹配成功，系统会先移除当前最早且仍有效的一位排队用户，然后让你进入队列。`
- queue capacity note: `这里的满员上限来自当前配置的 queue_capacity；默认上限是 1000，但以链上当前配置为准。`
- exclusivity: `一个地址同一时间只能处于一种参与状态：要么有一笔待确认配对，要么在排队中，不能同时两边都占着。`
- join balance split: `排队加入有两种结果：如果这次直接匹配成功，合约看的是当前剩余余额；如果只是成功入队，合约看的是扣除预留后的可用余额。`
- upgrade not finalized: `合约升级后的新参与入口还没重新开放，管理员完成 finalize 之前不能新发起配对或排队。`
- warmup: `合约升级后可能会有一个冷却期，在这个时间点之前不能新发起配对或排队。`
- certificate placeholder with strong payload: `证书功能还没开放，不过已经检测到你的强共振记录。`

Suggested plain-language content for `en`:

- queue timeout: `This queue entry is not permanent and will expire in <queue_timeout_humanized>.`
- default FIFO: `If you do not choose a policy, the contract first tries the earliest still-eligible queued address.`
- explicit RANDOM: `If you choose random mode, the contract picks from the currently eligible queued addresses without guaranteeing first-come-first-served order.`
- queue full: `If the queue is already full and this call still cannot match immediately, the contract removes the earliest still-valid queued address before letting you join.`
- queue capacity note: `The full-queue limit comes from the current on-chain queue_capacity. The default is 1000, but the live chain value wins.`
- exclusivity: `One address can only occupy one participation state at a time: either one active pending pair or one active queue entry.`
- join balance split: `Joining the queue has two balance paths: an immediate match checks raw remaining balance, while a queued result checks available balance after reservations.`
- upgrade not finalized: `New participation is still closed after the upgrade. New direct pairs and queue joins cannot start until the admin finalizes the upgrade.`
- warmup: `After an upgrade, the contract may enforce a warmup window before new direct pairs or queue joins are allowed.`
- certificate placeholder with strong payload: `Certificate issuance is not open yet, but the contract already shows your strong-resonance record.`

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
- `window_end_time` when available

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

After `JoinPairQueue`, summarize one of these outcomes:

- if the transaction matched immediately:
  - `PairResonated` event fields
  - matched initiator and counterparty
  - `outcome`
  - `random_number`
  - `reward_each`
  - `executed_time`
  - `GetAddressStats` when practical
  - `GetStrongRecord` and `GetCertificateStatus` when practical
- if the caller remained queued:
  - `GetPairQueueStatus`
  - `joined_time`
  - `expire_time`
  - `success_amount_snapshot`
  - `strong_bonus_amount_snapshot`
  - `window_end_time`
  - `GetPairQueueStats`

After `LeavePairQueue`, summarize:

- `GetPairQueueStatus`
- whether the caller is still in queue
- `GetPairQueueStats`
- removal reason when it can be inferred from the transaction result or event logs

For diagnostics-only replies, summarize the relevant subset of:

- `GetConfig`
- `GetPairStatus`
- `GetPendingPair`
- `GetActivePendingPair`
- `GetPairQueueStatus`
- `GetPairQueueStats`
- `GetRewardBalance`
- `GetAvailableRewardBalance`
- `GetRemainingBalance`
- `GetStrongRecord`
- `GetCertificateStatus`
- fallback evidence used

## Community CTA Policy

Append CTA blocks only when the reply is one of these:

- a pending pair was successfully created
- a pair confirmation returned a clear executed result
- a queue join returned a clear success result such as `queued` or `immediate match`
- a queue leave returned a clear non-error result
- a pure status query returned a clear non-error result such as active pending or executed

Do not append CTA blocks when the reply is a hard stop, missing-config stop, invalid-input stop, or failed precondition diagnosis.

## Community CTA Strings

### `zh-CN`

- `→ 想继续找更多共振对象？可以去 [X / Twitter](https://x.com/aelfblockchain) 留言发帖，同步你的配对状态。`
- `→ 也欢迎加入 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl)，带着你的当前结果和下一步需求继续找搭子。`

### `en`

- `→ Want to find more resonance partners? Post or reply on [X](https://x.com/aelfblockchain) and share your pairing status.`
- `→ You can also join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) with your current result and next step to match faster.`
