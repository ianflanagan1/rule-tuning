---
name: ianf-review-against-guides
description: Review changed code for bugs and language pitfalls against the guides in ~/.claude/guides and the rules Claude loads, confirming each suspected problem by running code, then hand the findings to ianf-tune-rules. Never changes the code.
argument-hint: [paths or git range]
disable-model-invocation: true
model: inherit
---

# Review against guides

Find what's wrong with the code, prove each problem by running something, and pass what was found to ianf-tune-rules, in a subagent, so the rules learn from it.

## Scope (`/ianf-review-against-guides $ARGUMENTS`)

- `$ARGUMENTS` names paths or a git range. Empty: the uncommitted changes (staged, unstaged and untracked); with none, the commits on the current branch since it left the default branch.
- Nothing in scope: reply `➖ Nothing to review.` and stop.
- Not in a git repository and no paths given: reply `⚠️ Not a git repository.` and say to pass paths.

## Sources

The guides mirror the rules: for each rules file in `~/.claude/rules/languages/` that applies to the code in scope, the guide of the same name in `~/.claude/guides/languages/`. Add `~/.claude/guides/all-languages.md` always, whose rules file `~/.claude/rules/all-languages.md` is already loaded. Note a language with no guide; review it from what you know.

## Review

Launch one general-purpose subagent for the review, so it doesn't share the reasoning the code was written with. Give it the scope (paths or the git range, not the code), the source files above, the project's root, and this brief:

> Review the code in scope for bugs, security holes and language pitfalls. Read the guide and rules files first: each entry is something to check. Don't stop at them; the most useful findings are ones no file mentions. Skip what the project's linters and type checker already enforce.
>
> Confirm each suspicion by running code: a snippet with the project's own runtime and dependencies, an existing test, or a new test written in your scratchpad directory. Never change the project's files. Report a suspicion you can't confirm as unconfirmed, with the reason.
>
> Return the findings, most severe first, each as:
> - where: path:line
> - problem: one sentence
> - confirmed: the command and the output that shows it, or `unconfirmed: <reason>`
> - fix: one sentence
> - entry: the heading of the guide entry it falls under, or `none`
>
> Then list each guide entry whose situation came up in the code and was handled correctly, as `applied: <heading> at path:line`. Only entries the code genuinely exercised.

## Report

Print the findings as the subagent returned them, confirmed ones first, then the `applied` list. Don't change the code; the user decides what to fix.

## Tune the rules

If no finding is confirmed and the `applied` list is empty, end with `➖ Nothing for ianf-tune-rules to record.`

Otherwise run ianf-tune-rules as a private sub-skill (design 2 in `~/projects/claude/skill-patterns/private-sub-skill.md`, where it exists), so it starts with only the findings, not this conversation. Launch one general-purpose subagent with the project's root, this brief, and then the findings and the `applied` list exactly as printed above:

> Read `~/.claude/skills/ianf-tune-rules/SKILL.md` and follow it, as a sub-skill. The findings below are its `$ARGUMENTS`.

It records evidence without asking, then returns either a status line or its proposals.

- **A status line** (`✅ Evidence recorded.` or `➖ Nothing to record.`): print what it returned, and stop.
- **Proposals:** print them as returned, then end with `👉 Reply with the proposals to apply: numbers, all or none.` When the user answers, send the answer to the same subagent with SendMessage, so it applies them with its context intact, and print what it returns.
