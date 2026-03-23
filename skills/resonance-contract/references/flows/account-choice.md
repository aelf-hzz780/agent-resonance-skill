# Account Choice And Onboarding

Use this branch for generic resonance requests before entering a concrete write path.

## When To Use

Use this flow when any of the following is true:

- the user has not chosen between `AA/CA` and `EOA`
- the local caller account context is not ready yet
- the user asks for general help such as `help me resonate` or `帮我共振`

## Required User-Facing Explanation

The agent must first explain:

- `AA/CA`: account-style experience with smoother recovery and account management semantics
- `AA`: the preferred user-facing term in this skill
- `CA`: accepted as the same route alias as `AA/CA`
- `EOA`: traditional self-custodied wallet experience
- recommendation: choose `AA/CA` when the user has no strong preference
- the counterparty input must be an on-chain `Address`, not `email`, not `caHash`

Then ask:

- `Which account type do you want to use: AA/CA or EOA?`

## Step-By-Step

1. Explain `AA/CA` vs `EOA` using the required explanation above.
2. Tell the user that `AA` is the preferred term and `CA` is accepted as the same route alias.
3. Tell the user that the counterparty value must be an on-chain `Address`.
4. Recommend `AA/CA` when the user has no strong preference.
5. Ask the user to choose `AA/CA` or `EOA`.
6. If the user chooses `AA/CA` and the local `AA/CA` context is already available, switch to the matching `AA/CA` create or confirm branch.
7. If the user chooses `AA/CA` but the local context is not ready, use the Portkey CA skill dependency to prepare local context first, then continue.
8. If the user chooses `EOA` and the local `EOA` account is already available, switch to the matching `EOA` create or confirm branch.
9. If the user chooses `EOA` but the local context is not ready, use the Portkey EOA skill dependency to prepare local context first, then continue.

## Must-Stop Conditions

Stop and redirect to diagnostics if:

- the user asks for admin methods
- the user wants the agent to custody or create keys on their behalf
- the user insists on using `email` or `caHash` as the counterparty input

## Output Shape

The reply should contain:

- chosen flow: `Account Choice And Onboarding`
- `AA/CA vs EOA` explanation
- explicit note that `CA` is accepted as the `AA/CA` alias
- explicit reminder that the counterparty must be an on-chain `Address`
- explicit question asking the user to choose `AA/CA` or `EOA`

## Example Reference

Read [../examples/account-choice.md](../examples/account-choice.md) before replying.
