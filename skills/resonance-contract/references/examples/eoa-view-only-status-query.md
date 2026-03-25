# Example: EOA View-Only Status Query

## User Input

- English: `Check my queue status on ResonanceContract with EOA. Do not send anything.`
- 中文: `我用 EOA 查一下 ResonanceContract 上的排队状态，不要发交易。`

## Agent Should Choose

- `Status Query And Diagnostics`

## Correct Output Shape

- say that the request is read-only and must use the generic EOA view path
- use `portkey_call_view_method`, CLI `contract view`, or an SDK read call for `GetPairQueueStatus`
- do not suggest `portkey_call_send_method` for `GetConfig`, `GetPairQueueStatus`, or any other resonance `Get*` method
- if the user pasted an EOA send receipt for a `Get*` method, explain that it is the wrong path before giving queue-state conclusions
