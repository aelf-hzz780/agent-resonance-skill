# AA/CA Join Pair Queue

Use this branch for the `AA/CA` queue-join path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.2.0`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `JoinPairQueue`

## Required Facts

- Forwarded write path: `manager signer -> CA.ManagerForwardCall -> resonanceContract.JoinPairQueue(JoinPairQueueInput)`
- Caller at the resonance contract layer is the resolved `AA/CA` holder address, not the manager signer
- If the input is empty, default, or `selection_policy` is unspecified, the contract uses `FIFO`
- If `selection_policy == RANDOM`, the contract chooses from currently eligible queued addresses without first-come-first-served guarantees
- If an eligible queued address already exists, the contract may match and execute resonance immediately in the same transaction
- If no eligible queued address exists, the caller joins the queue
- One address can hold only one active participation state at a time: one active pending pair or one active queue entry
- `JoinPairQueue` has two balance paths:
  - if an eligible queued address is found and resonance executes immediately, the contract checks raw `remaining balance`
  - if the holder ends up joining the queue, the contract reserves against `available balance`
- New queue participation requires current time to be at or after `new_participation_available_time`
- If the queue is already full and this call still cannot match immediately, the contract evicts the earliest still-valid queued address before the caller joins

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, `resonance_contract_address`, and `portkey_ca_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local `AA/CA` holder address, `caHash`, and a usable manager signer.
5. If recovery or manager switching happened in the same session, query holder info on the target execution chain and stop until the chosen manager is visible there.
6. Resolve the requested queue selection policy.
7. If the user did not specify a policy, keep the input empty or default and explain that the effective selection policy will be `FIFO`.
8. Read `GetConfig()`.
9. Stop if the contract appears uninitialized.
10. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
11. Stop if `new_participation_available_time` is missing or unset on an otherwise initialized contract, and explain that this is an abnormal state or decode issue first; only frame it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
12. Stop if `new_participation_available_time` is still in the future, and explain in plain language that new direct pairs and queue joins are warming up after an upgrade.
13. Read `GetActivePendingPair()` for the `AA/CA` holder address.
14. Read `GetPairQueueStatus()` for the `AA/CA` holder address.
15. Stop if the holder already has an active pending pair.
16. Stop if the holder is already in the pair queue.
17. Read `GetRemainingBalance()`.
18. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
19. Read `GetPairQueueStats()` when practical.
20. Compute the join-side maximum reward check as `2 * (config.success_amount + config.strong_bonus_amount)`.
21. Stop if the remaining balance is lower than the join-side maximum reward check, because neither the immediate-match path nor the queued path can succeed.
22. If the available reward balance is lower than the join-side maximum reward check but the remaining balance is still sufficient, explain that an immediate match may still succeed while a pure queue-enqueue result is not guaranteed by preflight.
23. Show the pre-send summary using the output contract, including normalized contract addresses, dependency mode, forwarded method chain, queue policy, queue timeout, queue capacity, remaining-balance and reward-balance reads, the join-side maximum reward check, and `user_explanation`.
24. Ask for explicit confirmation.
25. Only after explicit confirmation, use the Portkey CA skill to send the forwarded `JoinPairQueue(input)` call.
26. If a `txId` is returned, share the `txId` and explorer link.
27. Inspect the transaction result and logs.
28. If a `PairResonated` event is present, treat the result as immediate match and summarize the matched addresses, `outcome`, `random_number`, `reward_each`, and `executed_time`.
29. If the holder remained queued, read `GetPairQueueStatus()` for the holder and `GetPairQueueStats()` again.
30. Read `GetAddressStats()`, `GetStrongRecord()`, and `GetCertificateStatus()` when practical if the join matched immediately.
31. Return the read-after-write summary as either `queued` or `immediate match`.
32. Append the community CTA because the queue join returned a clear non-error result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- the target execution chain holder info does not yet include the chosen manager signer
- the contract is still warming up for new participation
- the holder already has an active pending pair
- the holder is already in the pair queue
- the remaining balance is lower than the join-side maximum reward check

## Output Shape

The response before sending should contain:

- chosen flow: `AA/CA Join Pair Queue`
- manager signer
- resolved `AA/CA` holder address
- `caHash` when available
- target resonance contract in normalized full-address and raw-address form
- target CA contract
- dependency mode and detected Portkey CA skill version when practical
- method chain `ManagerForwardCall -> JoinPairQueue(JoinPairQueueInput)`
- current window and reward tiers
- queue selection policy
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetActivePendingPair` for the holder
- `GetPairQueueStatus` for the holder
- `GetPairQueueStats` when available
- `GetRemainingBalance`
- `GetRewardBalance` or `GetAvailableRewardBalance`
- join-side maximum reward check
- note when the current balances still leave open only the immediate-match path
- `user_explanation` that translates timeout, FIFO or RANDOM behavior, queue-full eviction, exclusivity, and warmup into ordinary language
- explicit confirmation request

## Example Reference

Read [../examples/aa-ca-join-pair-queue.md](../examples/aa-ca-join-pair-queue.md) before replying.
