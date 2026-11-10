---
name: phased-tickets
description: Turn a plan, spec, or conversation into a phased rollout of tracer-bullet tickets, then execute it one phase at a time. Use whenever the user asks to break work down, plan a refactor or migration, split a spec into tasks or tickets, sequence a large feature, or asks "how should we approach this". Also use when a change touches many files or several systems and the user has not asked for phases by name, or when a change would be hard to land, review, or undo in one shot.
---

# Phased Tickets

Break work into **phases**, and each phase into **tickets**.

A phase is a checkpoint the project could safely stay at: it builds, its tests pass, and it is one coherent commit. A ticket is a tracer-bullet slice inside that phase, small enough to finish in one go and declared with the tickets that block it.

Phases keep the rollout reviewable and reversible. Tickets keep each phase honest about what actually has to happen, and in what order. Neither is ceremony: a two-file fix needs neither, so say so and just do the work.

## Process

### 1. Gather context

Work from what is already in the conversation. If the user passes a reference (a spec path, an issue number or URL), fetch it and read the full body and comments. If the request is vague, ask what the finished change should let a user do before proposing anything.

### 2. Read the code and the roadmap

Read the code the change touches, so the phases reflect what is actually there rather than what the spec assumes. Use the project's glossary vocabulary in titles and descriptions, and respect ADRs in the area you are touching.

Check for a roadmap, usually in `docs`. If this change is on it, plan the phases as steps within that entry, refer to it by its existing name, and follow its order and scope. Raise a conflict with the plan rather than quietly rewriting it.

Look for prefactoring that makes the real change easier. "Make the change easy, then make the easy change." Prefactoring belongs in its own ticket, before the work it unblocks.

### 3. Draft the phases

Aim for three to five phases, each with a one-line label:

```
Phase 1: Scaffolding
Phase 2: Data layer
Phase 3: UI wiring
Phase 4: Cleanup and tests
```

Name them plainly and use one scheme for the whole rollout. Do not mix letters and numbers within a plan.

Start with scaffolding: the files, interfaces, types, config, or stubs that later phases build against. Skipping straight to real logic tends to force rework once a second or third piece reveals the structure needed a different shape. Only scaffold what a later phase in this plan will use. If there is no structure to set up, such as a rename or a removal, say so and start with the first real step.

Phase a change when landing it in one shot would be hard to review or hard to undo: it touches many files or several systems, it has a natural dependency order, or a partial version is still safe to leave in place.

### 4. Slice each phase into tickets

Each ticket is a **tracer bullet**: a narrow but complete path through every layer it touches, not a horizontal slice of one layer. A finished ticket is demoable or verifiable on its own, and small enough to finish without losing the thread.

Give every ticket its **blocking edges**: the other tickets that must finish first. A ticket with no blockers can start immediately, and those are the ones to work first.

Prefer fewer, meatier tickets. A ticket that only makes sense alongside three others is not a slice, it is a layer, so fold it into the ticket that makes it observable.

**Wide refactors are the exception to vertical slicing.** A wide refactor is one mechanical change, such as renaming a column or retyping a shared symbol, whose blast radius fans across the whole codebase, so one edit breaks thousands of call sites and no vertical slice can land green. Sequence it as expand-contract instead:

1. **Expand**: add the new form beside the old, so nothing breaks.
2. **Migrate**: move call sites over in batches sized by blast radius, such as per package or per directory. Each batch is its own ticket, blocked by the expand, and CI stays green batch to batch because the old form still exists.
3. **Contract**: delete the old form once no caller remains, blocked by every migrate batch.

When even the batches cannot stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket. Green is promised only there.

### 5. Quiz the user

Present the breakdown as phases with their tickets. For each ticket show:

- **Title**: short and descriptive.
- **Blocked by**: which other tickets gate it, or none.
- **Delivers**: the end-to-end behaviour this ticket makes work, from the user's perspective.

Then ask:

- Does the granularity feel right, too coarse or too fine?
- Are the blocking edges correct, or does a ticket depend on something that does not actually gate it?
- Should any phase or ticket be merged or split?

Iterate until the user approves. Do not start editing before that approval.

### 6. Save the plan

Save the approved plan to `stash.local/phased-tickets/<feature-slug>.md` in the project. `stash.local` is git-ignored, which is what makes it the right home for a working checklist; a roadmap is a public document and should not grow phase detail it does not already carry.

Keep one plan file per rollout, with the checklist at the top and the phase detail below:

```markdown
# <Feature> rollout

Source: <spec path, issue, or "conversation">
Roadmap: <entry name, or "none">

- [ ] Phase 1: Scaffolding
- [ ] Phase 2: Data layer
- [ ] Phase 3: UI wiring

## Phase 1: Scaffolding

**Delivers:** what is true about the project once this phase lands.
**Done when:** the check that proves it.

### T1 - <title>
Blocked by: none
Delivers: ...

### T2 - <title>
Blocked by: T1
Delivers: ...
```

If the project's roadmap already tracks work at this level of detail, use the roadmap in its own format instead, and include each update in that phase's commit.

### 7. Work the rollout

Work the frontier: any ticket whose blockers are done. Inside a phase that usually means top to bottom.

Expect corrections mid-rollout. When the user changes direction, update the remaining phases and tickets rather than holding to the original plan.

Finish each phase before starting the next:

1. Build it and run the tests. Run the program too when the phase changes what it does. Do not mistake unit tests for verification of the program itself.
2. Commit locally as one coherent piece of work. Do not push.
3. Report the commit, what was checked, what was not, and any risks.
4. Update the checklist in the plan file and include that update in the same commit.
5. Show the updated checklist and wait for a go-ahead before the next phase, unless the user asked to run several through.

Check the plan against `git log` when picking the work back up, since the code is the record of what actually landed. When the rollout finishes an entry on the roadmap, update that entry in the final phase's commit, even if the checklist lived in `stash.local`.

## Reporting

Keep the report short enough to read in one pass:

```
Phase 2: Data layer - done
Commit: 4f2a1c9
Checked: build, tests, ran the importer against sample data
Not checked: Windows path handling, no VM handy
Risks: schema change is forward compatible, but old readers ignore the new column
```

Then the checklist, then the question: continue, or adjust.

## What Not To Do

- Do not phase small, self-contained changes. Ceremony on a two-file fix wastes the user's review time.
- Do not invent phases to hit a count. Three phases that fit the work beat five that pad it.
- Do not write tickets as layer-by-layer implementation lists. "Add the column" is not a ticket; "import a file with the new column and see it in the UI" is.
- Do not publish to an issue tracker unless the user names one. Local `stash.local` is the default.
- Do not claim a phase is done when the build or tests were not run, or when only the tests were run.
