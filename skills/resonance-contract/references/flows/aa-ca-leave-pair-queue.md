# AA/CA Leave Pair Queue

Use this branch for the `AA/CA` queue-leave path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.2.0`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `LeavePairQueue`

## Required Facts

- Forwarded write path: `manager signer -> CA.ManagerForwardCall -> resonanceContract.LeavePairQueue()`
- Caller at the resonance contract layer is the resolved `AA/CA` holder address, not the manager signer
- All resonance `Get*` and other view-only reads in this flow must use the direct view path such as `contract.<Method>.call(...)` or Portkey CA `view-call`, never `CA.ManagerForwardCall`
- Only the queued address itself can actively leave its queue entry
- `LeavePairQueue()` requires the caller to still have an active queue entry
- If the queue entry already expired, matched, or was evicted, `GetPairQueueStatus()` will already show `in_queue == false`

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, `resonance_contract_address`, and `portkey_ca_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local `AA/CA` holder address, `caHash`, and a usable manager signer.
5. If recovery or manager switching happened in the same session, query holder info on the target execution chain and stop until the chosen manager is visible there.
6. Read `GetConfig()` through the direct view path, not through `CA.ManagerForwardCall`.
7. Stop if the contract appears uninitialized.
8. Record `request_expire_seconds` and `queue_capacity` from `GetConfig()`.
9. Read `GetPairQueueStatus()` for the `AA/CA` holder address.
10. Read `GetPairQueueStats()` when practical.
11. Stop if the holder is not currently in the queue, and explain in plain language that the entry may already have expired, matched, been actively left before, or been evicted when the queue was full.
12. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on target contract address, whether the holder still appears queued, what leaving means in plain language, and whether the action can still do anything useful
    - surface `dependency_mode` in the default layer only when compatibility mode or runtime-metadata reliability materially affects the current reply
    - keep the raw execution address, target CA contract, forwarded method chain, queue timeout, current queue status, queue stats, and other engineering fields in `Technical Details` unless the user explicitly asks for them
13. Ask for explicit confirmation.
14. Only after explicit confirmation, use the Portkey CA skill to send the forwarded `LeavePairQueue()` call.
15. If a `txId` is returned, share the `txId` and explorer link.
16. Read `GetPairQueueStatus()` again for the `AA/CA` holder address.
17. Read `GetPairQueueStats()` again when practical.
18. Infer the removal reason from the transaction result or event logs when possible.
19. Return the read-after-write summary with post-leave queue state.
20. Append the community CTA because the queue leave returned a clear non-error result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- the target execution chain holder info does not yet include the chosen manager signer
- the holder is not currently in the pair queue

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, whether the holder still appears queued, timeout guidance when relevant, the practical effect of leaving now, and explicit confirmation request
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- localized technical-details layer on demand with chosen flow, manager signer, holder address, `caHash`, target raw execution address, target CA contract, dependency mode when relevant, forwarded method chain, current queue status, queue timeout fields, and queue stats

## Example Reference

Read [../examples/aa-ca-leave-pair-queue.md](../examples/aa-ca-leave-pair-queue.md) before replying.
