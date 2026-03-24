# EOA Leave Pair Queue

Use this branch for the `EOA` queue-leave path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `LeavePairQueue`

## Required Facts

- Method: `LeavePairQueue()`
- Only the queued address itself can actively leave its queue entry
- `LeavePairQueue()` requires the caller to still have an active queue entry
- If the queue entry already expired, matched, or was evicted, `GetPairQueueStatus()` will already show `in_queue == false`

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Use the Portkey EOA skill to resolve the active local `EOA` signer.
4. Read `GetConfig()`.
5. Stop if the contract appears uninitialized.
6. Record `request_expire_seconds` and `queue_capacity` from `GetConfig()`.
7. Read `GetPairQueueStatus()` for the caller address.
8. Read `GetPairQueueStats()` when practical.
9. Stop if the caller is not currently in the queue, and explain in plain language that the entry may already have expired, matched, been actively left before, or been evicted when the queue was full.
10. Show the pre-send summary using the output contract, including queue timeout, current queue status, queue stats, and `user_explanation`.
11. Ask for explicit confirmation.
12. Only after explicit confirmation, use the Portkey EOA skill to send `LeavePairQueue()`.
13. If a `txId` is returned, share the `txId` and explorer link.
14. Read `GetPairQueueStatus()` again for the caller address.
15. Read `GetPairQueueStats()` again when practical.
16. Infer the removal reason from the transaction result or event logs when possible.
17. Return the read-after-write summary with post-leave queue state.
18. Append the community CTA because the queue leave returned a clear non-error result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the caller is not currently in the pair queue

## Output Shape

The response before sending should contain:

- chosen flow: `EOA Leave Pair Queue`
- resolved signer
- target resonance contract in normalized full-address and raw-address form
- method `LeavePairQueue()`
- current `GetPairQueueStatus`
- `queue_timeout_seconds`
- `queue_timeout_humanized`
- `GetPairQueueStats` when available
- `user_explanation` that translates timeout and current queue state into ordinary language
- explicit confirmation request

## Example Reference

Read [../examples/eoa-leave-pair-queue.md](../examples/eoa-leave-pair-queue.md) before replying.
