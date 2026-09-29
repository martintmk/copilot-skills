# Guidance for changing skills

This repository contains reusable Copilot skills. Keep every skill easy for both
agents and people to follow.

## Writing style

- Start with the skill's purpose and when to use it.
- Use short sentences and familiar words.
- Prefer direct instructions: "Read the PR" rather than "The PR should be read".
- Explain why an important rule exists, but avoid unnecessary implementation detail.
- Keep one idea per paragraph or list item.
- Use headings, numbered steps, and small tables to make procedures scannable.
- Define unfamiliar terms the first time they appear.
- Avoid vague words such as "soon", "appropriately", or "as needed".
- Do not repeat rules that already belong to another skill. Link to the owner.

## Changing a skill

1. Read the skill's `SKILL.md` and any linked reference files before editing.
2. Keep the frontmatter name, trigger description, scope, and exclusions accurate.
3. Make the procedure specific enough that an agent can follow it without
   guessing.
4. State important limits, required evidence, failure handling, and write
   permissions plainly.
5. Update related README tables, examples, or reference files when the change
   affects them.
6. Check links and formatting, then review the final text as a human reader.

## Structure for `SKILL.md`

Use this order unless the skill has a good reason to differ:

1. Purpose and boundaries in the frontmatter.
2. A short introduction explaining the job of the skill.
3. The procedure, in the order an agent should perform it.
4. Decision rules and edge cases near the step where they matter.
5. Links to detailed reference material.

Keep detailed contracts, long examples, and specialist guidance in nearby
reference files. `SKILL.md` should remain the clear entry point.

## Before finishing

- Confirm the instructions still match the tools and files they mention.
- Confirm related skills do not now contradict this one.
- Run the smallest relevant validation available.
- Do not add planning notes or unrelated cleanup to the repository.
