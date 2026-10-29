---
name: ianf-tune-rules
description: Turn review findings into evidence in the guides in ~/.claude/guides, and into proposed changes to those guides, the rules files Claude loads and lint config, applying only what the user approves. Run after /ianf-review-against-guides in the same session, or on other review findings in the conversation.
argument-hint: [findings, if not in the conversation]
disable-model-invocation: true
model: inherit
---

# Tune rules

Record what a review found against the guides, and propose how the rules Claude loads should change because of it.

## Gather (`/ianf-tune-rules $ARGUMENTS`)

- The findings are `$ARGUMENTS`, or else the latest review findings in the conversation: from ianf-review-against-guides or the user. None: reply `➖ No findings.` and stop.
- Read `~/.claude/guides/FORMAT.md`. It defines the entries, their status and when status changes.
- Read the guide and the rules file for each language the findings touch, and `~/.claude/guides/all-languages.md`.
- Only confirmed findings count as evidence. List unconfirmed ones at the end, unrecorded.

## Classify

For each confirmed finding:

1. **Under a `rule` or `lint` entry, violated anyway.** The wording or the enforcement failed: propose rewording the rule line, or a checker that enforces it.
2. **Under an `omitted` entry.** Propose promoting it if the evidence now meets the threshold in the guide format.
3. **No entry.** Propose a new entry. State the claim as a behaviour of the language, library or domain, not as the incident. Its status is `omitted` unless this one finding meets the threshold.
4. **Caused by a rule.** Record `overreach` and propose narrowing the rule line.
5. **Specific to this project** (its domain, its own conventions) or a plain slip with no lesson. No guide entry. For a project convention, propose a line for the project's `CLAUDE.md` or an ADR, following the project's own process.

Never propose dropping a rule line because nothing has violated it. If one looks unnecessary, say an ablation would settle it.

## Record

Without asking, append an `evidence` line to each existing entry a confirmed finding or an `applied` item falls under. These lines record what happened; they change no status and no rules file.

## Propose

First list the unconfirmed findings, one line each, so the user can confirm any by hand. Then print the proposals as a numbered list. For each, give the kind of finding, the files it changes and the exact text: the new or changed guide entry, and the check to add to the language's baseline package. For an entry becoming `rule`, give its rule line's length in words; the built rules file loads into every session for the language, so prefer `lint` where a checker can enforce it. Then ask which to apply: numbers, `all` or `none`.

Run as a sub-skill (ianf-review-against-guides runs it in a subagent): don't ask. End your answer with the numbered proposals and stop; the caller asks the user and sends the answer back to you.

## Apply

Apply the approved proposals to the guides; never edit a rules file. Keep each guide's sections and voice. A new entry's prose says what goes wrong, gives a minimal example and the fix.

Then rebuild: in the guides' repository (`cd -P ~/.claude/guides/..`), run `devbox run build`, then `devbox run check`. A check added to a baseline package goes in that package's repository, with its own check task run afterwards. Report each result, and leave every repository uncommitted for the user.

## Output

- `✅ Rules updated.` when proposals were applied: the files changed and, for each proposal, applied or declined.
- `✅ Evidence recorded.` when evidence was appended but no proposal was applied: the entries that got evidence.
- `➖ Nothing to record.` when no confirmed finding or `applied` item falls under an entry and nothing qualifies for a proposal.
