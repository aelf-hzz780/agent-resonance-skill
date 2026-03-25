# Example: AA/CA Confirm Pair Request

## User Input

- English: `I choose AA/CA. My local account is ready. Confirm the pair with initiator <address>.`
- 中文: `我选 AA/CA，本地账号已经好了，帮我确认这个发起方的配对：<address>`

## Agent Should Choose

- `AA/CA Confirm Pair Request`

## Must Ask Or Confirm

- whether the user wants to send the forwarded `ConfirmPairRequest(initiator)` after seeing the pre-send summary

## Must Not Ask

- do not ask for the initiator `email`
- do not ask for `caHash` from the user when the local `AA/CA` context already exists
- do not skip `GetRemainingBalance()`

## Must-Stop Conditions

- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- no active pending pair
- initiator mismatch
- resolved holder address is not the pending counterparty
- remaining balance is too low for the effective maximum reward

## Correct Output Shape

- use a natural operation label such as `确认共振请求` in the default user-facing layer instead of exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, caller identity, initiator, target normalized full `resonance_contract_address`, whether the write can proceed, pending validity guidance, and the confirm-side balance conclusion
- keep manager signer, holder address, `caHash`, target raw execution address, target CA contract, active pending pair summary, and pool check summary in the localized technical-details layer unless the user asks to expand
- keep `GetRemainingBalance()` as the real confirm-side gate, and only treat `GetRewardBalance()` or `GetAvailableRewardBalance()` as diagnostics
- if `GetCertificateStatus()` is read after success, explain that certificate issuance is still `COMING_SOON` even when strong-resonance payload exists
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, final pair outcome, and any strong-record updates
- append community CTA because the confirm path returned a clear result
