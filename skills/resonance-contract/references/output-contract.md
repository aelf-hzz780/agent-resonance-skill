# Resonance Contract Output Contract

Version: `2.1.1`

Use this file for reply formatting after the branch flow is chosen.

## Language Selection

Resolve `output_language` before rendering:

1. explicit user language instruction wins
2. if the latest user request is mainly Chinese, use `zh-CN`
3. otherwise use `en`

## Monolingual Rule

- Once `output_language` is selected, keep all visible fixed strings monolingual.
- Proper nouns, chain IDs, method names, URLs, and filesystem paths may remain as-is.

## Two-Layer Response Contract

Every write reply and diagnostics-only reply must render in two layers:

- localized summary label as the default visible layer:
  - `zh-CN`: `普通用户摘要`
  - `en`: `User Summary`
- localized technical layer label as the expandable engineering layer:
  - `zh-CN`: `技术详情`
  - `en`: `Technical Details`

Render only the selected localized header for the chosen `output_language`.
Do not show both language labels together in the same user-visible reply.

Default rendering rules:

- show `skill_version` and `dependency_versions` in the default visible layer
- hide `dependency_mode` by default when it is just `normal`
- move `dependency_mode` into the default visible layer only when the dependency is in compatibility mode or dependency runtime metadata itself is unreliable
- keep `caHash`, the raw execution address, extra contract addresses such as the Portkey CA contract, method chain, full config reads, full pair or queue reads, and fallback evidence in `Technical Details` unless the user explicitly asks for them
- end the default visible layer with a short single-language hint that `Technical Details` can be expanded on request
- do not expose internal branch names in the default visible layer; use a natural operation label instead
- if the user is asking for read-only status and the available evidence came from a forwarded or send receipt instead of a direct view path, say that first in the default visible layer before diagnosing business state

Host rendering rule:

- if the host supports collapsible sections, render the technical layer collapsed by default
- if the host is plain chat without collapsible UI, render only the default visible layer and the expansion hint in the first reply
- in plain chat, do not print the technical-layer header or body until the user explicitly asks to expand
- in plain chat, the default visible layer may be rendered as plain prose without a literal section header as long as its content still follows this contract

Expand `Technical Details` when the user explicitly says:

- `展开详情`
- `debug`
- `看链上参数`
- `technical details`
- `show raw data`
- or explicitly asks for `caHash`, manager, holder, contract address, balance reads, queue stats, method chain, or raw fallback evidence

## CTA Classification

Every write reply and diagnostics-only reply must resolve `cta_type` before rendering:

- `success`
- `support`
- `none`

Default classification rule:

- use `success` for clear non-error outcomes that the user can build on now
- use `support` for blocked, stalled, or diagnosable-but-not-actionable-now states where the agent has explained the cause but cannot continue without outside help, coordination, or external recovery
- use `none` for malformed input, invalid address format, requests that still need more required user input, or purely local clarification that does not mean the user is actually stuck

CTA belongs to the default visible layer only. `Technical Details` should never be the only place that carries CTA text.

## Pre-Send Summary

Every write reply that can actually proceed must include a pre-send summary before asking for confirmation.

If preflight blocks the write, return a blocked summary instead:

- explain in plain language why the send cannot proceed
- show the key blocker or missing condition
- give the next practical step
- when the blocker is real but the agent cannot continue automatically, set `cta_type = support` and append the support CTA in the default visible layer
- do not ask for confirmation

For a concrete blocked example, read [./examples/blocked-write-summary.md](./examples/blocked-write-summary.md).

Default visible layer for all write paths:

- `skill_version`
- `dependency_versions` when available
- caller identity
- short natural operation name
- whether the write can proceed now
- expiry, timeout, or pending-validity anchor when relevant
- the most important success condition or blocker
- the practical outcome the user should expect after sending
- explicit confirmation request
- a short hint that `Technical Details` can be expanded on request
- include the target normalized full `resonance_contract_address` in the default visible layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant

