# EOA Create Pair Request

Use this branch for the `EOA` create path after account choice is complete.

## Required Dependency

Use the Portkey EOA skill explicitly:

- `https://github.com/Portkey-Wallet/eoa-agent-skills`

## When To Use

Use this flow only when all conditions below are true:

- the user explicitly chose `EOA`
- a local `EOA` account is already available
- the user wants to initiate `CreatePairRequest`

## Required Facts

- Method: `CreatePairRequest(Address counterparty)`
- Caller becomes the pair initiator
- Counterparty input must be a valid on-chain `Address`
- The pair cannot be self-paired
- An executed unordered pair cannot be created again
- An active pending pair cannot be created again
- An expired old pending pair for the same unordered pair can be auto-cleared and recreated in one transaction

## Step-By-Step

1. Validate that `chain_id`, `rpc_url`, and `resonance_contract_address` are available.
2. Use the Portkey EOA skill to resolve the active local `EOA` signer.
3. Validate the counterparty input as an on-chain `Address`.
4. Stop if the counterparty address equals the resolved signer address.
5. Read `GetConfig()`.
6. Stop if the contract appears uninitialized, for example `token_symbol` is empty or `admin` is missing.
7. Record the current window and reward tiers from `GetConfig()`.
8. Read `GetPairStatus()` for the unordered pair.
9. Read `GetPendingPair()` for the unordered pair.
10. Stop if `GetPairStatus().status == EXECUTED`.
11. Stop if `GetPairStatus().status == PENDING` or `GetPendingPair()` returns an active pending pair.
12. If a stale pending pair exists but is expired, explain that `CreatePairRequest` can auto-clear it in the same transaction.
13. Show the pre-send summary using the output contract.
14. Ask for explicit confirmation.
15. Only after explicit confirmation, use the Portkey EOA skill to send `CreatePairRequest(counterparty)`.
16. If a `txId` is returned, share the `txId` and explorer link.
17. Read `GetPairStatus()` again.
18. Read `GetPendingPair()` again.
19. Return the read-after-write summary with the new pending pair fields.
20. Append the community CTA because the pending pair was successfully created.

## Must-Stop Conditions

Stop immediately if any of the following is true:

- required config is missing
- no local `EOA` signer can be resolved
- the counterparty input is not a valid on-chain `Address`
- the counterparty equals the caller
- the pair has already resonated
- the pair request is already pending

## Output Shape

The response before sending should contain:

- chosen flow: `EOA Create Pair Request`
- resolved signer
- counterparty
- target contract
- method `CreatePairRequest(Address counterparty)`
- current window and reward tiers
- current pair state
- expired-pending auto-clear note when relevant
- explicit confirmation request

## Example Reference

Read [../examples/eoa-create-pair-request.md](../examples/eoa-create-pair-request.md) before replying.
