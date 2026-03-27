# Example: CA Leave Pair Queue

## User Input

- English: `Use my local CA account and leave the resonance queue.`
- 中文: `用我本地的 CA 账号退出共振队列`

## Agent Should Choose

- `CA Leave Pair Queue`

## Must Ask Or Confirm

- whether the user wants to send `LeavePairQueueByCa({ caHash })` after seeing the pre-send summary

## Correct Output Shape

- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions.portkey_ca`, caller identity, current queue-state conclusion, whether the write can proceed, and what leaving now will do
- keep `caller_ca_hash`, `caller_ca_address`, queue reads, and queue stats in the localized technical-details layer unless the user asks to expand
- ask for explicit confirmation before sending
- after sending, return `left queue` or the clearest non-error explanation if the address was already absent
- append success CTA for clear non-error leave results
- render success CTA in `X -> Telegram` order