Dependency-version rule:

- show only the dependency versions that were actually used for the current path
- `AA/CA` paths normally show `dependency_versions.portkey_ca`
- `EOA` paths normally show `dependency_versions.portkey_eoa`
- do not invent missing version values; if local metadata cannot be resolved, omit the missing field and explain it only in `Technical Details`

`dependency_mode` only has these meanings in this skill:

- `normal`
- `compatibility`

Do not invent extra values such as `fallback`. Runtime fallback behavior should be reported separately through `used_fallbacks` or fallback evidence.

Add `dependency_mode` to the default visible layer only when the dependency is in compatibility mode or its runtime metadata is unreliable.

`user_explanation` is the main content of the default visible layer rather than a minor appendix.

Technical Details for all write paths:

- chosen flow
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

Additional Technical Details for `CreatePairRequest`:

- pair participants in caller and counterparty form
- direct-mode counterparty
- pending state from `GetPendingPair`
- `GetActivePendingPair` for caller and counterparty when practical
- `GetPairQueueStatus` for caller and counterparty when practical
- `GetRewardBalance` or `GetAvailableRewardBalance`
- create-side maximum reservation check
- whether an expired old pending pair may be auto-cleared on create

Additional Technical Details for `ConfirmPairRequest`:

- pair participants in caller and initiator form
- active pending pair summary from `GetPendingPair`
- `GetRemainingBalance` result
- `GetRewardBalance` when practical for diagnostics
- baseline minimum pool check
- effective pair-specific minimum pool check when snapshots are available

Additional Technical Details for `JoinPairQueue`:

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

Additional Technical Details for `LeavePairQueue`:

