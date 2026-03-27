# Resonance Contract Output Contract

Version: `3.0.1`

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
- keep raw execution addresses, the configured Portkey CA contract, full config reads, full pair or queue reads, and fallback evidence in `Technical Details` unless the user explicitly asks for them
- end the default visible layer with a short single-language hint that `Technical Details` can be expanded on request
- do not expose internal branch names in the default visible layer; use a natural operation label instead
- if the user is asking for read-only status and the available evidence came from a legacy forwarded or generic send receipt instead of a direct view path, say that first in the default visible layer before diagnosing business state
- if queue preflight can already proceed with a resolved caller context plus signer or relayer readiness, do not downgrade the reply into support CTA or social fallback just because the host lacks a resonance-only standalone CLI

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
- or explicitly asks for `ca_hash`, `ca_address`, contract address, balance reads, queue stats, or raw fallback evidence

## CTA Classification

Every write reply and diagnostics-only reply must resolve `cta_type` before rendering:

- `success`
- `support`
- `none`

Default classification rule:

- use `success` for clear non-error outcomes that the user can build on now
- use `support` for blocked, stalled, or diagnosable-but-not-actionable-now states where the agent has explained the cause but cannot continue without outside help, coordination, or external recovery
- use `none` for malformed input, invalid identity format, requests that still need more required user input, or light routing corrections such as old-path explanations where the agent can still continue immediately
- when queue preflight can proceed, do not classify the state as `support`

CTA belongs to the default visible layer only. `Technical Details` should never be the only place that carries CTA text.

## CTA Rendering

When `cta_type = success`, append the localized success CTA in this order:

- first `X`
- then `Telegram`

When `cta_type = support`, append the localized support CTA in this order:

- first `Telegram`
- then `X`

Suggested success CTA for `zh-CN`:

- `→ 想继续找更多共振对象？可以去 [X / Twitter](https://x.com/aelfblockchain) 留言发帖，同步你的当前状态。`
- `→ 也欢迎加入 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl)，带着你当前的结果和下一步需求继续找搭子。`

Suggested success CTA for `en`:

- `→ Want to find more resonance partners? Post on [X](https://x.com/aelfblockchain) and share your current status.`
- `→ You can also join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) and continue matching with your latest result and next step.`

Suggested support CTA for `zh-CN`:

- `→ 如果这里卡住了，欢迎到 [Telegram 群](https://t.me/+tChFhfxgU6AzYjJl) 贴出你当前的步骤、报错和关键信息，我们可以一起帮你排查。`
- `→ 也可以去 [X / Twitter](https://x.com/aelfblockchain) 发帖求助，带上你当前的状态和卡点，方便社区更快看到并协助你。`

Suggested support CTA for `en`:

- `→ If you're stuck here, join the [Telegram group](https://t.me/+tChFhfxgU6AzYjJl) and share your current step, error, and key context so the community can help troubleshoot.`
- `→ You can also post on [X](https://x.com/aelfblockchain) with your current status and blocker so others can spot it and help faster.`

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
- `dependency_versions.portkey_ca` when available
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

- show only `dependency_versions.portkey_ca`
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
  - `portkey_ca_contract_address`
- current participation-state summary from the relevant pair or queue reads

Additional Technical Details for `CreatePairRequestByCa`:

- `caller_ca_hash`
- `caller_ca_address`
- `counterparty_ca_hash`
- `counterparty_ca_address`
- pending state from `GetPendingPair`
- `GetActivePendingPair` for caller and counterparty when practical
- `GetPairQueueStatus` for caller and counterparty when practical
- `GetRewardBalance` or `GetAvailableRewardBalance`
- create-side maximum reservation check
- whether an expired old pending pair may be auto-cleared on create
- write path note: `relayer wallet -> resonanceContract.CreatePairRequestByCa`

Additional Technical Details for `ConfirmPairRequestByCa`:

- `caller_ca_hash`
- `caller_ca_address`
- `initiator_ca_hash`
- `initiator_ca_address`
- active pending pair summary from `GetPendingPair`
- `GetRemainingBalance` result
- `GetRewardBalance` when practical for diagnostics
- baseline minimum pool check
- effective pair-specific minimum pool check when snapshots are available
- write path note: `relayer wallet -> resonanceContract.ConfirmPairRequestByCa`

Additional Technical Details for `JoinPairQueueByCa`:

- `caller_ca_hash`
- `caller_ca_address`
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
- write path note: `relayer wallet -> resonanceContract.JoinPairQueueByCa`

Additional Technical Details for `LeavePairQueueByCa`:

