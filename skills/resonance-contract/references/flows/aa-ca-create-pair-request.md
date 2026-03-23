# AA/CA Create Pair Request

Use this branch for the `AA/CA` create path after account choice is complete.

## Required Dependency

Use the Portkey CA skill explicitly:

- `https://github.com/Portkey-Wallet/ca-agent-skills`
- validated runtime version: `2.2.0`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `AA`, `CA`, or `AA/CA`
- a local `AA/CA` context is already available
- the user wants to initiate `CreatePairRequest`

## Required Facts

- Forwarded write path: `manager signer -> CA.ManagerForwardCall -> resonanceContract.CreatePairRequest(Address counterparty)`
- Caller at the resonance contract layer is the resolved `AA/CA` holder address, not the manager signer
- Counterparty input must be a valid on-chain `Address`
- The pair cannot be self-paired
- An executed unordered pair cannot be created again
- An active pending pair cannot be created again
- An expired old pending pair can be auto-cleared and recreated in one transaction

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, `resonance_contract_address`, and `portkey_ca_contract_address` are available.
2. Normalize `resonance_contract_address` into display full-address form and raw execution form.
3. Detect the local Portkey CA skill version and mark either normal mode or compatibility mode.
4. Use the Portkey CA skill to resolve the local `AA/CA` holder address, `caHash`, and a usable manager signer.
5. If recovery or manager switching happened in the same session, query holder info on the target execution chain and stop until the chosen manager is visible there.
6. Validate the counterparty input as an on-chain `Address`.
7. Stop if the counterparty address equals the resolved `AA/CA` holder address.
8. Read `GetConfig()`.
9. Stop if the contract appears uninitialized.
10. Record the current window and reward tiers from `GetConfig()`.
11. Read `GetPairStatus()` for the unordered pair using `AA/CA holder address` and `counterparty`.
12. Read `GetPendingPair()` for the unordered pair.
13. Stop if `GetPairStatus().status == EXECUTED`.
14. Stop if `GetPairStatus().status == PENDING` or `GetPendingPair()` returns an active pending pair.
15. If a stale pending pair exists but is expired, explain that `CreatePairRequest` can auto-clear it in the same transaction.
16. Show the pre-send summary using the output contract, including normalized contract addresses, dependency mode, and the forwarded method chain.
17. Ask for explicit confirmation.
18. Only after explicit confirmation, use the Portkey CA skill to send the forwarded `CreatePairRequest(counterparty)` call.
19. If a `txId` is returned, share the `txId` and explorer link.
20. Read `GetPairStatus()` again using the `AA/CA` holder address and `counterparty`.
21. Read `GetPendingPair()` again.
22. Return the read-after-write summary with the new pending pair fields.
23. Append the community CTA because the pending pair was successfully created.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- local `AA/CA` holder address cannot be resolved
- no usable manager signer can be resolved
- the target execution chain holder info does not yet include the chosen manager signer
- the counterparty input is not a valid on-chain `Address`
- the counterparty equals the resolved `AA/CA` holder address
- the pair has already resonated
- the pair request is already pending

## Output Shape

The response before sending should contain:

- chosen flow: `AA/CA Create Pair Request`
- manager signer
- resolved `AA/CA` holder address
- `caHash` when available
- counterparty
- target resonance contract in normalized full-address and raw-address form
- target CA contract
- dependency mode and detected Portkey CA skill version when practical
- method chain `ManagerForwardCall -> CreatePairRequest(Address counterparty)`
- current window and reward tiers
- current pair state
- expired-pending auto-clear note when relevant
- explicit confirmation request

## Example Reference

Read [../examples/aa-ca-create-pair-request.md](../examples/aa-ca-create-pair-request.md) before replying.