- current `GetPairQueueStatus` for the caller
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetPairQueueStats` when available

Additional Technical Details for `AA/CA`:

- manager signer
- resolved `AA/CA` holder address when available
- `caHash` when available
- `portkey_ca_contract_address`
- sender semantics: forwarded caller is the `AA/CA` holder address, not the manager signer

## User Explanation Block

When `user_explanation` is present, it must be written for ordinary users rather than contract engineers.

It now forms the backbone of the default visible layer and should summarize the practical meaning of the technical fields in short plain-language sentences.

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
- technical details hint: `如需展开技术详情 / 看链上参数 / debug，我可以继续展开。`

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
- technical details hint: `If you want Technical Details, raw on-chain parameters, or debug context, I can expand them.`

## Post-Send Receipt

Default visible layer after a write should include:

- `skill_version`
- `dependency_versions` when available
- `txId` when returned
- explorer link when available
- final status when available
- short practical result such as `queued`, `pending`, `SUCCESS`, or `FAILED`
- key outcome anchors such as `reward_each`, `executed_time`, `expire_time`, or matched-versus-queued result when relevant
- a short exact chain error quote when the write failed
- a short hint that `Technical Details` can be expanded on request

Move `dependency_mode` into the default visible layer only when the dependency is in compatibility mode or its runtime metadata is unreliable.

Technical Details after a write should include:

- short note if the first lookup is still pending
- `used_fallbacks` whenever any compatibility handling, event-decoding fallback, manifest-version override, or read fallback was actually used
- classify any supplied forwarded or send receipt as a write receipt rather than a view payload
- if present, treat `VirtualTransactionCreated` only as forwarded-write evidence instead of a standalone business-success proof
- exact chain error when failed

## Read-After-Write Summary

After `CreatePairRequest`, keep the default visible layer focused on:

- pending pair created or not
- `txId` and explorer link
- `expire_time`
- whether the user now needs the counterparty to confirm

Put these in `Technical Details`:

- `GetPairStatus`
- `GetPendingPair`
- active pending `initiator`
- active pending `counterparty`
- `created_time`
- `expire_time`
- `success_amount_snapshot`
- `strong_bonus_amount_snapshot`
- `window_end_time` when available

After `ConfirmPairRequest`, keep the default visible layer focused on:

- final execution result
- `txId` and explorer link
- `outcome`
- `reward_each`
- `executed_time`

Put these in `Technical Details`:

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

After `JoinPairQueue`, keep the default visible layer focused on one of these outcomes:

- immediate match happened now
- caller successfully entered the queue
- caller was blocked before send

Put these detailed fields in `Technical Details`:

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

After `LeavePairQueue`, keep the default visible layer focused on:

- whether the caller is still in queue
- whether the leave actually succeeded, or whether the user was already expired, already matched, or already absent

Put these in `Technical Details`:

- `GetPairQueueStatus`
- whether the caller is still in queue
- `GetPairQueueStats`
- removal reason when it can be inferred from the transaction result or event logs

## Diagnostics-Only Replies

Default visible layer for diagnostics should include:

- `skill_version`
- `dependency_versions` when available
- current status conclusion
- the most important time, status, or balance anchor
- the practical reason for the current state
- the next practical step for the user
- a short hint that `Technical Details` can be expanded on request
- include the target normalized full `resonance_contract_address` in the default visible layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- when the current state is clearly blocked or stalled and the next step needs outside help rather than more agent work, append the support CTA in the default visible layer

Move `dependency_mode` into the default visible layer only when the dependency is in compatibility mode or its runtime metadata is unreliable.

Technical Details for diagnostics should include the relevant subset of:

- whether the prior evidence came from a correct direct-view path or from a forwarded/send receipt
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

Diagnostics-first rule:

- if the user is asking about status and the prior agent used `CA.ManagerForwardCall` or an `EOA` send path for a resonance `Get*` or other view-only method, first explain that the method was routed incorrectly
- if the user pasted a forwarded receipt that contains `VirtualTransactionCreated`, explain that it is a forwarded-write receipt and not the inner view response

Always end the default diagnostics layer with a short hint that `Technical Details` can be expanded on request.

## Community CTA Policy

Resolve CTA blocks by `cta_type`:

- `success`
  - a pending pair was successfully created
  - a pair confirmation returned a clear executed result
  - a queue join returned a clear success result such as `queued` or `immediate match`
  - a queue leave returned a clear non-error result
  - a pure status query returned a clear non-error result such as active pending, executed, or still queued
- `support`
  - a blocked summary where warmup, queue conflict, active pending conflict, insufficient reward balance, or similar real precondition blocker is already diagnosed
  - a diagnostics-only reply where the agent has reached a clear blocker such as missing chain/runtime config, RPC transport trouble, manager sync delay, dependency/runtime metadata trouble, decode trouble, or other externally blocked state
  - any other stalled state where the user is genuinely stuck and the next practical step is outside the agent's automatic reach
- `none`
  - invalid-input stop
  - address-format stop
  - requests that still need more required user input the agent can continue collecting
  - purely local routing or light correction that does not mean the user is stuck

Do not show both success and support CTA blocks in the same reply.

## Community CTA Strings

### Success CTA `zh-CN`

- `→ 想继续找更多共振对象？可以去 [X / Twitter](https://x.com/aelfblockchain) 留言发帖，同步你的配对状态。`
- `→ 也欢迎加入 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl)，带着你的当前结果和下一步需求继续找搭子。`

### Success CTA `en`

- `→ Want to find more resonance partners? Post or reply on [X](https://x.com/aelfblockchain) and share your pairing status.`
- `→ You can also join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) with your current result and next step to match faster.`

### Support CTA `zh-CN`

- `→ 如果这里卡住了，欢迎到 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl) 贴出你当前的步骤、报错和关键信息，我们可以一起帮你排查。`
- `→ 也可以去 [X / Twitter](https://x.com/aelfblockchain) 发帖求助，带上你当前的状态和卡点，方便社区更快看到并协助你。`

### Support CTA `en`

- `→ If you're stuck here, join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) and share your current step, error, and key context so the community can help troubleshoot.`
- `→ You can also post on [X](https://x.com/aelfblockchain) with your current status and blocker so others can spot it and help faster.`
