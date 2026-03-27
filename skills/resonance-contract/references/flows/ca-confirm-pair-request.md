# CA Confirm Pair Request

Use this branch for the CA direct-confirm path after local CA context is ready.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.3.0`

## When To Use

Use this flow only when all conditions below are true:

- a local `CA` context is already available
- the user wants to confirm an existing pending pair
- the user provided or confirmed `initiator_ca_hash`

## Required Facts

- Current write path: `relayer wallet -> resonanceContract.ConfirmPairRequestByCa(ConfirmPairRequestByCaInput)`
- Input fields: `initiator_ca_hash`, `counterparty_ca_hash`
- `counterparty_ca_hash` normally comes from the local caller `CA` context
- All resonance `Get*` and other view-only reads in this flow must use the direct view path
- `GetConfig().portkey_ca_contract_address` must be present before any write can proceed
- Confirm must match an active pending pair after both sides resolve to `ca_address`
- Confirm checks the raw reward pool before randomness is sampled

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local caller `CA` context, including `caller_ca_hash`, `caller_ca_address`, and a usable local relayer or signer for the current write.
5. If multiple local `CA` accounts are available and the caller identity is still ambiguous, stop this write path and ask which local `CA` account should be used.
6. Validate the user-supplied `initiator_ca_hash`.
7. If the initiator input looks like an on-chain `Address` instead of a `ca_hash`, stop the write path and correct the input expectation in plain language.
8. Read `GetConfig()` through the direct view path.
9. Stop if the contract appears uninitialized.
10. Stop if `GetConfig().portkey_ca_contract_address` is missing.
11. Resolve `initiator_ca_hash` into `initiator_ca_address` through the configured Portkey CA contract.
12. Stop if the resolved initiator equals the caller.
13. Read `GetPairStatus()` and `GetPendingPair()` for the unordered pair using resolved addresses.
14. Stop if the pair is already `EXECUTED`.
15. Stop if there is no active pending pair.
16. Stop if the active pending pair does not match the resolved initiator and caller addresses.
17. Read `GetRemainingBalance()` and `GetRewardBalance()` when practical.
18. Compute the confirm-side minimum reward check from the pending-pair snapshots when available, otherwise from config.
19. Stop if the remaining balance is lower than the confirm-side minimum reward check.
20. If any stop condition above is hit, return the blocked summary from the output contract instead of a pre-send confirmation summary.
21. Show the pre-send summary using the output contract:
    - render the localized user-summary layer first, with visible `skill_version` and `dependency_versions.portkey_ca`
    - keep the default layer focused on caller identity, initiator identity, whether the write can proceed, pending expiry, and the balance conclusion
    - keep raw contract addresses, `caller_ca_hash`, `caller_ca_address`, `initiator_ca_hash`, `initiator_ca_address`, pending-pair reads, and reward-balance reads in `Technical Details` unless the user explicitly asks for them
22. Ask for explicit confirmation.
23. Only after explicit confirmation, send `ConfirmPairRequestByCa({ initiatorCaHash, counterpartyCaHash })`.
24. If a `txId` is returned, share the `txId` and explorer link.
25. Read `GetPairStatus()` again when practical.
26. Return the post-send execution summary.
27. Append the success CTA when the pair result is clearly returned.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local caller `CA` context cannot be resolved
- no usable local relayer or signer can be resolved for the current write
- multiple local `CA` accounts are available but the caller was not chosen yet
- initiator input is not a valid `ca_hash`
- initiator input is an on-chain `Address` rather than a `ca_hash`
- `GetConfig().portkey_ca_contract_address` is missing
- initiator `ca_hash` cannot be resolved into `ca_address`
- resolved initiator equals the caller
- the pair is already executed
- the pair request is not pending
- the caller is not the expected counterparty
- the remaining reward balance is lower than the confirm-side minimum reward check

## Output Shape

The response before sending should contain:

- localized user-summary layer first with `skill_version`, `dependency_versions.portkey_ca`, caller identity, initiator identity, whether the write can proceed, pending expiry, the main blocker or balance conclusion, and explicit confirmation request
- localized technical-details layer on demand with chosen flow, `caller_ca_hash`, `caller_ca_address`, `initiator_ca_hash`, `initiator_ca_address`, target raw execution address, current config, active pending pair, `GetRemainingBalance`, `GetRewardBalance`, and the confirm-side minimum reward check

## Example Reference

Read [../examples/ca-confirm-pair-request.md](../examples/ca-confirm-pair-request.md) before replying.
