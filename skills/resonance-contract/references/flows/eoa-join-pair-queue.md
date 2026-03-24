# EOA Join Pair Queue

Use this branch for the `EOA` queue-join path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `JoinPairQueue`

## Required Facts

- Method: `JoinPairQueue(JoinPairQueueInput)`
- If the input is empty, default, or `selection_policy` is unspecified, the contract uses `FIFO`
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
3. Use the Portkey EOA skill to resolve the active local `EOA` signer.
4. Resolve the requested queue selection policy.
5. If the user did not specify a policy, keep the input empty or default and explain that the effective selection policy will be `FIFO`.
6. Read `GetConfig()`.
7. Stop if the contract appears uninitialized.
8. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
9. Stop if `new_participation_available_time` is missing or unset on an otherwise initialized contract, and explain that this is an abnormal state or decode issue first; only frame it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
10. Stop if `new_participation_available_time` is still in the future, and explain in plain language that new direct pairs and queue joins are warming up after an upgrade.
11. Read `GetActivePendingPair()` for the caller address.
12. Read `GetPairQueueStatus()` for the caller address.
13. Stop if the caller already has an active pending pair.
14. Stop if the caller is already in the pair queue.
15. Read `GetRemainingBalance()`.
16. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
17. Read `GetPairQueueStats()` when practical.
18. Compute the join-side maximum reward check as `2 * (config.success_amount + config.strong_bonus_amount)`.
19. Stop if the remaining balance is lower than the join-side maximum reward check, because neither the immediate-match path nor the queued path can succeed.
20. If the available reward balance is lower than the join-side maximum reward check but the remaining balance is still sufficient, explain that an immediate match may still succeed while a pure queue-enqueue result is not guaranteed by preflight.
21. Show the pre-send summary using the output contract, including queue policy, queue timeout, queue capacity, remaining-balance and reward-balance reads, the join-side maximum reward check, and `user_explanation`.
22. Ask for explicit confirmation.
23. Only after explicit confirmation, use the Portkey EOA skill to send `JoinPairQueue(input)`.
24. If a `txId` is returned, share the `txId` and explorer link.
25. Inspect the transaction result and logs.
26. If a `PairResonated` event is present, treat the result as immediate match and summarize the matched addresses, `outcome`, `random_number`, `reward_each`, and `executed_time`.
27. If the caller remained queued, read `GetPairQueueStatus()` for the caller and `GetPairQueueStats()` again.
28. Read `GetAddressStats()`, `GetStrongRecord()`, and `GetCertificateStatus()` when practical if the join matched immediately.
29. Return the read-after-write summary as either `queued` or `immediate match`.
30. Append the community CTA because the queue join returned a clear non-error result.

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

- chosen flow: `EOA Join Pair Queue`
- resolved signer
- target resonance contract in normalized full-address and raw-address form
- method `JoinPairQueue(JoinPairQueueInput)`
- current window and reward tiers
- queue selection policy
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetActivePendingPair` for the caller
- `GetPairQueueStatus` for the caller
- `GetPairQueueStats` when available
- `GetRemainingBalance`
- `GetRewardBalance` or `GetAvailableRewardBalance`
- join-side maximum reward check
- note when the current balances still leave open only the immediate-match path
- `user_explanation` that translates timeout, FIFO or RANDOM behavior, queue-full eviction, exclusivity, and warmup into ordinary language
- explicit confirmation request

## Example Reference

Read [../examples/eoa-join-pair-queue.md](../examples/eoa-join-pair-queue.md) before replying.
