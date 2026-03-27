# Account Choice And Participation Mode

Use this branch for generic resonance requests before entering a concrete write path.

## When To Use

Use this flow when any of the following is true:

- the user has not chosen between `direct pair` and `queue`
- the local caller `CA` context is not ready yet
- the user asks for general help such as `help me resonate` or `帮我共振`
- the user still thinks the current contract supports `EOA` writes

## Required User-Facing Explanation

The agent must first explain:

- the current `ResonanceContract v2.0.0` user-side write model is `CA` only
- `AA`, `CA`, and `AA/CA` are accepted input aliases, but this skill uses `CA` as the canonical term
- legacy `EOA` write paths are no longer supported by the current contract version
- `direct pair`: use this when the user already knows the other side's `ca_hash`
- `queue`: use this when the user does not know a counterparty and wants automatic matching
- default queue policy: if the user does not later request a different policy, queue matching uses `FIFO`
- direct-mode counterparty input must be `counterparty_ca_hash`, not `email`, not an on-chain `Address`
- queue is not permanent; the exact timeout comes from the contract's current `request_expire_seconds` and should later be restated in plain language before any queue write
- if the queue is already full and the new caller still does not match immediately, the earliest still-valid queued user may be removed before the new caller joins

Then ask only the still-missing dimension:

- if local `CA` context is not already implied or ready: explain that a local `CA` account must be ready first, meaning the agent can resolve the caller's local `ca_hash` and `ca_address`
- if multiple local `CA` accounts are available and the caller identity is still ambiguous: ask which local `CA` account should be used for this round
- if the user did not already provide a counterparty `ca_hash` or explicitly ask for queue: ask `Do you want direct pair or queue?`

## Step-By-Step

1. Explain that the current contract is CA-only for user-side writes.
2. Tell the user that `AA`, `CA`, and `AA/CA` all map to the same CA route in this skill.
3. If the user asked for `EOA`, explain that `EOA` belongs to the legacy pre-`v2.0.0` contract path and cannot be used for current writes.
4. Explain `direct pair` vs `queue`, including default `FIFO`, queue timeout, and queue-full behavior in plain language.
5. Tell the user that direct mode now needs `counterparty_ca_hash`.
6. If the local `CA` context is not ready yet, use the Portkey CA skill dependency to prepare local context first, then continue.
7. If multiple local `CA` accounts are available and the caller identity is still ambiguous, ask the user which local `CA` account should be used before entering a write branch.
8. If the user already provided a counterparty `ca_hash`, treat the participation mode as `direct pair`.
9. If the user did not provide a counterparty `ca_hash` and did not explicitly ask for queue, ask whether they want `direct pair` or `queue`.
10. If the user insists on using `email` or on-chain `Address` for direct mode, stay in routing, explain that direct mode now only accepts `counterparty_ca_hash`, and offer two next steps: provide a `ca_hash` or switch to `queue`.
11. If the user still insists on `EOA`, stop the write-routing path and restate that the current contract version only supports CA-side participation.

## Must-Stop Conditions

Stop the current routing step and restate the available user-side choices if:

- the user asks for admin methods
- the user wants the agent to custody or create keys on their behalf
- the user insists on using `email` or `Address` as a direct-mode counterparty input
- the user insists on `EOA` writes on the current contract version

## Output Shape

The reply should contain:

- CA-only explanation
- `direct pair vs queue` explanation
- explicit note that queue uses `FIFO` by default unless the user later requests another supported policy
- explicit note that `AA`, `CA`, and `AA/CA` all point to the same current route
- when multiple local `CA` accounts are available, an explicit question asking which local `CA` account should be used
- explicit reminder that only direct mode requires a counterparty `ca_hash`
- explicit expectation-setting that later write and diagnostics replies will default to a user summary first, while technical details stay on demand
- explicit question asking the user to choose `direct pair` or `queue` only when the mode is not already implied
- when the user tried `email` or `Address` for direct mode, a correction that keeps them in onboarding and offers `provide a ca_hash` or `switch to queue`
- when the user asked for `EOA`, a clear correction that the current contract is CA-only and the old `EOA` path is legacy

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
