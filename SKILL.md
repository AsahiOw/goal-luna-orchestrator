---
name: goal-luna-orchestrator
description: "Goal-mode coding orchestrator optimized for limited Codex quota. The current main-thread model acts as the lead architect and coordinator, while GPT-5.6 Luna Max performs the majority of repository exploration, implementation, debugging, testing, and iterative fixes. Use for substantial coding goals, complex features, refactors, debugging, redesigns, optimization, or other hard multi-step tasks. Preserve the user's selected main model rather than switching it."
---

# Goal Luna Orchestrator

Use the current main-thread model as the lead.

The lead may be any model selected by the user, for example:

- GPT-5.6 Sol Low
- GPT-5.6 Sol Medium
- GPT-6 Astra Low

Do not attempt to detect, replace, upgrade, or downgrade the main-thread model.

The selected main-thread model determines the quality and cost of orchestration.

GPT-5.6 Luna Max is the primary execution worker.

The purpose of this skill is to optimize useful work per Codex quota by keeping expensive lead-model work focused on planning, architecture, coordination, and acceptance while delegating token-heavy implementation work to Luna Max.

---

## 1. Role separation

### Main thread: Lead

The current main-thread model owns:

- understanding the user's goal;
- resolving high-level ambiguity;
- identifying important constraints;
- architecture and system-level decisions;
- breaking a large goal into coherent work packages;
- determining dependency order;
- defining acceptance criteria;
- deciding what work should happen next;
- evaluating worker results;
- integrating the overall result;
- deciding when the user's goal is actually complete.

The lead should NOT normally perform:

- broad repository exploration;
- routine code implementation;
- repetitive edits;
- large code generation;
- ordinary debugging iterations;
- test fixing loops;
- lint fixing;
- repository-wide searches;
- mechanical refactors.

Those belong to Luna Max.

### Worker: GPT-5.6 Luna Max

Use:

- model = "gpt-5.6-luna"
- reasoning_effort = "max"

Luna Max owns:

- repository exploration;
- locating relevant files and symbols;
- implementation;
- code generation;
- local implementation decisions;
- refactoring;
- debugging;
- running tests;
- running type checks;
- running lint;
- reproducing failures;
- correcting failures caused by its changes;
- validating its implementation;
- reporting concise evidence to the lead.

Luna Max is the default implementation model for substantial work.

Do not reduce Luna's reasoning effort merely because a work package appears straightforward.

This skill is intended for difficult Goal-mode tasks, so Max is the normal worker configuration.

---

## 2. Verify worker capability

Before the first real delegation, inspect the currently available subagent tool schema.

Confirm that a subagent can be launched with:

- model = "gpt-5.6-luna"
- reasoning_effort = "max"

If the schema supports context isolation:

- V2: use `fork_turns = "none"`
- V1: use `fork_context = false`

Do not send unsupported fields.

If the environment supports `task_name`, use a short descriptive name ending in:

`_luna_max`

Example:

`implement_statistics_dashboard_luna_max`

If GPT-5.6 Luna Max cannot actually be launched, tell the user that the required worker is unavailable and continue with the main thread only if doing so is sensible.

Do not silently replace Luna Max with another expensive model.

---

## 3. Delegate early

The lead must avoid duplicating work that Luna Max will perform.

Before delegation, the lead should inspect only enough context to:

1. understand the requested outcome;
2. identify major constraints;
3. determine whether the task should be split;
4. give Luna a useful starting point.

The lead SHOULD NOT fully map the repository before delegating.

Do not perform this pattern:

Lead deeply explores repository
→ Lead designs exact implementation
→ Luna explores same repository
→ Luna implements
→ Lead rereads same repository

Prefer:

Lead understands goal
→ Lead defines work package
→ Luna performs deep exploration + implementation + verification
→ Lead reviews evidence/diff as needed

Repository discovery belongs primarily to Luna.

---

## 4. Use coarse-grained work packages

Prefer giving Luna a complete coherent outcome instead of many tiny steps.

BAD:

1. ask Luna to find files;
2. wait;
3. ask Luna to modify one component;
4. wait;
5. ask Luna to connect API;
6. wait;
7. ask Luna to write tests;
8. wait;
9. ask Luna to fix tests.

GOOD:

Give Luna one work package that includes:

- exploration;
- implementation;
- relevant tests;
- failure correction;
- final verification.

A good work package should normally represent one coherent feature, subsystem change, debugging objective, optimization objective, or refactor.

Avoid micro-management.

---

## 5. Worker contract

Every Luna Max assignment should clearly state:

### Objective

Describe the concrete outcome to produce.

### Constraints

List important compatibility, architectural, behavioral, design, or scope constraints.

### Ownership

Tell Luna that it owns repository exploration and implementation for the assigned work package.

### Completion criteria

Define observable conditions that mean the package is complete.

### Verification

Require appropriate checks such as:

- tests;
- typecheck;
- lint;
- build;
- reproduction steps;
- targeted runtime validation.

### Scope protection

Tell Luna not to modify unrelated behavior.

### Return format

Require a compact handoff containing:

1. summary of what changed;
2. files changed;
3. important implementation decisions;
4. verification performed and results;
5. unresolved issues;
6. risks or follow-up work, if any.

Do not ask Luna for long explanations unless the lead genuinely needs them.

---

## 6. Let Luna complete its own implementation loop

Once delegated, Luna should normally be allowed to:

explore
→ implement
→ run verification
→ diagnose failures
→ fix failures
→ rerun verification

without returning to the lead after every minor step.

The lead should intervene only when:

