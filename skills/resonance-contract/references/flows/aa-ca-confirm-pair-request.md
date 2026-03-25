# AA/CA Confirm Pair Request

Use this branch for the `AA/CA` direct-confirm path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.2.0`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user chose direct pair mode rather than queue mode
- the user wants to confirm an existing pending pair

## Required Facts

- Forwarded write path: `manager signer -> CA.ManagerForwardCall -> resonanceContract.ConfirmPairRequest(Address initiator)`
- Caller at the resonance contract layer is the resolved `AA/CA` holder address, not the manager signer
- Initiator input must be a valid on-chain `Address`
- Caller must be the pending counterparty
- `ConfirmPairRequest` requires an active pending pair
- `ConfirmPairRequest` requires the raw remaining reward pool to cover the maximum reward tier before randomness is sampled
- `ConfirmPairRequest` is not blocked by `new_participation_available_time`; the warmup gate applies to new create/join participation only
- `GetCertificateStatus()` still reports `COMING_SOON`, but it may also carry strong-record payload when a strong resonance already exists

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, `resonance_contract_address`, and `portkey_ca_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local `AA/CA` holder address, `caHash`, and a usable manager signer.
5. If recovery or manager switching happened in the same session, query holder info on the target execution chain and stop until the chosen manager is visible there.
6. Validate the initiator input as an on-chain `Address`.
7. Stop if the initiator address equals the resolved `AA/CA` holder address.
8. Read `GetConfig()`.
9. Stop if the contract appears uninitialized.
10. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
11. Read `GetPairStatus()` for the unordered pair using `AA/CA holder address` and `initiator`.
12. Read `GetPendingPair()` for the unordered pair.
13. Stop if `GetPairStatus().status == EXECUTED`.
14. Stop if there is no active pending pair.
15. Stop if the pending pair initiator does not match the requested initiator.
16. Stop if the resolved `AA/CA` holder address does not match the pending pair counterparty.
17. Read `GetRemainingBalance()`.
18. Read `GetRewardBalance()` and `GetAvailableRewardBalance()` when practical for diagnostic context, but keep `GetRemainingBalance()` as the actual confirm-side gate.
19. Compute the baseline minimum pool check as `2 * (config.success_amount + config.strong_bonus_amount)`.
20. If pending snapshots are available, also compute the effective pair-specific minimum pool check as `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)`.
21. Stop if the remaining balance is lower than the effective pair-specific minimum pool check.
22. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on target contract address, whether a valid pending pair exists, how long it remains confirmable, and whether the confirm-side balance check passes
    - surface `dependency_mode` in the default layer only when compatibility mode or runtime-metadata reliability materially affects the current reply
    - keep the raw execution address, target CA contract, forwarded method chain, active pending pair summary, raw balance check, optional reward-balance diagnostics, and other engineering fields in `Technical Details` unless the user explicitly asks for them
23. Ask for explicit confirmation.
24. Only after explicit confirmation, use the Portkey CA skill to send the forwarded `ConfirmPairRequest(initiator)` call.
25. If a `txId` is returned, share the `txId` and explorer link.
26. Read `GetPairStatus()` again.
27. Read `GetAddressStats()` for both participant addresses when practical.
28. Read `GetStrongRecord()` for both participant addresses when practical.
29. Read `GetCertificateStatus()` for both participant addresses when practical.
30. If executed-state views fail because of an SDK decode issue, decode the `PairResonated` event and combine it with `GetPendingPair()` and `GetAddressStats()` as the final confirmation.
31. Return the read-after-write summary with outcome, `reward_each`, and any strong-record or certificate-payload updates.
32. Append the community CTA because the confirm path returned a clear result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- the target execution chain holder info does not yet include the chosen manager signer
- the initiator input is not a valid on-chain `Address`
- the initiator equals the resolved `AA/CA` holder address
- the pair has already resonated
- there is no active pending pair
- the pending pair initiator does not match the requested initiator
- the resolved `AA/CA` holder address is not the pending counterparty
- the remaining balance is lower than the effective pair-specific minimum pool check

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, initiator, target normalized full `resonance_contract_address`, whether the write can proceed, pending validity window, the confirm-side balance conclusion, and explicit confirmation request
- localized technical-details layer on demand with chosen flow, manager signer, holder address, `caHash`, target raw execution address, target CA contract, dependency mode when relevant, forwarded method chain, current window and reward tiers, active pending pair summary, `GetRemainingBalance`, optional `GetRewardBalance` and `GetAvailableRewardBalance`, pool checks, and supporting `user_explanation`

## Example Reference

Read [../examples/aa-ca-confirm-pair-request.md](../examples/aa-ca-confirm-pair-request.md) before replying.
