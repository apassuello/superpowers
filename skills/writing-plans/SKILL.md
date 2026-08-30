---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, the contract (signatures + behavioral criteria + normative cases), testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Spec Baseline

**A plan is derived from a spec, never written in place of one.** The spec
owns the behavior; the plan owns the sequencing and the contracts that make
independent implementers converge. Before writing any task:

**1. Locate the spec.** Look in `docs/superpowers/specs/` (or wherever this
project keeps design docs) for the design document this plan implements.
Record its path in the plan header's **Spec:** field.

**2. If there is no spec, stop and get one.** Do not plan against an
unwritten design — you will invent behavior at plan time, in prose nobody
ratified, and every downstream implementer will treat your invention as
requirements. Say so plainly and invoke `superpowers:brainstorming` to
produce the spec first. A one-task change to already-specified behavior is
the only exception, and even then name the spec section it changes.

**3. Check the spec is not stale.** Skim the documents the spec depends on
— the ones it cites, and the README / architecture docs describing the
components it touches. If any contradicts the current code or the spec,
report the contradictions to your human partner before planning. Planning
on top of a stale document propagates the staleness into every task.

**4. Behavior is traceable, not invented.** Every behavioral criterion and
normative case in a task comes from the spec. If you cannot point to where
the spec says it, you are authoring the spec — see **Spec Amendment**:
amend, commit, and get it reviewed, rather than burying a new requirement
inside a task.

Two things count as traceable: a value the spec **states**, and a value
that follows from spec statements by arithmetic or direct composition —
the spec caps a field at 40 characters and defines the ellipsis, so the
39-character cut point is derived, not invented. Anything requiring a
*choice* between defensible answers is not a derivation, however obvious
it feels: tie-breaks, precedence between two rules, what happens at a
boundary the spec never mentions. Those go through Spec Amendment. When
unsure which side a value falls on, ask whether a second planner working
from the same spec would necessarily land on the same value.

The plan and the spec must agree at all times. If they diverge, the spec
governs and one of the two is wrong.

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Spec:** [path to the spec/design doc this plan implements — the plan
argues from the spec, so the spec travels with it; executors read both]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Artifact Content Rule

A plan — and the task briefs derived from it — states **what the code must satisfy**, not the pre-written solution. The implementer (a fresh subagent, or an inline TDD cycle) writes both the implementation and the tests.

**Contracts stay, implementations go.** A runnable test file is an
implementation too — of verification. Cross-task coherence comes from
pinned signatures, schemas, wire formats, and exact normative values, not
from shipping the test bodies.

**Belongs in a task (the contract):**
- Exact file paths (create / modify / test)
- Function/class/type **signatures & API surface** — pin these; independent implementers drift on names and interfaces otherwise
- Naming conventions and code-style directives
- Data schemas / formats / wire contracts — serialization key order, sort keys, the exact envelope shape
- Command / API surface grammar
- **Behavioral acceptance criteria** — what it must do, edge cases, error behavior
- **Normative cases** — a compact table of exact input → output/error values the implementation must produce, traceable to the spec (see Task Structure)
- Exact commands; commit commands
- Verbatim configuration / CI / schema files

**Never in a task:**
- The pre-written **implementation of the task's core logic** — function bodies, algorithms. That is precisely what the implementer is dispatched to produce.
- **Runnable test files** — imports, fixtures, `setUp`/teardown, test classes, test-function definitions, `# tests/test_x.py` headers. Test *design* is the implementer's job, done against the code in front of them; a transcribed test was designed once, in the abstract, and never re-derived. Plan-time test code is also unexecutable and therefore unverified — its bugs surface as an implementer unsure whether to fix the plan's test or their own code.
- **Expected test counts and test-runner error strings** — `PASS (5 tests OK)`, `FAIL with ImportError`. These bake in test-splitting decisions that belong to the implementer, and go stale the moment they split a case in two.

**Narrow exception:** a genuinely tricky algorithm may appear as a *clearly-labeled reference hint*, never as "the step." Rule of thumb: no pre-written implementation of the task's core logic.

Forbidding the solution does NOT lower the specificity bar (see No Placeholders): "show how" is satisfied by the contract — signatures, schemas, criteria, and exact normative values — not by a pre-written body and not by vague prose. The red flag is **criteria two implementers could not converge on**, never *absent code*.

**The revision test.** A ratified change to the command surface or the wire
format should be absorbable by editing prose and table values. If it forces
you to hand-edit assertion arguments inside embedded test bodies across
several tasks — each edit unverifiable until some implementer runs it —
the plan is carrying code it should not have.

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

**Contract — signature the implementation must expose:**

```python
def slugify(text: str) -> str: ...
```

