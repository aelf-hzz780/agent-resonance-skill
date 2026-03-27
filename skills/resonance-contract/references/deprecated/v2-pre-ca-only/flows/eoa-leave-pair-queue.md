# EOA Leave Pair Queue

Use this branch for the `EOA` queue-leave path after account choice and participation-mode choice are complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`
- validated runtime version: `1.2.6`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user chose queue mode rather than direct pair mode
- the user wants to initiate `LeavePairQueue`

## Required Facts

- Method: `LeavePairQueue()`
- Only the queued address itself can actively leave its queue entry
- All resonance `Get*` and other view-only reads in this flow must use the generic view path such as `portkey_call_view_method`, CLI `contract view`, or an SDK read call, never `portkey_call_send_method`
- `LeavePairQueue()` requires the caller to still have an active queue entry
- If the queue entry already expired, matched, or was evicted, `GetPairQueueStatus()` will already show `in_queue == false`

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize the incoming contract address into display full-address form and raw execution form.
3. Detect the local Portkey EOA skill version when runtime metadata or local manifest data is available; if the version still cannot be resolved reliably, omit `dependency_versions.portkey_eoa` from the default visible layer and explain the omission only in technical details.
4. Use the Portkey EOA skill to resolve the active local `EOA` signer.
5. Read `GetConfig()` through the generic view path, not through `portkey_call_send_method`.
6. Stop if the contract appears uninitialized.
7. Record `request_expire_seconds` and `queue_capacity` from `GetConfig()`.
8. Read `GetPairQueueStatus()` for the caller address.
9. Read `GetPairQueueStats()` when practical.
10. Stop if the caller is not currently in the queue, and explain in plain language that the entry may already have expired, matched, been actively left before, or been evicted when the queue was full.
11. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary; when the blocker is real and the agent cannot continue automatically, append the support CTA in the default layer.
12. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions`
    - keep the default layer focused on target contract address, whether the caller still appears to be queued, what leaving means in plain language, and whether the action can still do anything useful
    - keep the raw execution address, queue status, queue stats, and supporting engineering context in `Technical Details` unless the user explicitly asks for them
13. Ask for explicit confirmation.
14. Only after explicit confirmation, use the Portkey EOA skill to send `LeavePairQueue()`.
15. If a `txId` is returned, share the `txId` and explorer link.
16. Read `GetPairQueueStatus()` again for the caller address.
17. Read `GetPairQueueStats()` again when practical.
18. Infer the removal reason from the transaction result or event logs when possible.
19. Return the read-after-write summary with post-leave queue state.
20. Append the success CTA because the queue leave returned a clear non-error result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the caller is not currently in the pair queue

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions`, caller identity, whether the caller still appears queued, timeout guidance when relevant, the practical effect of leaving now, and explicit confirmation request
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- localized technical-details layer on demand with chosen flow, resolved signer, target raw execution address, method, current queue status, queue timeout fields, and queue stats

## Example Reference

Read [../examples/eoa-leave-pair-queue.md](../examples/eoa-leave-pair-queue.md) before replying.
