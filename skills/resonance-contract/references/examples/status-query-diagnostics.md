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

- identify the branch as `Status Query And Diagnostics`
- summarize `GetConfig`, including `new_participation_available_time` and `queue_capacity`
- when the input is pair-oriented, summarize `GetPairStatus` and `GetPendingPair`
- when the input is address-oriented, summarize `GetActivePendingPair`, `GetPairQueueStatus`, and `GetPairQueueStats` when relevant
- explain whether the observed state is pending, executed, queued, missing, likely expired, blocked by warmup, or blocked by balance semantics
- if executed, include outcome details and fallbacks used
- if certificate status is relevant, explain that it can remain `COMING_SOON` while still exposing strong-resonance payload
- if it is a hard-stop diagnosis, do not append community CTA
