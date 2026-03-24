# Example: EOA Leave Pair Queue

## User Input

- English: `I choose EOA. My local wallet is ready. Leave the resonance queue for me.`
- 中文: `我选 EOA，本地钱包已经好了，帮我退出共振排队。`

## Agent Should Choose

- `EOA Leave Pair Queue`

## Must Ask Or Confirm

- whether the user wants to send `LeavePairQueue()` after seeing the pre-send summary

## Must Not Ask

- do not ask for `email` or `caHash`

## Must-Stop Conditions

- caller is not currently in the queue

## Correct Output Shape

- identify the branch as `EOA Leave Pair Queue`
- show signer, normalized contract addresses, current queue status, timeout summary, and queue stats when available
- explain in plain language that if the user is already no longer in queue, the reason may be expiry, match, active leave, or capacity eviction
- ask for explicit confirmation before sending
- after sending, return the updated queue status and removal reason when it can be inferred
- append community CTA because the queue leave returned a clear non-error result
