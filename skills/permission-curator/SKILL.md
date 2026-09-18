---
name: permission-curator
description: >
  Review, curate, and maintain OpenCode tool permissions so agents can work efficiently
  without normalising high-impact authority. Use this skill when new shell commands or MCP
  tools are added, when the user asks to review or change permissions, or after a session
  produced too many or too few approval prompts.
metadata:
  category: agent-supervision
  policy-style: risk-tiered-adaptive
  target: opencode
---

# Permission Curator

Maintain OpenCode permissions as a **supervision boundary**, not as a blanket safety lock.

The goal is to let an agent complete substantial software engineering and cybersecurity work
without approval fatigue, while concentrating human attention on actions whose **effects are
consequential, difficult to reverse, privileged, secret-bearing, externally mutating, or
outside the expected task boundary**.

## Core philosophy

Use a **default-ask, known-good allow** model, but make the known-good set broad enough for
real agentic work.

Optimise for both:

1. **Useful autonomy** — routine local, reversible, inspectable work should normally proceed
   without interruption.
2. **Meaningful control** — human approval should be concentrated at consequential boundaries,
   where judgement adds value.

Do not equate "executes code" with "requires approval".

In a trusted first-party worktree, builds, tests, linters, package scripts, local dependency
installation, local Git operations, containerised development, passive security scanning and
similar activity are often the work itself. Prompting for each of these can reduce security by
training the user to approve reflexively.

Prefer **effect-oriented reasoning** over command-name anxiety.

## When to use this skill

Invoke this skill when any of the following is true:

- a new CLI, shell command, package manager, scanner, cloud tool or developer tool is introduced;
- a new MCP server or MCP tool appears;
- the user explicitly asks to review, loosen, tighten, explain or modify permissions;
- a session generated excessive approval prompts;
- a session generated too few approval prompts or allowed something the user expected to review;
- an approval was surprising, low-value, repetitive, or routinely accepted without inspection;
- a tool's behaviour, naming or authority changed;
- the repository or task has moved into a materially different trust context;
- the current permissions have become difficult to reason about due to overlapping patterns.

## Files and evidence to inspect

Before changing policy:

1. Locate the active OpenCode configuration.
    - Commonly: `$HOME/.config/opencode/opencode.jsonc`
   - Do not assume this path if the active configuration indicates otherwise.

2. Inspect the current permission block in full.

3. Enumerate currently available tools and MCP servers when possible.

4. If this is a post-session review, inspect available session/tool-call history or logs.
   If those are unavailable, use the user's description of which prompts were excessive or missing.

5. Preserve unrelated configuration exactly.

6. Determine whether the current configuration is using a legacy or newer OpenCode permission
   schema before editing. Do not silently translate schemas unless required.

## Trust context first

Before classifying commands, determine the working context.

### Trusted first-party worktree

Default posture: **high useful autonomy**.

Routine project execution may be allowed, including:

- builds;
- tests;
- linters and formatters;
- package scripts;
- local dependency installation;
- local Git changes;
- branch/worktree operations;
- local containers and compose environments;
- passive security scanning;
- IaC validation and planning;
- read-only cloud/Kubernetes inspection.

### Untrusted or adversarial worktree

Examples:

- unknown third-party repositories;
- malware samples;
- exploit proof-of-concepts;
- supply-chain investigations;
- deliberately hostile fixtures;
- repositories suspected of containing malicious build/install hooks.

Default posture: **more restrictive**.

Do not reuse trusted-worktree allowances blindly. Project scripts, package hooks, build systems,
containers and generated tooling can execute arbitrary code.

If the trust context has changed materially, recommend or create a separate restricted profile
rather than degrading the normal trusted-worktree profile for every task.

## Decision model

Classify the **effect** of the action, not merely the tool name.

### Normally ALLOW

Allow when the action is typically local, reversible, inspectable, and expected as part of
ordinary engineering or security work.

Common examples:

