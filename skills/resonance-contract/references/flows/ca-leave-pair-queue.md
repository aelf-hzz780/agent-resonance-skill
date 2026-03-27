# CA Leave Pair Queue

Use this branch for the CA queue-leave path.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.3.0`

## When To Use

Use this flow only when all conditions below are true:

- a local `CA` context is already available
- the user wants to leave the automatic matching queue

## Required Facts

- Current write path: `relayer wallet -> resonanceContract.LeavePairQueueByCa(LeavePairQueueByCaInput)`
- Input field: `ca_hash`
- Queue state is keyed by resolved `ca_address`
- All resonance `Get*` and other view-only reads in this flow must use the direct view path
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
9. Read `GetPairQueueStatus(caller_ca_address)`.
10. Read `GetPairQueueStats()` when practical.
11. If the caller is already absent from the queue, explain whether the most likely reason is `already left`, `expired`, `matched`, or `not in queue`.
12. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary.
13. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions.portkey_ca`
    - keep the default layer focused on caller identity, current queue state, whether the write can proceed, and what leaving now will do
    - keep raw contract addresses, `caller_ca_hash`, `caller_ca_address`, queue-state reads, and queue stats in `Technical Details` unless the user explicitly asks for them
14. Ask for explicit confirmation.
15. Only after explicit confirmation, send `LeavePairQueueByCa({ caHash })`.
16. If a `txId` is returned, share the `txId` and explorer link.
17. Read `GetPairQueueStatus()` again when practical.
18. Return the post-send summary as `left queue` or the most accurate non-error reason if the address was already absent.
19. Append the success CTA for clear non-error leave results.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local caller `CA` context cannot be resolved
- no usable local relayer or signer can be resolved for the current write
- multiple local `CA` accounts are available but the caller was not chosen yet
- `GetConfig().portkey_ca_contract_address` is missing

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions.portkey_ca`, caller identity, whether the write can proceed, the current queue-state conclusion, and explicit confirmation request
- localized technical-details layer on demand with chosen flow, `caller_ca_hash`, `caller_ca_address`, target raw execution address, current config, queue status, and queue stats

## Example Reference

Read [../examples/ca-leave-pair-queue.md](../examples/ca-leave-pair-queue.md) before replying.
