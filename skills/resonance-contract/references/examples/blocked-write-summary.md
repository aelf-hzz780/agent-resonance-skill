# Example: Blocked Write Summary

## User Input

- English: `Use my local CA account and create a pair request for this counterparty ca_hash: <ca-hash>.`
- 中文: `用我本地的 CA 账号，帮我给这个对手方 ca_hash 发起配对：<ca-hash>`

## Example Blocker

- `GetConfig().portkey_ca_contract_address` is missing

## Correct Output Shape

- show the localized user-summary layer first
- keep `skill_version` and `dependency_versions.portkey_ca` visible
- explain in plain language that the contract has not configured its Portkey CA contract yet, so it cannot resolve `ca_hash` into `ca_address`
- state clearly that the write cannot proceed now
- give the next practical step: wait for the contract to be configured or ask the operator/community to confirm deployment config
- do not ask for confirmation
- append the support CTA because this is a real blocker the agent cannot fix automatically
- render support CTA in `Telegram -> X` order
- keep raw config reads and exact contract fields in the localized technical-details layer unless the user asks to expand
