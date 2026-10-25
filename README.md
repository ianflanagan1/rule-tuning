# rule-tuning

Claude Code skills that let the rules Claude loads learn from code reviews. `/ianf-review-against-guides` reviews changed code against your guides and confirms each problem by running something. `/ianf-tune-rules` then records what it found in your guides and proposes changes to them, which rebuild your rules.

## Why

A rules file loads into every session that touches its language, so every line costs tokens and dilutes the others. It should carry only what Claude gets wrong without being told. Knowing which lines those are takes evidence, and a review is where the evidence turns up: the same model that wrote a bug usually spots it when asked to look.

## Guides and rules

The skills expect a guides repository, kept separately and linked into `~/.claude`:

| File | What it holds | Loaded |
|---|---|---|
| `~/.claude/guides/languages/<lang>.md` | Everything known about the language: each pitfall or preference, why it matters, its rule line, its status and the evidence for it | Only by these skills |
| `~/.claude/guides/all-languages.md` | The same for code in any language | Only by these skills |
| `~/.claude/guides/FORMAT.md` | The entry format, and when an entry's status changes | Only by these skills |
| `~/.claude/rules/languages/<lang>.md` | Built from the guide: the rule lines of its `rule` entries | When Claude reads a file matching its `paths` |
| `~/.claude/rules/all-languages.md` | Built from the guide, the same way | In every session |

The rule in the format that matters most: a review can show that a rule is needed, but never that it isn't. A line is dropped only after an ablation, which runs Claude on the same tasks with and without the line.

## What it does

1. **`/ianf-review-against-guides [paths or git range]`** reads the guide for each rules file that applies to the code in scope. It hands the review to a fresh subagent, so the review doesn't share the reasoning the code was written with. Each suspected problem is confirmed by running code, and the skill never changes the project. It prints the findings, then the guide entries the code handled correctly, then hands them to ianf-tune-rules in a second subagent, which sees only the findings, not your conversation. This is the private sub-skill pattern. If it has proposals, the review shows them and ends with a 👉 line asking which to apply; your answer goes back to the same subagent.
2. **ianf-tune-rules** sorts each confirmed finding:
   - violated despite a rule: reword the rule or enforce it with a check in the language's baseline package
   - under an omitted entry: promote it once the evidence is enough
   - not in the guide: add a new entry
   - caused by a rule: narrow the rule
   - specific to the project: nothing goes in the guide; it proposes a line for the project's `CLAUDE.md` or an ADR

   It appends evidence lines to guide entries without asking. Every other change is proposed with its exact text, and only the ones you approve are applied, to the guides, followed by a rebuild of the rules.

`/ianf-tune-rules` also works on its own, on review findings you give it or that are in the conversation. Only findings confirmed by running something count as evidence, so most of what it gets this way is listed as unconfirmed.

## Install

Symlink both skills:

```sh
ln -s ~/projects/claude/rule-tuning/skills/ianf-review-against-guides ~/.claude/skills/ianf-review-against-guides
ln -s ~/projects/claude/rule-tuning/skills/ianf-tune-rules ~/.claude/skills/ianf-tune-rules
```

The `ianf-` prefix keeps them from colliding with other skills of the same name.
