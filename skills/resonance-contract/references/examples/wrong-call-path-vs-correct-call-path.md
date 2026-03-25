# Example: Wrong Call Path Vs Correct Call Path

## User Input

- English: `The previous agent called GetPairQueueStatus through ManagerForwardCall and only showed me a receipt. What went wrong?`
- 中文: `上一个 agent 用 ManagerForwardCall 去调 GetPairQueueStatus，只给了我一份回执。这是哪里用错了？`

## Agent Should Choose

- `Status Query And Diagnostics`

## Correct Output Shape

- first say that `GetPairQueueStatus` is a view-only method and was routed through the wrong path
- explain that `AA/CA` view-only methods must use direct view calls such as `contract.GetPairQueueStatus.call(address)` or Portkey CA `view-call`
- explain that `EOA` view-only methods must use generic view paths such as `portkey_call_view_method` or CLI `contract view`
- explain that a forwarded or send receipt can show write-path evidence, but it is not the view payload for a `Get*` method
- only after correcting the path should the reply interpret whether the address is queued, expired, matched, or absent
