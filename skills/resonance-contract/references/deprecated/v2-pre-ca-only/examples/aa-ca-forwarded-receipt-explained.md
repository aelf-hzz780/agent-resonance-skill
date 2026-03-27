# Example: AA/CA Forwarded Receipt Explained

## User Input

- English: `My AA/CA join receipt shows VirtualTransactionCreated and PairQueueJoined. What does that mean?`
- 中文: `我的 AA/CA 入队回执里有 VirtualTransactionCreated 和 PairQueueJoined，这分别是什么意思？`

## Agent Should Choose

- `Status Query And Diagnostics`

## Correct Output Shape

- explain that `VirtualTransactionCreated` is expected on successful `CA.ManagerForwardCall` writes
- explain that it only proves the CA contract created the forwarded inner call
- explain that `PairQueueJoined` is the business event that shows the holder actually entered the queue
- say that final queue or pair state should still be confirmed with direct read-after-write queries such as `GetPairQueueStatus`
- keep low-level receipt and event details in `Technical Details` unless the user asks to expand
