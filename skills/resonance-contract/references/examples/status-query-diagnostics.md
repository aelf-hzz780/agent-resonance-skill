# Example: Status Query And Diagnostics

## User Input

- English: `Check the pair status between these two addresses on ResonanceContract: <address-a> and <address-b>.`
- 中文: `帮我查一下 ResonanceContract 上这两个地址的配对状态：<address-a> 和 <address-b>`

## Agent Should Choose

- `Status Query And Diagnostics`

## Must Ask Or Confirm

- no write confirmation if the user only asked for a read-only status lookup

## Must Not Ask

- do not route into admin methods
- do not ask for `email` or `caHash`

## Correct Output Shape

- identify the branch as `Status Query And Diagnostics`
- summarize `GetConfig`, `GetPairStatus`, and `GetPendingPair`
- explain whether the pair is pending, executed, missing, or likely expired
- if executed, include outcome details
- if it is a hard-stop diagnosis, do not append community CTA