- environment and version inspection;
- file/source searching and local inspection;
- formatting, linting, compiling and testing;
- project package scripts;
- local dependency installation;
- creating and editing project files;
- local Git add/commit/branch/rebase/merge/stash operations;
- Git fetch/pull/clone from expected sources;
- local container build/run/compose workflows without host escape characteristics;
- Terraform/OpenTofu fmt/validate/init/plan/show;
- Kubernetes get/describe/logs/diff/explain;
- passive or local security analysis such as SAST, SCA, IaC scanning, secret detection,
  SBOM generation and vulnerability database lookup;
- DNS/TLS metadata inspection;
- read-only GitHub/Atlassian/project-management queries;
- dedicated web/research search MCPs;
- browser observation, screenshots, snapshots, performance inspection and ordinary navigation.

### Normally ASK

Ask when the action crosses a consequential boundary.

Common reasons:

- destructive or difficult-to-reverse local operations;
- privilege escalation or ownership/permission changes;
- secret or credential retrieval;
- external mutations;
- publishing or releasing artifacts;
- pushing changes to a remote;
- production or cloud mutation;
- deployment;
- creating, merging, closing or modifying remote collaboration objects where the effect matters;
- destructive database operations;
- generic database execution tools whose SQL semantics are not visible to the permission layer;
- active security testing against network targets;
- arbitrary outbound network requests where the destination/content matters;
- browser form submission, upload, arbitrary JavaScript execution, or authenticated state mutation;
- host-level container escape capabilities such as privileged mode, host PID/network, or broad
  host filesystem mounts;
- package-manager account, registry, token or publishing operations;
- system package installation;
- service/process control;
- SSH/SCP/remote shell;
- writing outside the expected worktree.

### Normally DENY

Use deny sparingly.

Prefer `ask` when a human can reasonably make a contextual decision.

Use `deny` for:

- explicit user or organisational hard boundaries;
- operations that should never occur in the current profile;
- known-dangerous behaviour where prompting would add no meaningful decision value;
- commands or tools the user has explicitly prohibited;
- bypass mechanisms intended to disable or evade the permission system itself.

Do not invent hard-deny policy merely because a tool is powerful.

## Shell permission curation

### Principle

Shell permissions should support **end-to-end completion** of engineering and cybersecurity tasks.

A shell policy that only permits inspection is not a useful agent policy.

### Prefer families of known-good patterns

Where semantics are stable, allow coherent command families rather than individual exact commands.

Examples of useful allow families in a trusted worktree include:

- `npm *`, `pnpm *`, `yarn *`, `bun *`
- `pytest *`, `python -m pytest *`
- `make *`, `just *`, `task *`
- `go test*`, `go build*`, `go vet*`, `go mod *`
- `cargo *`
- `mvn *`, `./mvnw *`
- `gradle *`, `./gradlew *`
- `dotnet *`
- `composer *`
- `eslint *`, `prettier *`, `tsc *`, `jest *`, `vitest *`
- local/passive scanners such as `semgrep *`, `trivy *`, `grype *`, `syft *`,
  `osv-scanner *`, `gitleaks *`, `bandit *`, `gosec *`, `checkov *`,
  `shellcheck *`, `pip-audit *`

Then add **more-specific ASK overrides** for consequential subcommands such as publishing,
registry/account mutation or deployment.

Example pattern:

```jsonc
"npm *": "allow",
"npm publish*": "ask",
"npm unpublish*": "ask",
"npm token*": "ask",
"npm profile*": "ask",
"npm login*": "ask"
```

Do not shrink the broad allow merely because a small subset is risky when a specific override
can preserve both autonomy and control.

### Git philosophy

Local Git should normally be autonomous.

Typical ALLOW:

- status/diff/log/show/blame/grep;
- branch/tag inspection and normal local creation;
- add/commit;
- switch/checkout;
- merge/rebase/cherry-pick;
- stash;
- fetch/pull/clone.

Typical ASK:

- push;
- destructive clean/reset/restore patterns;
- forceful branch deletion;
- remote URL changes;
- global/system Git configuration.

A local commit is usually a reversible work product. A push crosses an external mutation boundary.

### Containers

Allow ordinary local development containers.

Typical ALLOW:

- inspect/list/log/stats;
- build;
- normal run;
- compose development workflows.

Typical ASK:

- `--privileged`;
- host PID/network;
- broad host-root mounts;
- pushing images;
- registry login;
- destructive prune/volume deletion.

