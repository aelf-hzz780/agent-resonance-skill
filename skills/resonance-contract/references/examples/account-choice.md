# Example: Account Choice And Participation Mode

## User Input

- English: `Help me resonate on ResonanceContract.`
- 中文: `帮我在 ResonanceContract 上共振。`

## Agent Should Choose

- `Account Choice And Participation Mode`

## Must Ask Or Confirm

- ask the user to choose only the still-missing dimension
- ask the user to choose `AA/CA` or `EOA` only when account type is not already implied
- ask the user to choose `direct pair` or `queue` only when the mode is not already implied by the request

## Must Not Ask

- do not ask for the counterparty `email`
- do not ask for the counterparty `caHash`
- do not start with an admin method

## Correct Output Shape

- explain `AA/CA` vs `EOA`
- explain `direct pair` vs `queue`
- explain that queue uses `FIFO` by default unless the user later requests another supported policy
- note that `CA` is accepted as the `AA/CA` alias
- recommend `AA/CA` when the user has no strong preference
- remind the user that only direct mode requires a counterparty on-chain `Address`
- explain in plain language that queue entries also expire and can be affected by queue capacity
- set the expectation that later write and diagnostics replies will start with a localized user-summary layer, while detailed chain context stays in the localized technical-details layer on demand
- if the user tried to use `email` or `caHash` for direct mode, keep the user in routing and offer `provide an Address` or `switch to queue`
- ask the user to choose `AA/CA` or `EOA` only when account type is not already implied
- ask the user to choose `direct pair` or `queue` only when the mode is not already implied
