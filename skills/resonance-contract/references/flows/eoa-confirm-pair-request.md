# EOA Confirm Pair Request

Use this branch for the `EOA` confirm path after account choice is complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants to confirm an existing pending pair

## Required Facts

- Method: `ConfirmPairRequest(Address initiator)`
- Caller must be the pending counterparty
- Initiator input must be a valid on-chain `Address`
- `ConfirmPairRequest` requires an active pending pair
- `ConfirmPairRequest` requires the reward pool to cover the maximum reward tier before randomness is sampled
- The contract uses the pending pair snapshots for the effective maximum reward

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Use the Portkey EOA skill to resolve the active local `EOA` signer.
3. Validate the initiator input as an on-chain `Address`.
4. Stop if the initiator address equals the resolved signer address.
5. Read `GetConfig()`.
6. Stop if the contract appears uninitialized.
7. Record the current window and reward tiers from `GetConfig()`.
8. Read `GetPairStatus()` for the unordered pair.
9. Read `GetPendingPair()` for the unordered pair.
10. Stop if `GetPairStatus().status == EXECUTED`.
11. Stop if there is no active pending pair.
12. Stop if the pending pair initiator does not match the requested initiator.
13. Stop if the resolved local signer does not match the pending pair counterparty.
14. Read `GetRemainingBalance()`.
15. Compute the baseline minimum pool check as `2 * (config.success_amount + config.strong_bonus_amount)`.
16. If pending snapshots are available, also compute the effective pair-specific minimum pool check as `2 * (success_amount_snapshot + strong_bonus_amount_snapshot)`.
17. Stop if the remaining balance is lower than the effective pair-specific minimum pool check.
18. Show the pre-send summary using the output contract.
19. Ask for explicit confirmation.
20. Only after explicit confirmation, use the Portkey EOA skill to send `ConfirmPairRequest(initiator)`.
21. If a `txId` is returned, share the `txId` and explorer link.
22. Read `GetPairStatus()` again.
23. Read `GetAddressStats()` for both participant addresses when practical.
24. Read `GetStrongRecord()` for both participant addresses when practical.
25. Read `GetCertificateStatus()` for both participant addresses when practical.
26. Return the read-after-write summary with outcome, `reward_each`, and any strong-record updates.
27. Append the community CTA because the confirm path returned a clear result.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the initiator input is not a valid on-chain `Address`
- the initiator equals the caller
- the pair has already resonated
- there is no active pending pair
- the pending pair initiator does not match the requested initiator
- the resolved signer is not the pending counterparty
- the remaining balance is lower than the effective pair-specific minimum pool check

## Output Shape

The response before sending should contain:

- chosen flow: `EOA Confirm Pair Request`
- resolved signer
- initiator
- target contract
- method `ConfirmPairRequest(Address initiator)`
- current window and reward tiers
- active pending pair summary
- remaining balance and minimum pool checks
- explicit confirmation request

## Example Reference

Read [../examples/eoa-confirm-pair-request.md](../examples/eoa-confirm-pair-request.md) before replying.
