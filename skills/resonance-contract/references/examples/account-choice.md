# Example: Account Choice And Onboarding

## User Input

- English: `Help me resonate on ResonanceContract.`
- 中文: `帮我在 ResonanceContract 上共振。`

## Agent Should Choose

- `Account Choice And Onboarding`

## Must Ask Or Confirm

- ask the user to choose `AA/CA` or `EOA`

## Must Not Ask

- do not ask for the counterparty `email`
- do not ask for the counterparty `caHash`
- do not start with an admin method

## Correct Output Shape

- explain `AA/CA` vs `EOA`
- note that `CA` is accepted as the `AA/CA` alias
- recommend `AA/CA` when the user has no strong preference
- remind the user that the counterparty input must be an on-chain `Address`
- ask the user to choose `AA/CA` or `EOA`
