# Example: CA Confirm Pair Request

## User Input

- English: `Use my local CA account and confirm the pending pair started by this initiator ca_hash: <initiator-ca-hash>.`
- 中文: `用我本地的 CA 账号，帮我确认这个 initiator ca_hash 发起的 pending pair：<initiator-ca-hash>`

## Agent Should Choose

- `CA Confirm Pair Request`

## Must Ask Or Confirm

- whether the user wants to send `ConfirmPairRequestByCa({ initiatorCaHash, counterpartyCaHash })` after seeing the pre-send summary

## Must Not Ask

- do not ask for legacy account-type or relayer-internals details
- do not ask for the initiator `Address`

## Correct Output Shape

- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions.portkey_ca`, caller identity, initiator identity, pending expiry, whether the write can proceed, and the balance conclusion
- keep `caller_ca_hash`, `caller_ca_address`, `initiator_ca_hash`, `initiator_ca_address`, pending-pair reads, and confirm-side balance details in the localized technical-details layer unless the user asks to expand
- ask for explicit confirmation before sending
- after sending, return `txId`, explorer link, final result, `reward_each`, and `executed_time`
- append success CTA for clear execution results
- render success CTA in `X -> Telegram` order