**Behavioral criteria:**
- Lowercases all ASCII letters.
- Replaces each run of non-alphanumeric characters with a single `-`.
- Strips leading and trailing `-`.
- Empty or all-punctuation input returns `""`.
- Non-`str` input raises `TypeError("text must be str")`.

**Normative cases** — the implementer's tests must cover every row; exact values:

| input | output |
|---|---|
| `"Hello World"` | `"hello-world"` |
| `"  A, B & C!  "` | `"a-b-c"` |
| `"!!!"` | `""` |
| `""` | `""` |
| `None` | raises `TypeError("text must be str")` |

- [ ] **Step 1: Write the failing test**

Design the test yourself from the contract above. Cover every normative
case, plus any edge case the contract implies that the table does not
name. Do NOT expect pre-written test code here — see
`superpowers:test-driven-development` and its `writing-good-tests.md`.

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test_slugify.py -v`
Expected: FAIL — `slugify` does not exist yet. Confirm the failure is the
*absence of the implementation*, not a broken test.

- [ ] **Step 3: Implement to pass the test**

Write the body to satisfy the contract and the tests you wrote in Step 1.

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test_slugify.py -v`
Expected: PASS, output pristine. (No test count — how many tests cover
these cases is your call.)

- [ ] **Step 5: Commit**

```bash
git add tests/path/test_slugify.py src/path/slugify.py
git commit -m "feat: add slugify helper"
```
````

Pin errors the same way you pin outputs — exact exception type and exact
message, stated as a behavioral criterion and as a normative-case row.
"Handles bad input appropriately" is not a normative case. Note that every
row in the example above answers to a criterion above it: a table row no
criterion accounts for is unsourced behavior, and the plan is the wrong
place to introduce it (see Spec Baseline).

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" / "test the happy path and errors" (without naming the normative cases and their exact values)
- "Similar to Task N" (repeat the contract — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how — "showing how" means the contract (exact signatures/API, schemas, and normative case values), NOT vague prose, NOT a pre-written implementation body, and NOT a transcribed test file (see Artifact Content Rule)
- Behavioral criteria so loose that two competent implementers would build measurably different things — this, not absent code, is the specificity failure to hunt for
- References to types, functions, or methods not defined in any task
- Behavior that appears in no spec section (see Spec Baseline — amend the spec instead of smuggling it into a task)

## Remember
- Exact file paths always
- Every task traces to the spec; the plan and the spec agree or one is wrong
- Show the contract — signatures, schemas, behavioral criteria, normative cases with exact values. The implementer writes the tests AND the body; never pre-write either
- Exact commands; expected output without test counts or runner error strings
- DRY, YAGNI, TDD, frequent commits

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage, both directions:** Skim each section/requirement in the spec — can you point to a task that implements it? Then read your tasks — can you point to the spec section each behavioral criterion and normative case came from? Unsourced behavior means you authored spec inside the plan; take it through **Spec Amendment** instead.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Artifact content scan:** Find every block of code in the plan — fenced, indented, or written out as numbered pseudo-test lines in prose. Fences are the common case, not the only one; a rule that only checks fences is one indent away from being bypassed. Every block must be a signature, a schema, a config/CI file, a command, or a clearly-labeled reference hint. If a block contains imports, fixtures, test-function definitions, or a function body implementing the task's core logic, delete it and replace it with the contract it was standing in for. Then search for expected test counts (`2 passed`, `5 tests OK`) and runner error strings (`ImportError`, `NameError`) and strike those too.

**4. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**5. Convergence check:** Go task by task — not a fixed sample, which leaves the vague criteria in task 9 undetected. For each task's vaguest criterion, ask: could two competent implementers, working from the criteria and normative cases alone, build measurably different things? If yes, add the missing exact values — not test code.

If you find issues, fix them inline. No need to re-review — just fix and move on. If you find a spec requirement with no task, add the task.

## Spec Amendment

While writing the plan you may surface gaps, new needs, or additions the spec missed — a missing requirement, an interface the spec never pinned, an edge case with no home. When you do, you MAY complete or correct the spec to close them: amend the spec, commit the change as its own commit, and note what changed and why. Then plan against the corrected spec.

**Amendments get reviewed — they are new design, not bookkeeping.** Before
planning against an amended spec, put the amendment in front of a reviewer:
your human partner, or a fresh agent with the spec and the amendment diff.
An amendment that slips through unreviewed is a requirement nobody ratified,
and it will be implemented as though it had been.

Spec *authorship* still belongs to the brainstorming phase — this is a planning-phase feedback loop, not a transfer of ownership. Amend to fix what planning revealed; don't redesign the spec.

The same rule applies later: when *implementation* forces a behavior change,
it goes back to the spec and the plan under review — never absorbed silently
into a task. See `superpowers:subagent-driven-development`.

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review

**If Inline Execution chosen:**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
- Batch execution with checkpoints for review