### Infrastructure tooling

Distinguish **planning/inspection** from **mutation**.

Typical ALLOW:

- Terraform/OpenTofu validate, fmt, init, plan, show;
- Kubernetes get, describe, logs, diff, explain;
- Helm list/status/history/get/show/template/lint.

Typical ASK:

- apply/destroy/import/state mutation;
- kubectl apply/delete/patch/exec and other cluster mutations;
- Helm install/upgrade/uninstall/rollback;
- cloud resource create/update/delete.

### Security tooling

Distinguish **local/passive analysis** from **active target interaction**.

Typical ALLOW:

- static analysis;
- dependency analysis;
- secret scanning;
- IaC analysis;
- local image/filesystem scanning;
- SBOM generation;
- vulnerability database lookup;
- DNS, TLS certificate and metadata inspection.

Typical ASK unless scope has been explicitly pre-approved:

- port scanning;
- content discovery;
- fuzzing;
- exploitation;
- credential attacks;
- intrusive web scanning;
- network-wide enumeration.

If the user has explicitly scoped a legitimate active-testing engagement, consider adding
narrower known-good patterns for that task instead of repeatedly prompting for every command.

## MCP permission curation

### Server-first default

For MCP servers that can both read and mutate:

1. set the server wildcard to `ask`;
2. allow specific low-risk read/query/inspect tools;
3. leave mutating or ambiguous tools on `ask`.

Example:

```jsonc
"github_*": "ask",
"github_get*": "allow",
"github_list*": "allow",
"github_search*": "allow",
"github_issue_read": "allow",
"github_pull_request_read": "allow"
```

### Dedicated read-only/research servers

If a server is genuinely read-oriented by design, it may be reasonable to allow the whole server.

Examples include dedicated web/search services.

Do not apply this treatment merely because most current tools happen to be read-only.

### Ambiguous tool names

Keep generic effect-bearing tools on `ask` when the permission layer cannot see enough semantics.

Examples:

- `execute`
- `execute_sql`
- `query`
- `run`
- `request`
- `invoke`
- `call`
- `command`

A generic SQL/query tool may support both `SELECT` and destructive operations.

### Secrets

Retrieving a secret is itself consequential.

Do not classify password-manager or secret-store reads as harmless just because the API verb is
`get`, `read` or `list`.

Default secret-bearing MCPs to `ask` unless the user has explicitly created a narrower policy.

### Browser automation

Generally ALLOW:

- list pages;
- inspect console/network;
- screenshots/snapshots;
- performance/audit tools;
- ordinary navigation and page selection.

Generally ASK:

- arbitrary JavaScript execution;
- file uploads;
- form submission;
- credential entry;
- purchases;
- account/security changes;
- actions that create external side effects.

## Handling a newly added tool

When a new command, CLI, MCP server or MCP tool appears:

1. Identify what it can actually do.
2. Classify its effects using the model above.
3. Check whether an existing pattern already covers it.
4. Check whether that existing pattern is too broad or too narrow.
5. Prefer the smallest rule that removes low-value prompts without silently granting unrelated authority.
6. Preserve a broad `ask` fallback.
7. For mixed-capability families, use broad allow + specific ask overrides when safe.
8. For ambiguous generic execution tools, keep `ask`.
9. Update comments so the intent remains understandable.
10. Report the change and its rationale concisely.

Do not add a tool to `allow` merely because it was used successfully once.

Do not leave a tool on `ask` merely because it is new.

## Approval-fatigue review

When the user reports **too many approvals**, or session evidence shows repeated approvals:

### Step 1 — Find noisy approval classes

Group prompts by:

- command/tool;
- effect;
- resource;
- destination;
- repeated argument shape;
- whether the user actually inspected the request before approving.

### Step 2 — Identify low-value prompts

A prompt is a strong candidate for auto-allow when:

- it occurs frequently;
- it is usually approved;
- it is local/reversible;
- its effect is easy to inspect after execution;
- it is required for normal task progress;
- the user's approval does not materially change based on arguments.

Examples:

- repeated test runs;
- repeated linters;
- repeated `git diff/status`;
- local build commands;
- read-only MCP queries;
- passive scanner invocations.