- Luna encounters a major architectural ambiguity;
- requirements conflict;
- the requested behavior is unclear;
- the worker proposes a high-risk change outside its package;
- Luna cannot make meaningful progress;
- a dependency requires a lead-level decision.

This reduces unnecessary handoffs and repeated context processing.

---

## 7. Prefer worker continuity

When additional work belongs to the same subsystem and depends on context already learned by an existing Luna worker, continue using the same worker when the environment supports it.

Prefer:

Luna worker implements package
→ lead reviews
→ same Luna worker performs targeted correction

over creating a fresh worker that must relearn the subsystem.

Create a fresh Luna worker when:

- the new workstream is largely independent;
- context from the previous worker is irrelevant;
- parallelism is genuinely useful;
- isolation reduces risk.

Do not spawn additional workers merely to increase parallelism.

---

## 8. Default concurrency

Use one Luna Max worker at a time by default.

Parallel workers are allowed only when work packages are genuinely independent and cannot conflict.

Examples of acceptable parallelism:

- frontend redesign and independent CI configuration;
- independent test investigation and unrelated documentation migration;
- two isolated modules with no overlapping files or shared state.

Avoid parallel implementation when workers could modify overlapping files or architectural assumptions.

Quota efficiency is more important than maximizing agent count.

---

## 9. Lead review should be proportional to risk

Do not automatically reread every file Luna inspected.

### Low-risk change

Examples:

- styling;
- copy;
- isolated UI adjustment;
- straightforward component work;
- test additions;
- obvious bug fixes.

If Luna provides successful verification, the lead may accept using:

- worker summary;
- changed-file list;
- relevant diff;
- test evidence.

Do not duplicate the full investigation.

### Medium-risk change

Examples:

- new feature logic;
- state management;
- API integration;
- multi-file feature;
- moderate refactor.

The lead should inspect important diffs and verification results.

### High-risk change

Examples:

- authentication;
- authorization;
- security;
- database migration;
- destructive behavior;
- concurrency;
- major architecture;
- public API compatibility;
- data integrity.

The lead should perform deeper review of important code paths and evidence.

Review depth must be based on risk, not merely task size.

---

## 10. Direct lead implementation exception

The lead may directly implement a change when the remaining work is extremely small, obvious, and bounded.

Examples:

- a tiny localized correction;
- one obvious typo;
- a trivial configuration adjustment;
- a very small follow-up fix where delegation would cost more than doing it directly.

If completing the work requires meaningful repository exploration, multiple files, debugging, or verification loops, delegate to Luna Max.

The lead must not gradually become the primary coder through repeated "small exceptions."

---

## 11. Goal decomposition

For a large Goal-mode request, the lead should create a lightweight internal task map.

For each work package determine:

- desired outcome;
- dependencies;
- risk level;
- acceptance criteria;
- whether it can be delegated immediately.

Do not create an excessively detailed implementation plan before Luna investigates the repository.

Planning should describe outcomes and boundaries rather than predict every code change.

Example:

Goal:
Improve the statistics system.

Possible work packages:

1. identify current performance bottlenecks and implement highest-impact optimization;
2. redesign statistics visualization while preserving existing data behavior;
3. improve responsive layout and interaction;
4. perform integrated verification.

Delegate coherent packages sequentially unless independence makes parallelism clearly beneficial.

---

## 12. Handling worker uncertainty

If Luna reports uncertainty, first determine what type it is.

### Implementation uncertainty

If the architecture and objective are clear but implementation is difficult:

- let Luna Max continue reasoning;
- provide any missing constraint;
- ask for a targeted correction.

### Architectural uncertainty

If the correct direction itself is unclear:

- the lead resolves the architectural decision;
- then sends the decision back to Luna for implementation.

If the current main thread is Astra, use its stronger reasoning directly.

If the current main thread is Sol, do not automatically spawn Astra unless the user explicitly configured or requested such escalation.

The selected main-thread model remains authoritative.

---

## 13. Do not switch the user's lead model

This orchestrator must respect the model selected when the user started the task.

Examples:

Sol Low selected
→ Sol Low remains lead
→ Luna Max performs implementation

Sol Medium selected
→ Sol Medium remains lead
→ Luna Max performs implementation

Astra Low selected
→ Astra Low remains lead
→ Luna Max performs implementation

Do not automatically change Sol Low to Sol Medium.

Do not automatically replace Sol with Astra.

Do not automatically change Astra reasoning effort.

The user chooses lead capability before invoking this skill.

---

## 14. Context efficiency

Optimize handoffs for compactness.

The lead should send Luna:

- goal;
- relevant user requirements;
- important constraints;
- known entry points if useful;
- acceptance criteria.

Do not copy the entire conversation unless necessary.

Luna should return concise structured evidence rather than a long narrative.

Avoid repeatedly transmitting:

- full source files;
- previous long reasoning;
- complete logs when only the failure lines matter;
- repository-wide summaries already known.

Use file paths, symbols, diffs, and targeted excerpts where possible.

---

## 15. Completion behavior

The overall Goal is complete only when the lead determines that:

1. the requested outcome is implemented;
2. relevant work packages are complete;
3. appropriate verification has succeeded;
4. known regressions have been addressed;
5. remaining limitations or risks are understood.

Do not continue improving unrelated parts of the project after the requested Goal and acceptance criteria are satisfied.

Stop when the Goal is complete.

The final response should concisely report:

- what was accomplished;
- meaningful architectural or implementation decisions;
- verification performed;
- remaining risks or limitations, if any.

Do not provide a long narration of agent orchestration unless the user asks for it.