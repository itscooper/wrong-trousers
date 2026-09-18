---
name: session-to-skill
description: Distill a successful agent session into a reusable skill, or improve an existing skill with newly learned execution knowledge. Use at the end of a repeatable workflow when future runs should benefit from conclusions, procedures, edge cases, tool usage, or failure lessons established during the current session.
---

# Session to Skill

Turn a successful agent session into a reusable skill, or improve an existing skill using what was learned during the session.

## Output location

Default to:

`$HOME/.agents/skills/<skill-name>/SKILL.md`

Use another location only when explicitly instructed.

A skill may also contain supporting resources where useful:

```text
<skill-name>/
├── SKILL.md
├── scripts/       # optional executable helpers
├── references/    # optional detailed knowledge
└── assets/        # optional templates/resources
```

## Objective

Capture **reusable execution knowledge**, not just the user's original prompt.

The resulting skill should let a future agent perform the same class of task:

- with less exploration and re-reasoning;
- using conclusions already established;
- avoiding approaches that failed or caused friction;
- following the most effective sequence of actions;
- preserving important constraints, edge cases, formats, commands, tool choices, and validation steps discovered during execution.

## Process

1. **Review the session**

   Identify:
   - the underlying repeatable task;
   - the successful workflow and order of operations;
   - decisions or conclusions that required investigation or reasoning;
   - assumptions that were validated;
   - failed approaches, traps, and unnecessary work;
   - useful commands, queries, file locations, schemas, formats, examples, or tool behaviours;
   - checks that established whether the result was correct.

2. **Check for an existing skill**

    Check the current session for an existing skill to improve as well as `$HOME/.agents/skills/` for a skill covering substantially the same activity.

   - If one exists, improve it rather than creating a duplicate.
   - Preserve useful existing knowledge unless the current session demonstrates it is wrong or obsolete.
   - Integrate new learning into the appropriate sections rather than appending a session log.

3. **Distil**

   Generalise session-specific details into reusable rules.

   Include concrete details when they are necessary to execute the task efficiently, but exclude:
   - conversation history;
   - one-off user data;
   - transient outputs;
   - reasoning narration;
   - facts that are obvious or cheap to rediscover.

   Prefer:

   > Use X before Y because Y assumes Z.

   Rather than:

   > In this session we tried Y, discovered Z, then switched to X.

4. **Create or update the skill**

   Every `SKILL.md` must begin immediately with YAML frontmatter:

   ```yaml
   ---
   name: skill-name
   description: What this skill does and when the agent should use it.
   ---
   ```

   Frontmatter requirements:

   - `name` and `description` are required.
   - `name` must exactly match the parent directory name.
   - `name` must be 1–64 characters using lowercase letters, numbers, and hyphens only.
   - Do not use leading, trailing, or consecutive hyphens.
   - `description` must be 1–1024 characters and should clearly describe both **what the skill does** and **when it should activate**.
   - Make the description specific enough that an agent can reliably decide whether to load the skill.

   Optional frontmatter fields may be added only when useful:

   ```yaml
   license: Apache-2.0
   compatibility: Requires git and network access
   metadata:
     author: example
     version: "1.0"
   ```

   After the frontmatter, write normal Markdown instructions. There is no required body schema.

5. **Structure for efficient reuse**

   Keep `SKILL.md` concise and operational. Include only sections that help execution, typically:

   - purpose / triggers;
   - key knowledge or invariants;
   - recommended workflow;
   - important edge cases or failure modes;
   - validation / definition of done;
   - concise examples where they remove ambiguity.

   Put detailed or conditional information in `references/`, reusable automation in `scripts/`, and templates/static material in `assets/` when this keeps the main skill focused.

6. **Optimise for the next run**

   Before saving, ask:

   > What did I have to work out during this session that a future agent should not have to work out again?

   Ensure those answers are encoded explicitly.

## Skill quality rules

- Do not simply restate the original prompt.
- Encode conclusions, not chain-of-thought.
- Prefer rules over session-specific narrative.
- Be specific where specificity saves future reasoning.
- Record failed approaches when knowing to avoid them is useful.
- Avoid generic agent behaviour the agent already knows.
- Make instructions executable and easy to scan.
- Preserve useful existing knowledge when updating.
- Replace knowledge shown by the session to be incorrect or obsolete.
- Do not create multiple skills where one coherent skill can cover the workflow.
- Keep `SKILL.md` focused; move large conditional material into supporting files.
- Use relative paths from the skill directory when referencing bundled resources.

## Naming

Choose a short, memorable, task-oriented kebab-case name, for example:

`session-to-skill`  
`triage-findings`  
`publish-release`  
`review-risk-acceptance`

Prefer an existing skill name when updating an existing skill.

## Completion

Create or modify the relevant skill under `$HOME/.agents/skills/` unless instructed otherwise.

Before finishing, verify:

1. `<skill-name>/SKILL.md` exists.
2. Frontmatter is the first content in the file.
3. `name` exactly matches `<skill-name>`.
4. `description` explains both capability and activation conditions.
5. The skill captures knowledge learned during execution rather than just the initiating request.
6. Existing useful knowledge has not accidentally been lost.
7. A future agent can perform the workflow without rediscovering the important conclusions from this session.

Report:
- skill name
- created or updated
- path
- a summary of the most important reusable learning points captured
