# Example: CA Join Pair Queue

## User Input

- English: `Use my local CA account and join the resonance queue with the default policy.`
- 中文: `用我本地的 CA 账号，按默认策略加入共振队列`

## Agent Should Choose

- `CA Join Pair Queue`

## Must Ask Or Confirm

- whether the user wants to send `JoinPairQueueByCa({ caHash, selectionPolicy? })` after seeing the pre-send summary

## Must Not Ask

- do not ask for a counterparty when the user already chose queue
- do not ask for an `Address`

## Correct Output Shape

- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions.portkey_ca`, caller identity, queue policy, timeout, whether the write can proceed, and the main blocker or balance conclusion
- explain that queue join may either match immediately or enter the queue
- explain that queue capacity comes from the live chain config, and if stats temporarily show `0` while config is positive, config wins
- keep `caller_ca_hash`, `caller_ca_address`, queue reads, queue stats, and balance reads in the localized technical-details layer unless the user asks to expand
- ask for explicit confirmation before sending
- after sending, return either `queued` or `immediate matched`
- append success CTA for clear non-error join results
- render success CTA in `X -> Telegram` order
