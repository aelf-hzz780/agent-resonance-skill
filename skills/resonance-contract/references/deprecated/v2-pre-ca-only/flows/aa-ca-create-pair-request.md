# AA/CA Create Pair Request

Use this branch for the `AA/CA` direct-create path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.3.0`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user chose direct pair mode rather than queue mode
- the user wants to initiate `CreatePairRequest`

## Required Facts

- Forwarded write path: `manager signer -> CA.ManagerForwardCall -> resonanceContract.CreatePairRequest(Address counterparty)`
- Caller at the resonance contract layer is the resolved `AA/CA` holder address, not the manager signer
- All resonance `Get*` and other view-only reads in this flow must use the direct view path such as `contract.<Method>.call(...)` or Portkey CA `view-call`, never `CA.ManagerForwardCall`
- Counterparty input must be a valid on-chain `Address`
- The pair cannot be self-paired
- An executed unordered pair cannot be created again
- An active pending pair cannot be created again
- One address can hold only one active participation state at a time: one active pending pair or one active queue entry
- New direct participation is blocked until `new_participation_available_time` when the contract is still warming up after an upgrade
- New direct pending pairs reserve their maximum reward exposure up front, so create-side preflight must use `GetRewardBalance()` or `GetAvailableRewardBalance()`
- An expired old pending pair can be auto-cleared and recreated in one transaction

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, `resonance_contract_address`, and `portkey_ca_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local `AA/CA` holder address, `caHash`, and a usable manager signer.
5. If recovery or manager switching happened in the same session, query holder info on the target execution chain and stop until the chosen manager is visible there.
6. Validate the counterparty input as an on-chain `Address`.
7. Stop if the counterparty address equals the resolved `AA/CA` holder address.
8. Read `GetConfig()` through the direct view path, not through `CA.ManagerForwardCall`.
9. Stop if the contract appears uninitialized.
10. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
11. Stop if `new_participation_available_time` is missing or unset on an otherwise initialized contract, and explain that this is an abnormal state or decode issue first; only frame it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
12. Stop if `new_participation_available_time` is still in the future, and explain in plain language that new direct pairs and queue joins are warming up after an upgrade.
13. Read `GetPairStatus()` for the unordered pair using `AA/CA holder address` and `counterparty`.
14. Read `GetPendingPair()` for the unordered pair.
15. Read `GetActivePendingPair()` for the `AA/CA` holder address.
16. Read `GetActivePendingPair()` for the counterparty address.
17. Read `GetPairQueueStatus()` for the `AA/CA` holder address.
18. Read `GetPairQueueStatus()` for the counterparty address.
19. Stop if `GetPairStatus().status == EXECUTED`.
20. Stop if `GetPairStatus().status == PENDING` or `GetPendingPair()` returns an active pending pair for the unordered pair.
21. Stop if either address already has another active pending pair, and preserve the exact contract meaning that one address cannot start another direct request while an active pair request already exists.
22. Stop if either address is already in the pair queue, and explain the direct-versus-queue exclusivity rule in ordinary language.
23. If a stale pending pair exists but is expired, explain that `CreatePairRequest` can auto-clear it in the same transaction.
24. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
25. Compute the create-side maximum reservation check as `2 * (config.success_amount + config.strong_bonus_amount)`.
26. Stop if the available reward balance is lower than the create-side maximum reservation check.
27. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary; when the blocker is real and the agent cannot continue automatically, append the support CTA in the default layer.
28. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on caller identity, target counterparty, target contract address, whether the write can proceed, timeout, queue or pending conflicts, and the balance conclusion
    - surface `dependency_mode` in the default layer only when compatibility mode or runtime-metadata reliability materially affects the current reply
    - keep the raw execution address, target CA contract, forwarded method chain, pair-state reads, queue-state reads, reward-balance reads, and other engineering fields in `Technical Details` unless the user explicitly asks for them
29. Ask for explicit confirmation.
30. Only after explicit confirmation, use the Portkey CA skill to send the forwarded `CreatePairRequest(counterparty)` call.
31. If a `txId` is returned, share the `txId` and explorer link.
32. Read `GetPairStatus()` again using the `AA/CA` holder address and `counterparty`.
33. Read `GetPendingPair()` again.
34. Return the read-after-write summary with the new pending pair fields, including `window_end_time` when available.
35. Append the success CTA because the pending pair was successfully created.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- the target execution chain holder info does not yet include the chosen manager signer
- the counterparty input is not a valid on-chain `Address`
- the counterparty equals the resolved `AA/CA` holder address
- `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract
- the contract is still warming up for new participation
- the pair has already resonated
- the pair request is already pending
- the holder or counterparty already has another active pending pair
- the holder or counterparty is already in the pair queue
- the available reward balance is lower than the create-side maximum reservation check

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, counterparty, whether the write can proceed, timeout or pending-validity guidance, the main blocker or balance conclusion, and explicit confirmation request
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- localized technical-details layer on demand with chosen flow, manager signer, holder address, `caHash`, target raw execution address, target CA contract, dependency mode when relevant, forwarded method chain, current window and reward tiers, pair state, address-scoped pending and queue state, reward-balance reads, create-side maximum reservation check, and the expired-pending auto-clear note

## Example Reference

Read [../examples/aa-ca-create-pair-request.md](../examples/aa-ca-create-pair-request.md) before replying.
