---
name: anti-frustration-features
description: Design and implement anti-frustration features (quality-of-life mechanics) for games, including checkpoints and save recovery, retry and death loops, input forgiveness, dynamic difficulty, bad-luck protection, anti-waste safeguards, hints and navigation, accessibility options, and softlock escape hatches. Use whenever the user is programming or reviewing game mechanics and mentions difficulty, player frustration, quality of life, checkpoints, retries, grind, RNG, softlocks, onboarding, tutorials, accessibility, or "make this less annoying" — even when they do not use the term "anti-frustration".
---

# Anti-Frustration Features

Frustration is not the same thing as difficulty. Difficulty is the part of the game the player signed up to solve. Frustration is everything else: time lost to repetition, progress lost to an unrecoverable mistake, rules that hide information the player needs, inputs that demand more precision than the game's feedback supports, or a punishment that lands long after the mistake.

Anti-frustration work removes the second without touching the first. A feature that makes the game easier is not automatically a good one; a feature that makes the intended challenge clearer, fairer, or faster to retry usually is.

## Core model

Classify every frustration by what the player loses:

- **Time** — waiting, walking, replaying, watching the same scene, grinding.
- **Progress** — levels, items, currency, map completion, a long run.
- **Confidence** — not knowing whether a failure was their fault, the game's, or bad luck.
- **Agency** — being trapped, softlocked, or forced into a choice they cannot undo.

Then ask whether that loss is the challenge. Losing a boss fight is intended. Replaying twenty minutes of easy enemies to reach the boss is usually not. Losing a run in a roguelike is intended. Losing it because a random number generator produced an unplayable opening is usually not.

When in doubt, name the skill or decision the game is actually testing. Anything that punishes the player without testing that skill is a candidate.

Most features fall into three families. **Safety nets** bend or suspend the rules to prevent an unfair or unwinnable state. **Conveniences** reduce the time and steps between intention and action. **Accessibility and forgiveness** widen the input and perception channel so the game meets the player where they are. The families overlap; the label matters less than knowing which loss you are removing.

### The assistance ladder

Prefer the earliest rung that solves the problem. Earlier rungs preserve more of the game's stakes and are harder to notice, which keeps the feature from feeling like a verdict on the player.

1. **Inform** — make the state legible: a timer, a marker, a warning, a damage preview, a "you can't use that here" message.
2. **Prevent** — stop the mistake before it happens: confirmations, undo windows, locks, guard rails, disabled destructive actions.
3. **Forgive** — absorb the mistake: coyote time, input buffering, mercy invincibility, a missed input that still counts, partial refunds.
4. **Recover** — let the player get back quickly: checkpoints, retry-from-boss, corpse retrieval, fast travel, retain progress on death.
5. **Skip** — let the player bypass the content: skip cutscene, skip minigame, assist mode, difficulty reduction.

Skipping is legitimate, but it is the heaviest tool. Reach for it when the content is optional, inaccessible, or already proven; not as the first response to a mechanic that is merely unclear.

## Workflow

### 1. Establish the intended challenge

Write one sentence: "This game is about ___." Then list the skills and decisions that sentence implies. This is the boundary that keeps anti-frustration work from sanding off the game's identity. For a horror game, vulnerability may be the point. For a precision platformer, execution is the point — but the *readability* of the jump is not.

### 2. Find the friction

Use whatever evidence exists, in this order:

- Watch a real player, or your own recording, and note where they stop, sigh, backtrack, alt-tab, or quit.
- Read telemetry: deaths per checkpoint, retries per boss, time between attempts, quit points, item waste, hint usage.
- Inspect the mechanics directly: save points, resource costs, RNG tables, input windows, respawn distance, menu depth.

Do not start from a feature list. Start from a moment that feels bad, then choose the feature.

### 3. Classify and choose the rung

For each moment, decide:

- What does the player lose?
- Is that loss testing the intended skill?
- Which rung fixes it with the least damage to the stakes?

