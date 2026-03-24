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
- contract warmup has not finished yet
- pair already resonated
- pair already pending
- caller or counterparty already has another active pending pair
- caller or counterparty is already in the queue
- available reward balance is too low for a new direct reservation

## Correct Output Shape

- use a natural operation label such as `Create pair request` in the default user-facing layer instead of exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, caller identity, counterparty, target normalized full `resonance_contract_address`, whether the write can proceed, timeout guidance, and the main blocker or balance conclusion
- keep signer, counterparty, target raw execution address, method, config window, reward tiers, current pair state, address-scoped pending state, queue state, and create-side balance check in the localized technical-details layer unless the user asks to expand
- include plain-language guidance when relevant, such as queue exclusivity, warmup, timeout, and queue-full implications
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, and pending pair summary with `window_end_time` when available
- append community CTA because a pending pair was created
