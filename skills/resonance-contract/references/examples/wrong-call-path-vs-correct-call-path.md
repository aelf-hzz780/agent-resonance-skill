# Example: Wrong Call Path Vs Correct Call Path

## User Input

- English: `Another agent used ManagerForwardCall to query GetPairQueueStatus. Is that okay?`
- 中文: `另一个 agent 用 ManagerForwardCall 去查 GetPairQueueStatus，这样对吗？`

## Correct Interpretation

- current `ResonanceContract v2.0.0` reads must use direct view calls
- legacy `ManagerForwardCall` belongs to the pre-CA-only write model and is not the correct read path now
- if the user only wants queue state:
  - resolve `ca_hash` into `ca_address` when needed
  - then call `GetPairQueueStatus(ca_address)` through the direct view path

## Correct Output Shape

- first explain in the localized user-summary layer that the prior agent used the wrong path for a view-only method
- if the pasted receipt contains `VirtualTransactionCreated`, explain that it is not the direct view payload
- explain the correct next step in plain language: resolve identity if needed, then use the direct view method
- do not append CTA if the agent can continue immediately with the corrected view path
- keep exact replacement query details and raw contract method names in the localized technical-details layer unless the user asks to expand
