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

- identify the branch as `AA/CA Leave Pair Queue`
- show manager signer, holder address, `caHash` when available, normalized target contracts, current queue status, timeout summary, and queue stats when available
- explain in plain language that if the holder is already no longer in queue, the reason may be expiry, match, active leave, or capacity eviction
- ask for explicit confirmation before sending
- after sending, return the updated queue status and removal reason when it can be inferred
- append community CTA because the queue leave returned a clear non-error result
