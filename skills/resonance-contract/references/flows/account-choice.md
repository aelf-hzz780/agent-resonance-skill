# Account Choice And Participation Mode

Use this branch for generic resonance requests before entering a concrete write path.

## When To Use

Use this flow when any of the following is true:

- the user has not chosen between `AA/CA` and `EOA`
- the user has not chosen between `direct pair` and `queue`
- the local caller account context is not ready yet
- the user asks for general help such as `help me resonate` or `帮我共振`

## Required User-Facing Explanation

The agent must first explain:

- `AA/CA`: account-style experience with smoother recovery and account management semantics
- `AA`: the preferred user-facing term in this skill
- `CA`: accepted as the same route alias as `AA/CA`
- `EOA`: traditional self-custodied wallet experience
- `direct pair`: use this when the user already knows the other side's on-chain `Address`
- `queue`: use this when the user does not know a counterparty and wants automatic matching
- default queue policy: if the user does not later request a different policy, queue matching uses `FIFO`
- recommendation: choose `AA/CA` when the user has no strong preference
- direct-mode counterparty input must be an on-chain `Address`, not `email`, not `caHash`
- queue is not permanent; the exact timeout comes from the contract's current `request_expire_seconds` and should later be restated in plain language before any queue write
- if the queue is already full and the new caller still does not match immediately, the earliest still-valid queued user may be removed before the new caller joins

Then ask:

- ask only the still-missing dimension:
  - `Which account type do you want to use: AA/CA or EOA?`
  - if the user did not already provide a direct counterparty address or explicitly ask for queue: `Do you want direct pair or queue?`

## Step-By-Step

1. Explain `AA/CA` vs `EOA` using the required explanation above.
2. Tell the user that `AA` is the preferred term and `CA` is accepted as the same route alias.
3. Explain `direct pair` vs `queue`, including the default `FIFO` queue behavior, the queue timeout concept, and queue-full behavior in plain language.
4. Tell the user that only direct mode requires a counterparty on-chain `Address`.
5. Recommend `AA/CA` when the user has no strong preference.
6. If the user has not chosen an account type yet, ask the user to choose `AA/CA` or `EOA`.
7. If the user already provided a valid direct counterparty address, treat the participation mode as `direct pair`.
8. If the user did not provide a direct counterparty address and did not explicitly ask for queue, ask whether they want `direct pair` or `queue`.
9. If the user insists on using `email` or `caHash` as a direct-mode counterparty input, do not redirect to diagnostics; stay in routing, explain that direct mode only accepts an on-chain `Address`, and offer two next steps: provide an `Address` or switch to `queue`.
10. If the user chooses `AA/CA` and the local `AA/CA` context is already available, switch to the matching `AA/CA` direct or queue branch.
11. If the user chooses `AA/CA` but the local context is not ready, use the Portkey CA skill dependency to prepare local context first, then continue.
12. If the user chooses `EOA` and the local `EOA` account is already available, switch to the matching `EOA` direct or queue branch.
13. If the user chooses `EOA` but the local context is not ready, use the Portkey EOA skill dependency to prepare local context first, then continue.

## Must-Stop Conditions

Stop the current direct-path routing and restate the available user-side choices if:

- the user asks for admin methods
- the user wants the agent to custody or create keys on their behalf
- the user insists on using `email` or `caHash` as a direct-mode counterparty input

## Output Shape

The reply should contain:

- `AA/CA vs EOA` explanation
- `direct pair vs queue` explanation
- explicit note that queue uses `FIFO` by default unless the user later requests another supported policy
- explicit note that `CA` is accepted as the `AA/CA` alias
- explicit reminder that only direct mode requires an on-chain `Address`
- explicit expectation-setting that later write and diagnostics replies will default to a user summary first, while technical details stay on demand
- explicit question asking the user to choose `AA/CA` or `EOA` only when account type is not already implied
- explicit question asking the user to choose `direct pair` or `queue` only when the mode is not already implied
- when the user tried `email` or `caHash` for direct mode, a correction that keeps them in onboarding and offers `provide an Address` or `switch to queue`

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
