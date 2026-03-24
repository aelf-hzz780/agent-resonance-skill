# Example: AA/CA Create Pair Request

## User Input

- English: `I choose CA. My local account is ready. Create a pair request for this address: <address>.`
- 中文: `我选 CA，本地账号已经好了，帮我给这个地址发起配对：<address>`

## Agent Should Choose

- `AA/CA Create Pair Request`

## Must Ask Or Confirm

- whether the user wants to send the forwarded `CreatePairRequest(counterparty)` after seeing the pre-send summary

## Must Not Ask

- do not re-explain `AA/CA vs EOA`
- do not ask for the counterparty `email`
- do not ask for the counterparty `caHash`

## Must-Stop Conditions

- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- target execution chain holder info does not yet include the chosen manager signer
- counterparty is invalid
- contract warmup has not finished yet
- pair already resonated
- pair already pending
- holder or counterparty already has another active pending pair
- holder or counterparty is already in the queue
- available reward balance is too low for a new direct reservation

## Correct Output Shape

- identify the branch as `AA/CA Create Pair Request`
- explain that `CA` still maps to the `AA/CA` branch
- show manager signer, holder address, `caHash` when available, counterparty, normalized target contracts, forwarded method chain, address-scoped pending state, queue state, and create-side balance check
- include plain-language guidance when relevant, such as queue exclusivity, warmup, timeout, and queue-full implications
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, and pending pair summary with `window_end_time` when available
- append community CTA because a pending pair was created