### Step 3 — Widen carefully

Prefer:

- a stable command family;
- a stable read-only MCP family;
- a resource-scoped rule;
- broad allow plus specific dangerous-subcommand overrides.

Avoid creating dozens of brittle exact-command rules if one semantic family is appropriate.

### Step 4 — Protect the consequential edge

Before broadening a rule, explicitly test whether it would also match:

- publish;
- deploy;
- push;
- delete;
- destructive reset;
- privilege changes;
- secret retrieval;
- production mutation;
- active third-party interaction.

Add later/more-specific `ask` rules where required.

## Under-approval review

When the user reports **too few approvals**, or an action occurred that should have been reviewed:

1. Identify the exact effect that crossed the expected boundary.
2. Determine which existing allow rule matched it.
3. Avoid overcorrecting by removing a whole useful command family if a narrow override is possible.
4. Add a more-specific `ask` or, only where justified, `deny`.
5. Search for sibling operations with the same effect.
6. Explain what remains allowed and what newly requires review.

Example:

Bad correction:

```jsonc
"npm *": "ask"
```

Preferred correction:

```jsonc
"npm *": "allow",
"npm publish*": "ask",
"npm unpublish*": "ask",
"npm token*": "ask"
```

## Rule-order and shadowing review

After every edit, check for policy shadowing.

For every new rule:

- identify broader matching rules;
- identify later matching rules;
- confirm the intended winner;
- ensure a broad allow does not swallow a consequential subcommand;
- ensure a broad ask does not accidentally nullify intended known-good allowances.

When the host uses "last matching rule wins", put:

1. broad fallback first;
2. broad known-good allow next;
3. specific consequential ask/deny overrides after it.

If the host uses different semantics, adapt accordingly.

## Avoid brittle overfitting

Do not create permission policy that mirrors one transcript line-by-line.

Optimise for **classes of work**.

Prefer:

- `pytest *`

over:

- `pytest tests/unit/test_auth.py -q`

Prefer:

- `github_search*`

over:

- one exact GitHub search invocation.

But do not generalise across unrelated servers or capabilities solely because they share verbs such
as `get`, `read`, `search` or `query`.

## Human-attention principle

Every approval consumes human attention.

Treat approval prompts as a scarce supervisory resource.

A permission should remain `ask` when the decision genuinely benefits from contextual human
judgement.

A permission should move toward `allow` when the human is predictably rubber-stamping a routine,
low-impact operation.

The objective is not the lowest possible prompt count.

The objective is the **highest-value prompt set**.

## Modification procedure

When asked to modify the active policy:

1. Read the active config.
2. Make the smallest coherent change.
3. Preserve formatting/comments where practical.
4. Keep the default fallback explicit.
5. Keep consequential boundaries explicit even if they already fall through to default `ask`;
   these comments/rules document design intent.
6. Validate JSON/JSONC syntax where possible.
7. Re-read the final permission block for shadowing.
8. Do not modify unrelated settings.
9. Present a concise summary:
   - what was added/changed;
   - what now runs without approval;
   - what still requires approval;
   - any notable residual risk or trust assumption.

If the user asked only for analysis/recommendations, do not edit the config until they request the change.

## Suggested review cadence

Review permissions:

- after adding a material new tool/server;
- after a session with obvious approval fatigue;
- after a surprising autonomous side effect;
- when task/repository trust changes;
- periodically after enough real usage has accumulated to reveal recurring prompt patterns.

Do not churn policy after every isolated prompt.

## Success criteria

A well-curated permission policy should let an agent:

- investigate;
- edit;
- build;
- test;
- lint;
- scan;
- iterate;
- use local Git;
- inspect infrastructure;
- gather evidence;
- use read-oriented external tools;

without constant interruption.

It should still make the human deliberately notice actions such as:

- pushing/publishing;
- production/cloud mutation;
- destructive operations;
- privilege changes;
- secret access;
- external consequential writes;
- high-impact infrastructure changes;
- active security interaction outside pre-approved scope.

That is the intended balance:

> **Delegate the work. Reserve human attention for consequential boundaries.**
