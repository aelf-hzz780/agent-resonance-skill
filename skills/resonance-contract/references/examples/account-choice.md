# Example: Account Choice And Participation Mode

## User Input

- English: `Help me resonate on ResonanceContract.`
- 中文: `帮我在 ResonanceContract 上共振。`

## Agent Should Choose

- `Account Choice And Participation Mode`

## Must Ask Or Confirm

- ask the user to choose only the still-missing dimension
- ask the user to choose `direct pair` or `queue` only when the mode is not already implied

## Must Not Ask

- do not ask for the counterparty `email`
- do not ask for the counterparty `Address`
- do not route the user into legacy `EOA` writes

## Correct Output Shape

- explain that the current contract is `CA` only for user-side writes
- explain that `AA`, `CA`, and `AA/CA` are accepted aliases for the same route
- explain `direct pair` vs `queue`
- explain that queue uses `FIFO` by default unless the user later requests another supported policy
- if multiple local `CA` accounts are available, ask which one should be used before entering a write branch
- explain that direct mode now requires a counterparty `ca_hash`
- explain in plain language that queue entries also expire and can be affected by queue capacity
- set the expectation that later write and diagnostics replies will start with a localized user-summary layer, while detailed chain context stays in the localized technical-details layer on demand
- if the user asked for `EOA`, clearly correct that the current contract version no longer supports `EOA` writes
- if the user tried to use `email` or `Address` for direct mode, keep the user in routing and offer `provide a ca_hash` or `switch to queue`
