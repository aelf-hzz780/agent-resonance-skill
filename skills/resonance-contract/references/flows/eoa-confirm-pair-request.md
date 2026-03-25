# EOA Confirm Pair Request

Use this branch for the `EOA` direct-confirm path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`
- validated runtime version: `1.2.6`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose direct pair mode rather than queue mode
- the user wants to confirm an existing pending pair

## Required Facts

- Method: `ConfirmPairRequest(Address initiator)`
- Caller must be the pending counterparty
- All resonance `Get*` and other view-only reads in this flow must use the generic view path such as `portkey_call_view_method`, CLI `contract view`, or an SDK read call, never `portkey_call_send_method`
- Initiator input must be a valid on-chain `Address`
- `ConfirmPairRequest` requires an active pending pair
- `ConfirmPairRequest` requires the raw remaining reward pool to cover the maximum reward tier before randomness is sampled
- The contract uses the pending pair snapshots for the effective maximum reward
- `ConfirmPairRequest` is not blocked by `new_participation_available_time`; the warmup gate applies to new create/join participation only
- `GetCertificateStatus()` still reports `COMING_SOON`, but it may also carry strong-record payload when a strong resonance already exists

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Detect the local Portkey EOA skill version when runtime metadata or local manifest data is available; if the version still cannot be resolved reliably, omit `dependency_versions.portkey_eoa` from the default visible layer and explain the omission only in technical details.
4. Use the Portkey EOA skill to resolve the active local `EOA` signer.
5. Validate the initiator input as an on-chain `Address`.
6. Stop if the initiator address equals the resolved signer address.
7. Read `GetConfig()` through the generic view path, not through `portkey_call_send_method`.
8. Stop if the contract appears uninitialized.
9. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
10. Read `GetPairStatus()` for the unordered pair.
11. Read `GetPendingPair()` for the unordered pair.
12. Stop if `GetPairStatus().status == EXECUTED`.
13. Stop if there is no active pending pair.
14. Stop if the pending pair initiator does not match the requested initiator.
15. Stop if the resolved local signer does not match the pending pair counterparty.
16. Read `GetRemainingBalance()`.
17. Read `GetRewardBalance()` and `GetAvailableRewardBalance()` when practical for diagnostic context, but keep `GetRemainingBalance()` as the actual confirm-side gate.
18. Compute the baseline minimum pool check as `2 * (config.success_amount + config.strong_bonus_amount)`.
19. If pending snapshots are available, also compute the effective pair-specific minimum pool check as `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)`.
20. Stop if the remaining balance is lower than the effective pair-specific minimum pool check.
21. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on target contract address, whether a valid pending pair exists, how long it remains confirmable, and whether the confirm-side balance check passes
    - keep the raw execution address, active pending pair summary, raw balance check, optional reward-balance diagnostics, and other engineering fields in `Technical Details` unless the user explicitly asks for them
22. Ask for explicit confirmation.
23. Only after explicit confirmation, use the Portkey EOA skill to send `ConfirmPairRequest(initiator)`.
24. If a `txId` is returned, share the `txId` and explorer link.
25. Read `GetPairStatus()` again.
26. Read `GetAddressStats()` for both participant addresses when practical.
27. Read `GetStrongRecord()` for both participant addresses when practical.
28. Read `GetCertificateStatus()` for both participant addresses when practical.
29. If executed-state views fail because of an SDK decode issue, decode the `PairResonated` event and combine it with `GetPendingPair()` and `GetAddressStats()` as the final confirmation.
30. Return the read-after-write summary with outcome, `reward_each`, and any strong-record or certificate-payload updates.
31. Append the community CTA because the confirm path returned a clear result.

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

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, initiator, whether the write can proceed, pending validity window, the confirm-side balance conclusion, and explicit confirmation request
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- localized technical-details layer on demand with chosen flow, resolved signer, target raw execution address, method, current window and reward tiers, active pending pair summary, `GetRemainingBalance`, optional `GetRewardBalance` and `GetAvailableRewardBalance`, and pool checks

## Example Reference

Read [../examples/eoa-confirm-pair-request.md](../examples/eoa-confirm-pair-request.md) before replying.
