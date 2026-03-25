# Example: Status Query And Diagnostics

## User Input

- English: `Check the pair or queue status on ResonanceContract for these inputs: <address-a>, <address-b>.`
- 中文: `帮我查一下 ResonanceContract 上这几个输入对应的配对或排队状态：<address-a>、<address-b>`

## Agent Should Choose

- `Status Query And Diagnostics`

## Must Ask Or Confirm

- no write confirmation if the user only asked for a read-only lookup

## Must Not Ask

- do not route into admin methods
- do not ask for `email` or `caHash`

## Correct Output Shape

- use a natural operation label such as `查询当前状态` or `Check current status`, rather than exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, the state conclusion, the most important time or balance anchor, the practical reason for that state, and the next practical step
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- if a prior agent used `CA.ManagerForwardCall` or an `EOA` send path for a resonance `Get*` method, say that the call path is wrong before interpreting pair or queue state
- keep `GetConfig`, pair reads, address-scoped queue or pending reads, and queue stats in the localized technical-details layer unless the user asks to expand
- explain whether the observed state is pending, executed, queued, missing, likely expired, blocked by warmup, or blocked by balance semantics
- if the user pasted a forwarded receipt with `VirtualTransactionCreated`, explain that it is a write receipt rather than a direct view response
- if executed, keep fallback evidence and the exact fallback route in the localized technical-details layer unless the user explicitly asks to expand
- if certificate status is relevant, explain that it can remain `COMING_SOON` while still exposing strong-resonance payload
- if it is a hard-stop diagnosis, do not append community CTA
