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
- counterparty is invalid
- pair already resonated
- pair already pending

## Correct Output Shape

- identify the branch as `AA/CA Create Pair Request`
- explain that `CA` still maps to the `AA/CA` branch
- show manager signer, holder address, `caHash` when available, counterparty, target contracts, and forwarded method chain
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, and pending pair summary
- append community CTA because a pending pair was created
