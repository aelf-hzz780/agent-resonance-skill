# Example: EOA Create Pair Request

## User Input

- English: `I choose EOA. My local wallet is ready. Create a pair request for this address: <address>.`
- 中文: `我选 EOA，本地钱包已经好了，帮我给这个地址发起配对：<address>`

## Agent Should Choose

- `EOA Create Pair Request`

## Must Ask Or Confirm

- whether the user wants to send `CreatePairRequest(counterparty)` after seeing the pre-send summary

## Must Not Ask

- do not re-explain `AA/CA vs EOA`
- do not ask for `email` or `caHash`

## Must-Stop Conditions

- counterparty is invalid
- counterparty equals caller
- pair already resonated
- pair already pending

## Correct Output Shape

- identify the branch as `EOA Create Pair Request`
- show signer, counterparty, contract, method, config window, reward tiers, and current pair state
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, and pending pair summary
- append community CTA because a pending pair was created
