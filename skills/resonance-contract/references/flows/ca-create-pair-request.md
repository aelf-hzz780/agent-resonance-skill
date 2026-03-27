# CA Create Pair Request

Use this branch for the CA direct-create path after local CA context and participation-mode choice are complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.3.0`

## When To Use

Use this flow only when all conditions below are true:

- a local `CA` context is already available
- the user chose direct pair mode rather than queue mode
- the user wants to initiate `CreatePairRequestByCa`
- the user provided or confirmed `counterparty_ca_hash`

## Required Facts

- Current write path: `relayer wallet -> resonanceContract.CreatePairRequestByCa(CreatePairRequestByCaInput)`
- Input fields: `ca_hash`, `counterparty_ca_hash`
- All resonance `Get*` and other view-only reads in this flow must use the direct view path
- Direct-mode counterparty input must be a valid `ca_hash`
- The pair cannot be self-paired after both sides resolve to the same `ca_address`
- An executed unordered pair cannot be created again
- An active pending pair cannot be created again
- One `ca_address` can hold only one active participation state at a time: one active pending pair or one active queue entry
- New direct participation is blocked until `new_participation_available_time` when the contract is still warming up after an upgrade
- New direct pending pairs reserve their maximum reward exposure up front, so create-side preflight must use `GetRewardBalance()` or `GetAvailableRewardBalance()`
- An expired old pending pair can be auto-cleared and recreated in one transaction
- `GetConfig().portkey_ca_contract_address` must be present before any write can proceed

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local caller `CA` context, including `caller_ca_hash`, `caller_ca_address`, and a usable local relayer or signer for the current write.
5. If multiple local `CA` accounts are available and the caller identity is still ambiguous, stop this write path and ask which local `CA` account should be used.
6. Validate the user-supplied `counterparty_ca_hash`.
7. If the counterparty input looks like an on-chain `Address` instead of a `ca_hash`, stop the write path and correct the input expectation in plain language.
8. Read `GetConfig()` through the direct view path.
9. Stop if the contract appears uninitialized.
10. Stop if `GetConfig().portkey_ca_contract_address` is missing.
11. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, `queue_capacity`, and `portkey_ca_contract_address` from `GetConfig()`.
12. Stop if `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract.
13. Stop if `new_participation_available_time` is still in the future.
14. Resolve `counterparty_ca_hash` into `counterparty_ca_address` through the configured Portkey CA contract.
15. Stop if the resolved `counterparty_ca_address` equals `caller_ca_address`.
16. Read `GetPairStatus()` for the unordered pair using `caller_ca_address` and `counterparty_ca_address`.
17. Read `GetPendingPair()` for the unordered pair.
18. Read `GetActivePendingPair()` for the caller address.
19. Read `GetActivePendingPair()` for the counterparty address.
20. Read `GetPairQueueStatus()` for the caller address.
21. Read `GetPairQueueStatus()` for the counterparty address.
22. Stop if `GetPairStatus().status == EXECUTED`.
23. If the unordered pair is `PENDING`, inspect `GetPendingPair()` carefully:
    - stop only when the current pending pair is still active
    - if the current pending pair is expired, explain that `CreatePairRequestByCa` can auto-clear and recreate it in the same transaction
24. Stop if either address already has another active pending pair.
25. Stop if either address is already in the pair queue.
26. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
27. Compute the create-side maximum reservation check as `2 * (config.success_amount + config.strong_bonus_amount)`.
28. Stop if the available reward balance is lower than the create-side maximum reservation check.
29. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary.
30. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions.portkey_ca`
    - keep the default layer focused on caller identity, target counterparty, whether the write can proceed, timeout, queue or pending conflicts, and the balance conclusion
    - keep raw contract addresses, `caller_ca_hash`, `caller_ca_address`, `counterparty_ca_hash`, `counterparty_ca_address`, exact pair-state reads, queue-state reads, and reward-balance reads in `Technical Details` unless the user explicitly asks for them
31. Ask for explicit confirmation.
32. Only after explicit confirmation, send `CreatePairRequestByCa({ caHash, counterpartyCaHash })`.
33. If a `txId` is returned, share the `txId` and explorer link.
34. Read `GetPairStatus()` again using the resolved pair addresses.
35. Read `GetPendingPair()` again.
36. Return the read-after-write summary with the new pending pair fields, including `window_end_time` when available.
37. Append the success CTA because the pending pair was successfully created.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local caller `CA` context cannot be resolved
- no usable local relayer or signer can be resolved for the current write
- multiple local `CA` accounts are available but the caller was not chosen yet
- counterparty input is not a valid `ca_hash`
- counterparty input is an on-chain `Address` rather than a `ca_hash`
- `GetConfig().portkey_ca_contract_address` is missing
- counterparty `ca_hash` cannot be resolved into `ca_address`
- the resolved counterparty equals the caller
- `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract
- the contract is still warming up for new participation
- the pair has already resonated
- the pair request is already pending and still active
- the caller or counterparty already has another active pending pair
- the caller or counterparty is already in the pair queue
- the available reward balance is lower than the create-side maximum reservation check

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions.portkey_ca`, caller identity, counterparty identity, whether the write can proceed, timeout or pending-validity guidance, the main blocker or balance conclusion, and explicit confirmation request
- localized technical-details layer on demand with chosen flow, `caller_ca_hash`, `caller_ca_address`, `counterparty_ca_hash`, `counterparty_ca_address`, target raw execution address, current config, pair state, address-scoped pending and queue state, reward-balance reads, create-side maximum reservation check, and the expired-pending auto-clear note

## Example Reference

Read [../examples/ca-create-pair-request.md](../examples/ca-create-pair-request.md) before replying.
