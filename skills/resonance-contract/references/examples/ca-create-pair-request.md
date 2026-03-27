# Example: CA Create Pair Request

## User Input

- English: `I want to use CA. My local account is ready. Create a pair request for this counterparty ca_hash: <ca-hash>.`
- 中文: `我想用 CA，本地账号已经好了，帮我给这个对手方 ca_hash 发起配对：<ca-hash>`

## Agent Should Choose

- `CA Create Pair Request`

## Must Ask Or Confirm

- whether the user wants to send `CreatePairRequestByCa({ caHash, counterpartyCaHash })` after seeing the pre-send summary

## Must Not Ask

- do not reopen the legacy account-type explanation
- do not ask for the counterparty `email`
- do not ask for the counterparty `Address`

## Must-Stop Conditions

- local caller `CA` context cannot be resolved
- counterparty `ca_hash` is invalid or cannot be resolved
- `GetConfig().portkey_ca_contract_address` is missing
- contract warmup has not finished yet
- pair already resonated
- pair already pending
- caller or counterparty already has another active pending pair
- caller or counterparty is already in the queue
- available reward balance is too low for a new direct reservation

## Correct Output Shape

- use a natural operation label such as `发起配对请求` or `Create pair request`, rather than exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions.portkey_ca`, caller identity, counterparty identity, whether the write can proceed, timeout guidance, and the main blocker or balance conclusion
- keep `caller_ca_hash`, `caller_ca_address`, `counterparty_ca_hash`, `counterparty_ca_address`, target raw execution address, config reads, pair reads, queue reads, and create-side balance check in the localized technical-details layer unless the user asks to expand
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, and pending pair summary with `window_end_time` when available
- append success CTA because a pending pair was created
- render success CTA in `X -> Telegram` order
