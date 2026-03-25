# Example: AA/CA Join Pair Queue

## User Input

- English: `I choose AA/CA. My local account is ready. Put me into the resonance queue.`
- 中文: `我选 AA/CA，本地账号已经好了，帮我进共振排队。`

## Agent Should Choose

- `AA/CA Join Pair Queue`

## Must Ask Or Confirm

- whether the user wants to send the forwarded `JoinPairQueue(input)` call after seeing the pre-send summary

## Must Not Ask

- do not re-explain `AA/CA vs EOA`
- do not ask for `email`
- do not ask for `caHash` from the user when the local `AA/CA` context already exists

## Must-Stop Conditions

- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- target execution chain holder info does not yet include the chosen manager signer
- contract warmup has not finished yet
- holder already has an active pending pair
- holder is already in the queue
- remaining balance is too low for any queue-join outcome

## Correct Output Shape

- use a natural operation label such as `加入共振队列` or `Join resonance queue`, rather than exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, caller identity, queue policy in plain language, timeout, whether the write can proceed, and whether the likely result is immediate match or queued entry
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- keep manager signer, holder address, `caHash`, target raw execution address, target CA contract, queue capacity, current queue state, the dual-path balance interpretation, and join-side balance check in the localized technical-details layer unless the user asks to expand
- if the user later asks to inspect queue state, switch back to direct view reads such as `GetPairQueueStatus` instead of reusing the forwarded receipt
- if available balance is low but remaining balance is still enough, explain that immediate match may still succeed while plain enqueue is not guaranteed
- if the user did not specify a policy, explain in plain language that the default is FIFO, meaning the contract first tries the earliest still-eligible queued address
- if the user explicitly chose `RANDOM`, explain in plain language that the contract chooses from the currently eligible queued addresses without guaranteeing first-come-first-served order
- explain in plain language that a full queue may evict the earliest still-valid queued address before this holder joins
- ask for explicit confirmation before sending
- after sending, return either `queued` with queue status or `immediate match` with executed result
- if the forwarded receipt includes `VirtualTransactionCreated`, keep it in technical details as forwarded-write evidence only; do not present it as the queue-status result
- append community CTA because the queue join returned a clear non-error result
