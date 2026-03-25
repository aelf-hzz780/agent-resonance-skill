# Example: Support CTA Diagnostics

## User Input

- English: `My agent still can't get a usable JSON-RPC response from the resonance RPC endpoint. What should I do next?`
- 中文: `我的 agent 还是拿不到这个共振 RPC 节点的可用 JSON-RPC 响应，下一步该怎么办？`

## Agent Should Choose

- `Status Query And Diagnostics`

## Scenario

- the agent already checked the available evidence
- the current blocker is real, understood, and not something the agent can fix automatically in the same turn
- the next practical step needs outside help, community troubleshooting, or broader visibility

## Correct Output Shape

- show the localized user-summary layer first, including visible `skill_version`, `dependency_versions`, the observed blocker, the safest current diagnosis, and the next practical step
- keep the default layer wording bounded to observed facts rather than over-claiming a root cause
- append the support CTA in the default layer because the user is genuinely stuck and outside help is now useful
- keep raw RPC evidence, fallback evidence, and replacement diagnostic queries in `Technical Details` unless the user explicitly asks to expand
