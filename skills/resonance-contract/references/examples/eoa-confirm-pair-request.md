# Example: EOA Confirm Pair Request

## User Input

- English: `I choose EOA. My local wallet is ready. Confirm the pair with initiator <address>.`
- 中文: `我选 EOA，本地钱包已经好了，帮我确认这个发起方的配对：<address>`

## Agent Should Choose

- `EOA Confirm Pair Request`

## Must Ask Or Confirm

- whether the user wants to send `ConfirmPairRequest(initiator)` after seeing the pre-send summary

## Must Not Ask

- do not ask for `email`
- do not ask for `caHash`
- do not skip `GetRemainingBalance()`

## Must-Stop Conditions

- no active pending pair
- initiator mismatch
- caller is not the pending counterparty
- remaining balance is too low for the effective maximum reward

## Correct Output Shape

- identify the branch as `EOA Confirm Pair Request`
- show signer, initiator, normalized contract addresses, method, active pending pair summary, and pool check summary
- keep `GetRemainingBalance()` as the real confirm-side gate, and only treat `GetRewardBalance()` or `GetAvailableRewardBalance()` as diagnostics
- if `GetCertificateStatus()` is read after success, explain that certificate issuance is still `COMING_SOON` even when strong-resonance payload exists
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, final pair outcome, and any strong-record updates
- append community CTA because the confirm path returned a clear result
