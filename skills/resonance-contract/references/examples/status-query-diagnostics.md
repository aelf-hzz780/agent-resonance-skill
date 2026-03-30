# Example: Status Query And Diagnostics

## User Input

- English: `Check the pair or queue status on ResonanceContract for this ca_hash or these ca_address values: <input>.`
- 中文: `帮我查一下这个 ca_hash 或这组 ca_address 在 ResonanceContract 上的配对或排队状态：<input>`

## Agent Should Choose

- `Status Query And Diagnostics`

## Must Ask Or Confirm

- no write confirmation if the user only asked for a read-only lookup

## Must Not Ask

- do not route into admin methods
- do not ask for `email`
- do not assume the current contract still supports any non-CA write route

## Correct Output Shape

- use a natural operation label such as `查询当前状态` or `Check current status`, rather than exposing the internal branch name
- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions.portkey_ca`, the state conclusion, the most important time or balance anchor, the practical reason for that state, and the next practical step
- if the user supplied `ca_hash`, explain that the skill first resolves it into `ca_address`
- if a prior agent used a generic send path or another non-view transport for a resonance `Get*` method, say that the call path is wrong before interpreting pair or queue state
- keep `GetConfig`, pair reads, identity-normalization details, address-scoped queue or pending reads, and queue stats in the localized technical-details layer unless the user asks to expand
- explain whether the observed state is pending, executed, queued, missing, likely expired, blocked by warmup, blocked by missing Portkey CA config, or blocked by balance semantics
- if it is a clear non-error status such as pending, executed, or still queued, append the success CTA in `X -> Telegram` order
- if it is a real blocker or externally stalled diagnosis such as warmup, missing Portkey CA config, or RPC transport trouble, append the support CTA in `Telegram -> X` order
- if the issue is only invalid input or a light route correction the agent can immediately fix, do not append CTA
