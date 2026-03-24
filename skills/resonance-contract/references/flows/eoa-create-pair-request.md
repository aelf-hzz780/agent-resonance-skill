# EOA Create Pair Request

Use this branch for the `EOA` direct-create path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose direct pair mode rather than queue mode
- the user wants to initiate `CreatePairRequest`

## Required Facts

- Method: `CreatePairRequest(Address counterparty)`
- Caller becomes the pair initiator
- Counterparty input must be a valid on-chain `Address`
- The pair cannot be self-paired
- An executed unordered pair cannot be created again
- An active pending pair cannot be created again
- One address can hold only one active participation state at a time: one active pending pair or one active queue entry
- New direct participation is blocked until `new_participation_available_time` when the contract is still warming up after an upgrade
- New direct pending pairs reserve their maximum reward exposure up front, so create-side preflight must use `GetRewardBalance()` or `GetAvailableRewardBalance()`
- An expired old pending pair for the same unordered pair can be auto-cleared and recreated in one transaction

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Use the Portkey EOA skill to resolve the active local `EOA` signer.
4. Validate the counterparty input as an on-chain `Address`.
5. Stop if the counterparty address equals the resolved signer address.
6. Read `GetConfig()`.
7. Stop if the contract appears uninitialized, for example `token_symbol` is empty or `admin` is missing.
8. Record the current window, reward tiers, `request_expire_seconds`, `new_participation_available_time`, and `queue_capacity` from `GetConfig()`.
9. Stop if `new_participation_available_time` is missing or unset on an otherwise initialized contract, and explain that this is an abnormal state or decode issue first; only frame it as pre-finalize upgrade blocking when the deployment is known to be an upgraded legacy instance.
10. Stop if `new_participation_available_time` is still in the future, and explain in plain language that new direct pairs and queue joins are warming up after an upgrade.
11. Read `GetPairStatus()` for the unordered pair.
12. Read `GetPendingPair()` for the unordered pair.
13. Read `GetActivePendingPair()` for the caller address.
14. Read `GetActivePendingPair()` for the counterparty address.
15. Read `GetPairQueueStatus()` for the caller address.
16. Read `GetPairQueueStatus()` for the counterparty address.
17. Stop if `GetPairStatus().status == EXECUTED`.
18. Stop if `GetPairStatus().status == PENDING` or `GetPendingPair()` returns an active pending pair for the unordered pair.
19. Stop if either address already has another active pending pair, and preserve the exact contract meaning that one address cannot start another direct request while an active pair request already exists.
20. Stop if either address is already in the pair queue, and explain the direct-versus-queue exclusivity rule in ordinary language.
21. If a stale pending pair exists but is expired, explain that `CreatePairRequest` can auto-clear it in the same transaction.
22. Read `GetRewardBalance()` when practical and always read `GetAvailableRewardBalance()`.
23. Compute the create-side maximum reservation check as `2 * (config.success_amount + config.strong_bonus_amount)`.
24. Stop if the available reward balance is lower than the create-side maximum reservation check.
25. Show the pre-send summary using the output contract, including normalized contract addresses, pair-state reads, queue-state reads, reward-balance reads, queue timeout, and `user_explanation`.
26. Ask for explicit confirmation.
27. Only after explicit confirmation, use the Portkey EOA skill to send `CreatePairRequest(counterparty)`.
28. If a `txId` is returned, share the `txId` and explorer link.
29. Read `GetPairStatus()` again.
30. Read `GetPendingPair()` again.
31. Return the read-after-write summary with the new pending pair fields, including `window_end_time` when available.
32. Append the community CTA because the pending pair was successfully created.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the counterparty input is not a valid on-chain `Address`
- the counterparty equals the caller
- `new_participation_available_time` is unexpectedly missing on an otherwise initialized contract
- the contract is still warming up for new participation
- the pair has already resonated
- the pair request is already pending
- the caller or counterparty already has another active pending pair
- the caller or counterparty is already in the pair queue
- the available reward balance is lower than the create-side maximum reservation check

## Output Shape

The response before sending should contain:

- chosen flow: `EOA Create Pair Request`
- resolved signer
- counterparty
- target resonance contract in normalized full-address and raw-address form
- method `CreatePairRequest(Address counterparty)`
- current window and reward tiers
- current pair state
- `GetActivePendingPair` for caller and counterparty when practical
- `GetPairQueueStatus` for caller and counterparty when practical
- `GetRewardBalance` or `GetAvailableRewardBalance`
- create-side maximum reservation check
- expired-pending auto-clear note when relevant
- `user_explanation` that translates timeout, exclusivity, warmup, and queue-full implications into ordinary language when relevant
- explicit confirmation request

## Example Reference

Read [../examples/eoa-create-pair-request.md](../examples/eoa-create-pair-request.md) before replying.
