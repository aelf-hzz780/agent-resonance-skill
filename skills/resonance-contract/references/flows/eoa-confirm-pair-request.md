# EOA Confirm Pair Request

Use this branch for the `EOA` direct-confirm path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose direct pair mode rather than queue mode
- the user wants to confirm an existing pending pair

## Required Facts

- Method: `ConfirmPairRequest(Address initiator)`
- Caller must be the pending counterparty
- Initiator input must be a valid on-chain `Address`
- `ConfirmPairRequest` requires an active pending pair
- `ConfirmPairRequest` requires the raw remaining reward pool to cover the maximum reward tier before randomness is sampled
- The contract uses the pending pair snapshots for the effective maximum reward
- `ConfirmPairRequest` is not blocked by `new_participation_available_time`; the warmup gate applies to new create/join participation only
- `GetCertificateStatus()` still reports `COMING_SOON`, but it may also carry strong-record payload when a strong resonance already exists

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Use the Portkey EOA skill to resolve the active local `EOA` signer.
4. Validate the initiator input as an on-chain `Address`.
5. Stop if the initiator address equals the resolved signer address.
6. Read `GetConfig()`.
7. Stop if the contract appears uninitialized.
8. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
9. Read `GetPairStatus()` for the unordered pair.
10. Read `GetPendingPair()` for the unordered pair.
11. Stop if `GetPairStatus().status == EXECUTED`.
12. Stop if there is no active pending pair.
13. Stop if the pending pair initiator does not match the requested initiator.
14. Stop if the resolved local signer does not match the pending pair counterparty.
15. Read `GetRemainingBalance()`.
16. Read `GetRewardBalance()` and `GetAvailableRewardBalance()` when practical for diagnostic context, but keep `GetRemainingBalance()` as the actual confirm-side gate.
17. Compute the baseline minimum pool check as `2 * (config.success_amount + config.strong_bonus_amount)`.
18. If pending snapshots are available, also compute the effective pair-specific minimum pool check as `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)`.
19. Stop if the remaining balance is lower than the effective pair-specific minimum pool check.
20. Show the pre-send summary using the output contract, including normalized contract addresses, active pending pair summary, raw balance check, optional reward-balance diagnostics, and `user_explanation`.
21. Ask for explicit confirmation.
22. Only after explicit confirmation, use the Portkey EOA skill to send `ConfirmPairRequest(initiator)`.
23. If a `txId` is returned, share the `txId` and explorer link.
24. Read `GetPairStatus()` again.
25. Read `GetAddressStats()` for both participant addresses when practical.
26. Read `GetStrongRecord()` for both participant addresses when practical.
27. Read `GetCertificateStatus()` for both participant addresses when practical.
28. If executed-state views fail because of an SDK decode issue, decode the `PairResonated` event and combine it with `GetPendingPair()` and `GetAddressStats()` as the final confirmation.
29. Return the read-after-write summary with outcome, `reward_each`, and any strong-record or certificate-payload updates.
30. Append the community CTA because the confirm path returned a clear result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the initiator input is not a valid on-chain `Address`
- the initiator equals the caller
- the pair has already resonated
- there is no active pending pair
- the pending pair initiator does not match the requested initiator
- the resolved signer is not the pending counterparty
- the remaining balance is lower than the effective pair-specific minimum pool check

## Output Shape

The response before sending should contain:

- chosen flow: `EOA Confirm Pair Request`
- resolved signer
- initiator
- target resonance contract in normalized full-address and raw-address form
- method `ConfirmPairRequest(Address initiator)`
- current window and reward tiers
- active pending pair summary
- `GetRemainingBalance`
- baseline minimum pool check
- effective pair-specific minimum pool check when snapshots are available
- `GetRewardBalance` and `GetAvailableRewardBalance` when practical for diagnostics
- `user_explanation` that clarifies pending validity, confirm-side balance semantics, and certificate placeholder behavior when relevant
- explicit confirmation request

## Example Reference

Read [../examples/eoa-confirm-pair-request.md](../examples/eoa-confirm-pair-request.md) before replying.
