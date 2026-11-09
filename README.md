# LinearUI

**Change a UX pattern with one command, and eventually with one sentence.**

LinearUI is an open research vision from **Setaleur Research**, developed as part of an initiative that started inside **Setaleur Foundation**. This repository holds the idea, a working interactive prototype, and an open invitation to help shape where it goes next.

> Status: **research prototype.** The demo is a simulation. No real project code is modified.

---

## Table of contents

1. [The problem](#1-the-problem)
2. [The idea](#2-the-idea)
3. [Current approach (v0): recipes and feature flags](#3-current-approach-v0-recipes-and-feature-flags)
4. [Where traditional isolation breaks down](#4-where-traditional-isolation-breaks-down)
5. [The new direction: AI without isolation](#5-the-new-direction-ai-without-isolation)
6. [Open research questions](#6-open-research-questions)
7. [Who we are looking for](#7-who-we-are-looking-for)
8. [How to take part](#8-how-to-take-part)
9. [Roadmap](#9-roadmap)
10. [Governance: Setaleur Foundation and Setaleur Research](#10-governance-setaleur-foundation-and-setaleur-research)
11. [The demo](#11-the-demo)

---

## 1. The problem

Teams regularly need to change a core UX pattern: move the profile from a top-bar avatar to a bottom tab, replace a hamburger menu with tabs, restructure onboarding. These changes look small on a design file, but in code they touch navigation, screens, state, deep links, analytics events and tests all at once.

As a result:

- A pattern change takes days or weeks, not minutes.
- Experimenting with an alternative is expensive, so teams commit to a pattern based on opinion rather than evidence.
- Old patterns linger in the codebase long after they are abandoned.

## 2. The idea

Treat a UX pattern as a **swappable unit**, and make swapping it a single, reviewable, reversible operation.

```
/profile-bottom-tab
```

One command rewrites the navigation and screens, records exactly what changed, and can be undone.

## 3. Current approach (v0): recipes and feature flags

The prototype in this repository implements the classic approach:

- **Recipes.** Each pattern (`profile-direct`, `profile-bottom-tab`, `profile-drawer`) is a predefined recipe that knows which files to create.
- **Freezing behind a flag.** When you switch, the previous pattern is not deleted. It is wrapped behind a feature flag (`profileEntry`), kept on disk, and no longer imported.
- **Manifest.** A manifest records the active pattern, the frozen patterns, their files and their flags.
- **History and undo.** Every command is logged with a snapshot, so any step can be reverted.
- **`/remove-old`.** Once you are confident, one command deletes all frozen patterns and their files.

This works well as a proof of concept, and it makes the core promise concrete: *switching is reversible and inspectable.*

## 4. Where traditional isolation breaks down

The flag-and-freeze model is safe, but it carries structural costs that grow with every pattern you try:

| Limitation | Why it matters |
| --- | --- |
| **Dead code accumulates** | Frozen patterns stay in the repository, are never exercised, and silently rot. |
| **Flag debt** | Each pattern adds branches. Removing them later is its own project. |
| **Combinatorial complexity** | With several patterns across several surfaces, the number of valid combinations explodes. |
| **Untestable paths** | Nobody tests the frozen branches, so reviving one is risky. |
| **Recipes are static** | A recipe is a template written for one code structure. Real projects rarely match it. |
| **Does not generalize** | Every new pattern or framework needs a hand-written recipe. |

Isolation solves *safety* by paying with *complexity*. We think there is a better trade.

## 5. The new direction: AI without isolation

The central research bet of LinearUI:

> **If an AI can understand a codebase, plan a transformation, apply it, and prove it is correct, then the old pattern does not need to live on in the code. Version control is the isolation.**

Instead of keeping every alternative alive behind flags, the project always contains **one** clean, active pattern. Switching means *transforming* the codebase, not toggling it.

### 5.1 From command to intent

The command becomes a statement of intent, not a lookup key:

```
Move the profile entry from the top-bar avatar to a bottom tab,
keep the back-stack behavior, and preserve the deep link /u/:handle.
```

### 5.2 Proposed pipeline

1. **Understand.** The AI reads the project: navigation graph, screens, state, routes, analytics, tests.
2. **Plan.** It produces a transformation plan: files to change, files to delete, behavior to preserve. The plan is shown to a human first.
3. **Apply on a branch.** Changes are made on an isolated Git branch, never on the main line.
4. **Verify automatically.** Type-check, lint, unit tests, navigation tests, and visual snapshots before and after.
5. **Report.** A structured report states what changed, what was verified, and what could not be verified.
6. **Human review and merge.** A person approves. Nothing merges on its own.
7. **Rollback.** Reverting is a Git revert plus a lightweight manifest entry. No flags are required.

### 5.3 Recipes become contracts

In v0 a recipe is a block of code. In the AI-driven model a recipe is a **contract**: a description of what must be true after the change.

Example contract for `profile-bottom-tab`:

- The profile is reachable in one tap from any top-level screen.
- The tab bar is visible on all top-level screens and hidden on modal screens.
- Back navigation from the profile returns to the previously selected tab.
- The deep link `/u/:handle` still resolves to the profile.
- No unused code from the previous pattern remains.

The AI is free to implement this however the project's structure demands. The contract is what gets verified. This is what lets the idea generalize across codebases and frameworks without hand-written templates.

### 5.4 What replaces the flags

| v0 (isolation) | Direction (AI, no isolation) |
| --- | --- |
| Feature flag per pattern | Git branch per transformation |
| Frozen code kept on disk | Old pattern lives in Git history only |
| Static recipe templates | Behavioral contracts |
| Manual `/remove-old` cleanup | Nothing to clean: dead code is never created |
| Undo via snapshot | Undo via Git revert and manifest entry |

### 5.5 Design principles

- **The repository stays clean.** One active pattern, no dormant branches.
- **Humans decide.** The AI proposes and proves; people approve.
- **Everything is auditable.** Plan, diff, verification results and manifest are always visible.
- **Reversible by default.** Any transformation can be undone with standard tools.
- **Verification before trust.** A change that cannot be verified is reported as unverified, never as done.
- **Framework-agnostic at the contract level.** Contracts describe behavior, not libraries.

### 5.6 Optional: flags as a research tool, not a requirement

We do not rule flags out. For **live A/B experiments with real users**, a runtime flag is the right tool. The difference is intent: flags become an *experimentation* choice made deliberately, rather than the default safety mechanism for every change.

## 6. Open research questions

These are the questions we want help answering. Several need real scientific rigor, not just engineering.

1. **Correctness.** How do we measure that a transformed UI is behaviorally equivalent where it should be, and different only where intended?
2. **Contracts.** What is the right language for a UX contract: natural language, a DSL, executable tests, or a mix? How precise can it be without becoming a second codebase?
3. **State and navigation.** How do we guarantee that deep links, restored state, back stacks and in-flight navigation survive a pattern change?
4. **Visual verification.** How reliable are snapshot and vision-model checks for judging that a screen still communicates what it should?
5. **Blast radius.** How do we bound what the AI is allowed to touch, and detect when a change leaks outside the intended surface?
6. **Evaluation.** What benchmark would let us compare approaches fairly? We would like a public set of codebases, patterns and contracts.
7. **Human factors.** What does a reviewer need to see to approve a large AI-generated change with confidence?
8. **Failure modes.** When does the AI confidently produce a plausible but wrong transformation, and how can the pipeline catch it?
9. **Cost and scale.** Does the approach hold on large monorepos and mixed-platform apps?
10. **Evidence for UX decisions.** Can cheap, safe pattern switching change how teams *decide* between patterns?

## 7. Who we are looking for

- **Mobile and web engineers** who have lived through a painful navigation refactor.
- **UX researchers and designers** interested in testing patterns empirically instead of by opinion.
- **AI and program-transformation researchers** working on code agents, verification, or evaluation.
- **Testing and QA engineers** with ideas on behavioral and visual verification.
- **Teams** willing to try the approach on a real (non-critical) project and share what happened.

You do not need to be an expert in all of these. A sharp question or a real-world failure story is a valuable contribution.

## 8. How to take part

- **Discussions.** Share ideas, objections and use cases. Critical feedback is welcome.
- **Issues.** Open one for a concrete research question, a missing pattern, a bug in the demo, or a proposed contract.
- **Pull requests.** Improve the demo, add pattern recipes or contracts, or refine this document.
- **Field reports.** Tried something similar? Tell us what broke. Failure reports are some of the most useful input we can get.

A good first contribution: write a **contract** for a pattern you know well (for example, `settings-as-modal`) and open it as an issue.

## 9. Roadmap

| Phase | Focus | Status |
| --- | --- | --- |
| **0** | Interactive prototype of the flag-based model | Done |
| **1** | Public research vision and contract format draft | In progress |
| **2** | Reference pipeline: read, plan, apply on branch, verify | Planned |
| **3** | Verification suite: type, test, navigation and visual checks | Planned |
| **4** | Public evaluation set and first results | Planned |
| **5** | Pilot with real teams, report findings openly | Planned |

The roadmap is a direction, not a promise. Research results may change it.

## 10. Governance: Setaleur Foundation and Setaleur Research

LinearUI began as a project within **Setaleur Foundation**.

- **Setaleur Foundation** remains responsible for realizing this vision and for expanding it over time, because the project originated there.
- **Setaleur Research** carries the research work and keeps broad flexibility wherever a decision needs real scientific judgment, so that choices about method, evaluation and direction are made on what best serves the project.

In practice: the Foundation holds the long-term commitment to the vision, and Setaleur Research has the freedom to follow the evidence, including changing the plan when the evidence demands it.

## 11. The demo

`linearui-demo.html` is a single, dependency-free HTML file. Open it in a browser.

- It shows the **v0 flag-based model** described in section 3.
- It is a **simulation**: no real project code is changed.
- Press `/` to focus the command bar, or use the guided tour to see switching, `/remove-old` and `/undo` in action.

The AI-driven direction in section 5 is a research proposal. It is not implemented in the demo yet.

---

*LinearUI is an open-source research project by Setaleur Research, with Setaleur Foundation as the originating organization. Currently at the prototype stage.*
