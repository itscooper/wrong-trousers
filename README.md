# The Wrong Trousers Problem - Companion Repo

Agentic AI can perform useful cybersecurity work without either requiring a human approval for every minor action or being given unrestricted autonomy. The practical problem is supervision: deciding what to delegate, equipping the agent to do it, controlling its authority, verifying its conclusions, intervening when necessary, and spending finite resources deliberately.

This repository is a practical companion for experimenting with that model. It contains reusable Skills, a generic cybersecurity system prompt, a sanitized OpenCode permissions block, and research references.

It accompanies a 44CON talk on supervising agentic cybersecurity work.

**[Read the research references](REFERENCES.md)**

## License

Unless otherwise stated, the original material in this repository is licensed under [CC BY 4.0](LICENSE.md). See the license file for attribution requirements and exclusions for third-party material.

## Six supervision domains

### 1. Intent & Task — What should it do?

Good supervision begins before the first tool call.

Define the work clearly, provide relevant human context, distinguish hypotheses from facts, and retain human ownership of consequential judgement.

Useful principle:

**Delegate investigation and execution without quietly delegating the decision you remain accountable for.**

### 2. Skills & Tools — What does it need?

Agent performance depends on its working environment, not only its prompt.

Consider:

- durable system/base instructions;
- reusable Skills;
- files and repository context;
- architecture and evidence;
- CLI tools;
- APIs;
- MCP integrations;
- web access where appropriate.

Prefer the smallest useful integration surface.

A general agent plus reusable Skills can often be more useful than many artificial agent personas.

### 3. Authority & Boundaries — What may it do?

Capability is not authority.

Allow enough routine, local, reversible and inspectable work for the agent to be useful.

Require stronger control at consequential boundaries such as:

- remote mutation;
- deployment;
- privilege changes;
- secret access;
- destructive operations.

Unknown behaviour should not silently inherit authority.

The objective is not zero approval prompts.

It is **high-value approval prompts**.

### 4. Verification & Challenge — Why should I believe it?

Do not substitute plausible explanations for evidence.

Ask:

- What did you inspect?
- What did you not inspect?
- What evidence supports this?
- What contradicts it?
- What would prove the conclusion wrong?
- Can the result be reproduced or tested?

Cybersecurity has an advantage here because many claims can be checked through code, configuration, logs, scanners, tests, runtime evidence and independent tools.

Useful principle:

**Trust evidence, not eloquence.**

### 5. Intervention & Recovery — How do I stay in control?

Supervision must let the human change the trajectory, not merely approve actions requested by the agent.

Useful capabilities include:

- observe;
- pause;
- steer;
- correct;
- veto;
- resume;
- roll back;
- take over;
- abandon.

A reusable correction should not remain trapped in a chat transcript.

Where appropriate, convert a useful lesson into:

- a Skill;
- a test;
- an instruction;
- a policy;
- another durable part of the working environment.

Useful principle:

**Correct this task. Improve the tasks that follow.**

The agent may propose changes to its durable instructions or Skills, but must not silently expand or rewrite its own operating authority.

### 6. Resource Management — What may it consume?

Supervision is also resource allocation.

Consider:

- tokens;
- model/API cost;
- elapsed time;
- tool calls;
- compute;
- external API usage;
- context;
- human attention.

Human expert attention can be more expensive than model tokens.

A cheap agent run requiring twenty trivial approvals may be worse than a somewhat more expensive run that performs the routine work independently and asks one meaningful question.

Useful principle:

**Spend human attention where human judgement has the highest value.**

## Closing principle

Use AI to increase the amount of useful cyber work humans can supervise without reducing the knowledge and judgement needed to supervise it.

AI can do more of the legwork while humans continue to frame, challenge, interpret and decide.

## Artifact boundaries

The artifacts have different jobs:

```text
System prompt = durable operating posture
Skill         = reusable task-specific procedure
Tools         = concrete capabilities
Permissions   = authority boundary over their effects
```

A Skill does not grant authority. Review durable Skill changes. A strong prompt does not establish correctness, a permission rule does not establish truth, a read-only tool does not make data private, and a visible agent does not automatically provide meaningful control. More approval prompts are not automatically better supervision: permissions should focus on meaningful effects, while verification should rely on inspectable evidence. Agents should report evidence, uncertainty, failed checks, and uninspected areas; humans retain consequential risk, prioritisation, authorization, and acceptance decisions. Local execution is not automatically private; understand data flows through models, telemetry and external tools.

## Repository map

- [`skills/`](skills/) contains reusable Skills.
- [`prompts/`](prompts/) contains a cybersecurity system prompt.
- [`config/`](config/) contains the full example OpenCode permissions block.
- [`REFERENCES.md`](REFERENCES.md) contains research references supporting the repository's guidance.

These examples were derived from practical experiments with local coding/agent harnesses and then sanitized for public reuse.
