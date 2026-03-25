# Example: Blocked Write Summary

## User Input

- English: `Use EOA and create a pair request for this address: <address>.`
- 中文: `用 EOA 帮我给这个地址发起配对：<address>`

## Agent Should Choose

- the matching write branch for the user path

## Scenario

- preflight found a real blocker such as warmup, queue conflict, active pending conflict, or insufficient available reward balance
- the write cannot proceed now

## Correct Output Shape

- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, caller identity when known, a plain-language blocker, and the next practical step
- include the target normalized full `resonance_contract_address` in the default layer only when the user explicitly supplied a non-default deployment or the deployment choice itself is materially relevant
- do not ask for confirmation because the send cannot proceed yet
- do not phrase the reply as `ready to send` or `confirm to continue`
- keep exact reads such as `GetConfig`, queue status, pending state, raw balance, available balance, and any fallback evidence in the localized technical-details layer unless the user asks to expand
- if the blocker is warmup, explain that new participation is temporarily unavailable until the warmup time passes
- if the blocker is direct-versus-queue exclusivity, explain that one address cannot hold both an active queue entry and an active pending direct request at the same time
- if the blocker is reward balance, explain the missing condition in plain language before showing the technical fields