A useful test: if the player understood the situation perfectly and still lost, is that acceptable? If yes, the loss is intended and the feature should focus on recovery, not prevention. If no, the problem may be information or fairness, not difficulty.

### 4. Specify behavior precisely

A feature is not designed until these are answered:

- **Trigger** — what state, count, or event activates it? Is it per-attempt, per-session, per-save?
- **Effect** — exactly what changes? Which values, for how long, with what stacking rules?
- **Limits** — caps, cooldowns, diminishing returns, one-shot flags, difficulty interaction.
- **Visibility** — what does the player see before, during, and after? Silence can feel like cheating; too much can feel like pity.
- **Opt-out** — can challenge-run players disable it? Is it on by default?
- **Persistence** — does it survive save/load, scene changes, and app restarts?
- **Failure modes** — what happens if the player exploits it, ignores it, or triggers it twice?

### 5. Implement with the state model first

Most anti-frustration bugs come from state, not from the feature itself. Before writing the effect, define:

- Where the authoritative state lives (save data, run state, scene state, UI state).
- Whether the feature is deterministic and replay-safe.
- How it interacts with save migration, difficulty changes, and multiplayer authority.
- How it can be disabled for tests, replays, and challenge modes.

### 6. Test the moments, not just the code

Acceptance criteria should describe player-visible behavior. Cover the edges:

- The player triggers the feature on the first attempt and the hundredth.
- The player saves and quits mid-recovery, then returns.
- The player deliberately tries to farm or exploit it.
- The player is already at the cap, or the feature would overflow a resource.
- The feature interacts with another assist (for example, a checkpoint plus a difficulty drop).
- The feature is disabled, and the game still behaves correctly.

### 7. Tune with evidence

Assistance that is too weak does nothing; assistance that is too strong removes the game. Watch the distribution, not the average: if only the bottom 5% use it, it may be too hidden; if everyone uses it every run, it may be masking a core balance problem. Prefer tuning the trigger threshold over rewriting the feature.

## Output format

When asked to review or design, produce a short audit followed by implementable features. Use this structure.

```markdown
## Intended challenge
One sentence, plus the skills/decisions it implies.

## Frustration audit
| Moment | What the player loses | Intended challenge? | Evidence |
|---|---|---|---|
| ... | time / progress / confidence / agency | yes / no / partly | playtest, telemetry, code path |

## Proposed features
### <Feature name>
- **Family and rung:** <safety net / convenience / accessibility> · <inform / prevent / forgive / recover / skip>
- **Fixes:** <the audit row it addresses>
- **Trigger:** <exact condition>
- **Behavior:** <exact effect, values, duration>
- **Limits:** <caps, cooldowns, stacking, difficulty interaction>
- **Visibility:** <what the player sees>
- **Opt-out:** <how to disable, if applicable>
- **Acceptance tests:** <player-visible scenarios, including edge cases>

## Implementation plan
Ordered steps. Call out state ownership, persistence, save migration, telemetry, feature flags, and what must stay deterministic.

## Risks and tradeoffs
What this could make too easy, what it might hide, and what to measure.
```

Keep the audit to the moments that matter. A short, specific review is more useful than an exhaustive list of every possible quality-of-life feature.

### Worked example

Request: "Players keep quitting at the second boss. Make it less punishing without making it easy."

