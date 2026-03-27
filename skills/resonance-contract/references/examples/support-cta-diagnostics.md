# Example: Support CTA Diagnostics

## User Input

- English: `Why can’t my current CA account join the queue right now?`
- 中文: `为什么我现在这个 CA 账号还不能加入队列？`

## Example Diagnosis

- `GetConfig().portkey_ca_contract_address` is missing, so the contract cannot resolve `ca_hash` into `ca_address`

## Correct Output Shape

- show the localized user-summary layer first
- keep `skill_version` and `dependency_versions.portkey_ca` visible
- explain the blocker in plain language
- state clearly that the agent cannot continue this write automatically
- append the support CTA in the default visible layer
- render support CTA in `Telegram -> X` order
- keep exact config fields and raw read results in the localized technical-details layer unless the user asks to expand
