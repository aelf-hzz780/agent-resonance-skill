# EOA Join Pair Queue

Use this branch for the `EOA` queue-join path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`
- validated runtime version: `1.2.6`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `JoinPairQueue`

## Required Facts

- Method: `JoinPairQueue(JoinPairQueueInput)`
- If the input is empty, default, or `selection_policy` is unspecified, the contract uses `FIFO`
- All resonance `Get*` and other view-only reads in this flow must use the generic view path such as `portkey_call_view_method`, CLI `contract view`, or an SDK read call, never `portkey_call_send_method`
- If `selection_policy == RANDOM`, the contract chooses from currently eligible queued addresses without first-come-first-served guarantees
- If an eligible queued address already exists, the contract may match and execute resonance immediately in the same transaction
- If no eligible queued address exists, the caller joins the queue
- One address can hold only one active participation state at a time: one active pending pair or one active queue entry
- `JoinPairQueue` has two balance paths:
  - if an eligible queued address is found and resonance executes immediately, the contract checks raw `remaining balance`
  - if the caller ends up joining the queue, the contract reserves against `available balance`
- New queue participation requires current time to be at or after `new_participation_available_time`
- If the queue is already full and this call still cannot match immediately, the contract evicts the earliest still-valid queued address before the caller joins

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Detect the local Portkey EOA skill version when runtime metadata or local manifest data is available; if the version still cannot be resolved reliably, omit `dependency_versions.portkey_eoa` from the default visible layer and explain the omission only in technical details.
4. Use the Portkey EOA skill to resolve the active local `EOA` signer.
5. Resolve the requested queue selection policy.
6. If the user did not specify a policy, keep the input empty or default and explain that the effective selection policy will be `FIFO`.
7. Read `GetConfig()` through the generic view path, not through `portkey_call_send_method`.
8. Stop if the contract appears uninitialized.
9. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
10. Stop if `new_participation_available_time` is missing or unset on an otherwise initialized contract, and explain that this is an abnormal state or decode issue first; only frame it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
11. Stop if `new_participation_available_time` is still in the future, and explain in plain language that new direct pairs and queue joins are warming up after an upgrade.
12. Read `GetActivePendingPair()` for the caller address.
13. Read `GetPairQueueStatus()` for the caller address.
14. Stop if the caller already has an active pending pair.
15. Stop if the caller is already in the pair queue.
16. Read `GetRemainingBalance()`.
17. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
18. Read `GetPairQueueStats()` when practical.
19. Compute the join-side maximum reward check as `2 * (config.success_amount + config.strong_bonus_amount)`.
20. Stop if the remaining balance is lower than the join-side maximum reward check, because neither the immediate-match path nor the queued path can succeed.
21. If the available reward balance is lower than the join-side maximum reward check but the remaining balance is still sufficient, explain that an immediate match may still succeed while a pure queue-enqueue result is not guaranteed by preflight.
22. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary; when the blocker is real and the agent cannot continue automatically, append the support CTA in the default layer.
23. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on target contract address, queue policy in plain language, timeout, whether the write can proceed, and whether the likely result is immediate match or queued entry
    - keep the raw execution address, queue stats, remaining-balance and reward-balance reads, the join-side maximum reward check, and other engineering fields in `Technical Details` unless the user explicitly asks for them
24. Ask for explicit confirmation.
25. Only after explicit confirmation, use the Portkey EOA skill to send `JoinPairQueue(input)`.
26. If a `txId` is returned, share the `txId` and explorer link.
27. Inspect the transaction result and logs.
28. If a `PairResonated` event is present, treat the result as immediate match and summarize the matched addresses, `outcome`, `random_number`, `reward_each`, and `executed_time`.
29. If the caller remained queued, read `GetPairQueueStatus()` for the caller and `GetPairQueueStats()` again.
30. Read `GetAddressStats()`, `GetStrongRecord()`, and `GetCertificateStatus()` when practical if the join matched immediately.
31. Return the read-after-write summary as either `queued` or `immediate match`.
32. Append the success CTA because the queue join returned a clear non-error result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the contract is still warming up for new participation
- the caller already has an active pending pair
- the caller is already in the pair queue
- the remaining balance is lower than the join-side maximum reward check

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, default or requested queue policy, timeout, whether the write can proceed, the main balance conclusion, likely outcome guidance, and explicit confirmation request
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- localized technical-details layer on demand with chosen flow, resolved signer, target raw execution address, method, current window and reward tiers, queue reads, queue stats, reward-balance reads, the join-side maximum reward check, and the immediate-match-only note when relevant

## Example Reference

Read [../examples/eoa-join-pair-queue.md](../examples/eoa-join-pair-queue.md) before replying.