```markdown
## Intended challenge
Reading attack patterns and managing stamina over a long fight.

## Frustration audit
| Moment | What the player loses | Intended challenge? | Evidence |
|---|---|---|---|
| Run back from the bonfire | 3-4 minutes of easy enemies per attempt | No - traversal is already proven | 6 of 8 playtesters stopped after 3 wipes |
| Phase 2 transition | Health is not restored and the phase opens with a grab | Partly - the grab is fair, the resource drought is not | 71% of deaths occur within 10s of phase 2 |
| "You died" screen | A menu tour before retry | No | 22s average death-to-control |

## Proposed features
### Boss-door checkpoint
- **Family and rung:** safety net / recover
- **Fixes:** run-back row
- **Trigger:** first entry into the boss arena
- **Behavior:** persistent checkpoint outside the arena with the current loadout
- **Limits:** one per boss; does not refill consumables
- **Visibility:** visible marker and a short "checkpoint" toast
- **Opt-out:** disabled in challenge-run mode
- **Acceptance tests:** quit and reload at the door restores the checkpoint; the run-back is skipped; consumables are unchanged

### Phase-2 resource floor
- **Family and rung:** safety net / forgive
- **Fixes:** phase-2 resource drought
- **Trigger:** phase 2 begins with less than 30% health
- **Behavior:** restore health to 30% and refill one stamina chunk; no effect above the floor
- **Limits:** once per attempt; does not stack with a retry bonus
- **Visibility:** brief, diegetic "second wind" effect
- **Opt-out:** covered by the assist-mode setting
- **Acceptance tests:** triggers at 29%, not at 31%; triggers once; does not fire above the floor

## Implementation plan
1. Add `bossCheckpoint` to run state, keyed by encounter ID, and persist it with a schema version.
2. Snapshot loadout and world flags at arena entry, excluding consumables.
3. On retry, restore the snapshot and skip the approach trigger.
4. Implement `phaseResourceFloor` in encounter logic, using the same health value as the damage system.
5. Expose both behind assist flags; default on, off in challenge mode.
6. Track attempts, death-to-control time, and phase-2 death rate.

## Risks and tradeoffs
The checkpoint may make the boss feel detached from the run. Measure completion and retry rate, not just deaths. The resource floor could erase the stamina lesson; watch whether players stop managing stamina entirely.
```

## When not to intervene

Anti-frustration is not a universal good. Some games are *about* the friction:

- **Horror** may depend on vulnerability, scarcity, and the dread of a long run back.
- **Roguelikes** may depend on loss and run-scoped consequences.
- **Comedy and satire** may use tedium or unfairness as the joke.
- **Puzzle games** may depend on the player sitting with an unclear problem.
- **Frustration simulators** and deliberate challenge games exist; removing the friction removes the product.

In those cases, the work is not to add assistance but to make the intended friction legible, fair, and recoverable. A horror game can keep the long run back and still fix a checkpoint that saves the player into a dead end.

## Guardrails

- **Preserve the fantasy.** If a feature contradicts the world's rules, either justify it in fiction or make it invisible. A respawn that the game acknowledges can feel generous; one that breaks its own rules can feel careless.
- **Preserve stakes.** Failure needs a cost, or success has no weight. Choose which cost to remove — usually time and progress before resources and consequence.
- **Assistance is not a verdict.** Neutral labels ("assist mode", "retry", "hint") land better than "easy mode" or "you seem to be struggling". Make help opt-in where possible, and never mock the player for using it.
- **Do not punish improvement.** Invisible rubber-banding that makes a player's better performance produce worse outcomes destroys trust. If difficulty adapts, adapt to repeated failure, not to success.
- **Prefer prevention over compensation.** A confirmation prompt is cheaper and kinder than a refund system.
- **Do not make the assist the dominant strategy.** If the fastest way to play is to fail on purpose, the feature is mispriced.
- **Keep challenge runs possible.** Speedrunners, accessibility testers, and difficulty purists need a clean way to turn assistance off.
- **Measure the right thing.** "Players died fewer times" is not success if they also stopped playing. Pair outcome metrics with engagement and comprehension metrics.

## Reference

Read [references/feature-catalog.md](references/feature-catalog.md) when you need concrete patterns for a specific friction source. It is organized by player problem — failure, waiting, navigation, RNG, resources, softlocks, difficulty, accessibility, multiplayer — with implementation notes and pitfalls for each. Use it to choose a pattern, not as a checklist to apply wholesale.

The framing comes from the [Anti-Frustration Features](https://tvtropes.org/pmwiki/pmwiki.php/Main/AntiFrustrationFeatures) trope, which catalogs examples across genres. The catalog here is a design and engineering adaptation, not a copy of that page.
