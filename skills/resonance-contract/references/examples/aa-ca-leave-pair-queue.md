# Example: AA/CA Leave Pair Queue

## User Input

- English: `I choose AA/CA. My local account is ready. Leave the resonance queue for me.`
- 中文: `我选 AA/CA，本地账号已经好了，帮我退出共振排队。`

## Agent Should Choose

- `AA/CA Leave Pair Queue`

## Must Ask Or Confirm

- whether the user wants to send the forwarded `LeavePairQueue()` call after seeing the pre-send summary

## Must Not Ask

- do not ask for `email`
- do not ask for `caHash` from the user when the local `AA/CA` context already exists

## Must-Stop Conditions

- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- target execution chain holder info does not yet include the chosen manager signer
- holder is not currently in the queue

## Correct Output Shape

- use a natural operation label such as `退出共振队列` in the default user-facing layer instead of exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, caller identity, whether the holder still appears queued, timeout guidance when relevant, and the practical effect of leaving now
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- keep manager signer, holder address, `caHash`, target raw execution address, target CA contract, current queue status, timeout summary, and queue stats when available in the localized technical-details layer unless the user asks to expand
- explain in plain language that if the holder is already no longer in queue, the reason may be expiry, match, active leave, or capacity eviction
- ask for explicit confirmation before sending
- after sending, return the updated queue status and removal reason when it can be inferred
- append community CTA because the queue leave returned a clear non-error result
