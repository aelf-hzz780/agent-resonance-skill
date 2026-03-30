# Example: Wrong Call Path Vs Correct Call Path

## User Input

- English: `Another agent used a send receipt to query GetPairQueueStatus. Is that okay?`
- 中文: `另一个 agent 拿一笔发送回执去查 GetPairQueueStatus，这样对吗？`

Alternative user input in the same diagnostics family:

- English: `I only have an old receipt from a route that is not CreatePairRequestByCa / ConfirmPairRequestByCa / JoinPairQueueByCa / LeavePairQueueByCa. Can I still use it as the current state source?`
- 中文: `我手上只有一笔旧路径留下的回执，不是现在这些 *ByCa 方法。还能拿它当当前状态依据吗？`

## Correct Interpretation

- current `ResonanceContract v2.0.0` reads must use direct view calls
- a send receipt is not the correct source for a view-only query result
- if the pasted receipt comes from an older or unsupported route, explain that it does not belong to the current CA-only runbook before diagnosing current state
- if the user only wants queue state:
  - resolve `ca_hash` into `ca_address` when needed
  - then call `GetPairQueueStatus(ca_address)` through the direct view path

## Correct Output Shape

- first explain in the localized user-summary layer that the prior agent used the wrong path for a view-only method
- if the receipt belongs to an older or unsupported route, first explain that it is not part of the current CA-only flow and should not be treated as the current state source
- if the pasted receipt only proves a send happened, explain that it is not the direct view payload
- explain the correct next step in plain language: resolve identity if needed, then use the direct view method
- do not append CTA if the agent can continue immediately with the corrected view path
- keep exact replacement query details and raw contract method names in the localized technical-details layer unless the user asks to expand