- `caller_ca_hash`
- `caller_ca_address`
- current `GetPairQueueStatus` for the caller
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetPairQueueStats` when available
- write path note: `relayer wallet -> resonanceContract.LeavePairQueueByCa`

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
- that queue remains the formal executable path once the local CA account and dependency signer or relayer are ready
- whether the contract still lacks a configured Portkey CA contract
- whether new participation is still blocked during the warmup window
- direct mode now needs `counterparty_ca_hash`, not `email` and not `Address`
- if the user asked for `EOA`, explain that the current contract version is CA-only

Suggested plain-language content for `zh-CN`:

- queue timeout: `这次排队不是永久有效的，会在 <queue_timeout_humanized> 后失效。`
- default FIFO: `如果你不指定策略，系统会默认按先进入队列、且仍然符合条件的人优先尝试匹配。`
- explicit RANDOM: `如果你选择随机模式，系统会在当前可匹配的人里随机选一个，不保证先来先配。`
- queue full: `如果队列已经满了，而且这次也没有马上匹配成功，系统会先移除当前最早且仍有效的一位排队用户，然后让你进入队列。`
- queue capacity note: `这里的满员上限来自当前配置的 queue_capacity；如果队列统计接口暂时读到 0，但配置里是正数，要以配置值为准。`
- exclusivity: `一个 CA 地址同一时间只能处于一种参与状态：要么有一笔待确认配对，要么在排队中，不能同时两边都占着。`
- join balance split: `排队加入有两种结果：如果这次直接匹配成功，合约看的是当前剩余余额；如果只是成功入队，合约看的是扣除预留后的可用余额。`
- warmup: `合约升级后可能会有一个冷却期，在这个时间点之前不能新发起配对或排队。`
- missing Portkey CA config: `当前合约还没有配置 Portkey CA 合约地址，所以系统暂时没法把 ca_hash 解析成 ca_address，写操作现在不能继续。`
- direct by ca hash: `当前 direct 模式要传的是对手方的 ca_hash，不是邮箱，也不是链上 Address。`
- EOA not supported: `当前合约版本已经下线 EOA 写路径，用户侧共振现在只支持 CA。`
- certificate placeholder with strong payload: `证书功能还没开放，不过已经检测到你的强共振记录。`
- technical details hint: `如需展开技术详情 / 看链上参数 / debug，我可以继续展开。`

Suggested plain-language content for `en`:

- queue timeout: `This queue entry is not permanent and will expire in <queue_timeout_humanized>.`
- default FIFO: `If you do not choose a policy, the contract first tries the earliest still-eligible queued CA address.`
- explicit RANDOM: `If you choose random mode, the contract picks from the currently eligible queued CA addresses without guaranteeing first-come-first-served order.`
- queue full: `If the queue is already full and this call still cannot match immediately, the contract removes the earliest still-valid queued entry before letting you join.`
- queue capacity note: `The full-queue limit comes from the current on-chain queue_capacity. If queue stats temporarily show 0 but config shows a positive number, trust config.`
- exclusivity: `One CA address can only occupy one participation state at a time: either one active pending pair or one active queue entry.`
- join balance split: `Joining the queue has two balance paths: an immediate match checks raw remaining balance, while a queued result checks available balance after reservations.`
- warmup: `After an upgrade, the contract may enforce a warmup window before new direct pairs or queue joins are allowed.`
- missing Portkey CA config: `The contract has not configured its Portkey CA contract yet, so it cannot resolve ca_hash into ca_address. Writes cannot continue yet.`
- direct by ca hash: `Direct mode now needs the counterparty ca_hash, not an email and not an on-chain Address.`
- EOA not supported: `The current contract version no longer supports user-side EOA writes. Resonance participation is CA-only now.`
- certificate placeholder with strong payload: `Certificate issuance is not open yet, but the contract already shows your strong-resonance record.`
- technical details hint: `If you want Technical Details, raw on-chain parameters, or debug context, I can expand them.`

## Post-Send Receipt

Default visible layer after a write should include:

- `skill_version`
- `dependency_versions.portkey_ca` when available
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
- `used_fallbacks` whenever any compatibility handling, event-decoding fallback, manifest-version override, queue-capacity fallback, or read fallback was actually used
- if present, treat any pasted `VirtualTransactionCreated` only as legacy forwarded-write evidence instead of a standalone business-success proof
- exact chain error when failed

## Read-After-Write Summary

After `CreatePairRequestByCa`, keep the default visible layer focused on:

- pending pair created or not
- `txId` and explorer link
- `expire_time`
- whether the user now needs the counterparty to confirm

Put these in `Technical Details`:

- `caller_ca_hash`
- `caller_ca_address`
- `counterparty_ca_hash`
- `counterparty_ca_address`
- `GetPairStatus`
- `GetPendingPair`
- active pending `initiator`
- active pending `counterparty`
- `created_time`
- `expire_time`
- `success_amount_snapshot`
- `strong_bonus_amount_snapshot`
- `window_end_time` when available

After `ConfirmPairRequestByCa`, keep the default visible layer focused on:

- final execution result
- `txId` and explorer link
- `outcome`
- `reward_each`
- `executed_time`

Put these in `Technical Details`:

- `caller_ca_hash`
- `caller_ca_address`
- `initiator_ca_hash`
- `initiator_ca_address`
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

After `JoinPairQueueByCa`, keep the default visible layer focused on one of these outcomes:

- immediate match happened now
- caller successfully entered the queue
- caller was blocked before send

Put these detailed fields in `Technical Details`:

- `caller_ca_hash`
- `caller_ca_address`
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
  - `window_end_time`
  - `GetPairQueueStats`

After `LeavePairQueueByCa`, keep the default visible layer focused on:

- whether the caller successfully left the queue
- `txId` and explorer link
- whether the address was already absent, expired, matched, or actively removed

Put these detailed fields in `Technical Details`:

- `caller_ca_hash`
- `caller_ca_address`
- latest `GetPairQueueStatus`
- latest `GetPairQueueStats`
- any `PairQueueEntryRemoved` event and reason when available
