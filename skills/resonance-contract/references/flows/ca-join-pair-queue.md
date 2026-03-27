# CA Join Pair Queue

Use this branch for the CA queue-join path.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.3.0`

## When To Use

Use this flow only when all conditions below are true:

- a local `CA` context is already available
- the user wants automatic matching or queue participation

## Required Facts

- Current write path: `relayer wallet -> resonanceContract.JoinPairQueueByCa(JoinPairQueueByCaInput)`
- Input fields: `ca_hash`, optional `selection_policy`
- All resonance `Get*` and other view-only reads in this flow must use the direct view path
- Queue participation is keyed by resolved `ca_address`
- New participation is blocked until `new_participation_available_time` when the contract is still warming up after an upgrade
- `JoinPairQueueByCa` has two balance paths:
  - immediate match checks `GetRemainingBalance()`
  - queued entry checks `GetAvailableRewardBalance()` / `GetRewardBalance().available_balance`
- `GetConfig().portkey_ca_contract_address` must be present before any write can proceed

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize `resonance_contract_address`.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local caller `CA` context, including `caller_ca_hash`, `caller_ca_address`, and a usable local relayer or signer for the current write.
5. If multiple local `CA` accounts are available and the caller identity is still ambiguous, stop this write path and ask which local `CA` account should be used.
6. Read `GetConfig()`.
7. Stop if the contract appears uninitialized.
8. Stop if `GetConfig().portkey_ca_contract_address` is missing.
9. Record the current timeout, queue capacity, reward tiers, and warmup fields from `GetConfig()`.
10. Stop if `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract.
11. Stop if `new_participation_available_time` is still in the future.
12. Read `GetActivePendingPair(caller_ca_address)`.
13. Read `GetPairQueueStatus(caller_ca_address)`.
14. Stop if the caller already has an active pending pair.
15. Stop if the caller is already in the queue.
16. Read `GetPairQueueStats()`.
17. Read `GetRemainingBalance()`.
18. Read `GetRewardBalance()` and `GetAvailableRewardBalance()`.
19. Compute the join-side maximum reward check as `2 * (config.success_amount + config.strong_bonus_amount)`.
20. Stop if the remaining balance is lower than the join-side maximum reward check.
21. Note whether the current balances guarantee both outcomes or only keep the immediate-match path open.
22. If `GetPairQueueStats().queue_capacity == 0` but `GetConfig().queue_capacity > 0`, explain that the current queue-capacity source of truth is `GetConfig().queue_capacity`.
23. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary.
24. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions.portkey_ca`
    - keep the default layer focused on caller identity, queue policy, timeout, whether the write can proceed, and the balance conclusion
    - keep raw contract addresses, `caller_ca_hash`, `caller_ca_address`, queue-state reads, queue stats, and detailed balance reads in `Technical Details` unless the user explicitly asks for them
25. Ask for explicit confirmation.
26. Only after explicit confirmation, send `JoinPairQueueByCa({ caHash, selectionPolicy? })`.
27. If a `txId` is returned, share the `txId` and explorer link.
28. Read `GetPairQueueStatus()` again when practical.
29. Return the post-send summary as either `immediate matched` or `queued`.
30. Append the success CTA because the join result is a clear non-error outcome.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local caller `CA` context cannot be resolved
- no usable local relayer or signer can be resolved for the current write
- multiple local `CA` accounts are available but the caller was not chosen yet
- `GetConfig().portkey_ca_contract_address` is missing
- `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract
- the contract is still warming up for new participation
- the caller already has an active pending pair
- the caller is already in the queue
- the remaining reward balance is lower than the join-side minimum reward threshold

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions.portkey_ca`, caller identity, queue policy, whether the write can proceed, timeout guidance, the main blocker or balance conclusion, and explicit confirmation request
- localized technical-details layer on demand with chosen flow, `caller_ca_hash`, `caller_ca_address`, target raw execution address, current config, queue status, queue stats, `GetRemainingBalance`, `GetRewardBalance`, `GetAvailableRewardBalance`, join-side maximum reward check, and whether the result may become immediate match or queued entry

## Example Reference

Read [../examples/ca-join-pair-queue.md](../examples/ca-join-pair-queue.md) before replying.
