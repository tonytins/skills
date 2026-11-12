# Anti-Frustration Feature Catalog

Organized by the player problem, not by genre. Each pattern lists when it helps, how to implement it, and the failure mode that makes it backfire. Pick the lightest pattern that addresses the actual loss.

## Contents

1. [Failure and progress loss](#1-failure-and-progress-loss)
2. [Retry loops and downtime](#2-retry-loops-and-downtime)
3. [Repetition, backtracking, and traversal](#3-repetition-backtracking-and-traversal)
4. [Waiting and timers](#4-waiting-and-timers)
5. [Navigation and direction](#5-navigation-and-direction)
6. [Hidden state and legibility](#6-hidden-state-and-legibility)
7. [Execution and input](#7-execution-and-input)
8. [Randomness and bad luck](#8-randomness-and-bad-luck)
9. [Resources and irreversible choices](#9-resources-and-irreversible-choices)
10. [Softlocks and dead ends](#10-softlocks-and-dead-ends)
11. [Difficulty and onboarding](#11-difficulty-and-onboarding)
12. [Accessibility and perception](#12-accessibility-and-perception)
13. [Cognitive load and interfaces](#13-cognitive-load-and-interfaces)
14. [Multiplayer and other players](#14-multiplayer-and-other-players)
15. [Cross-cutting implementation notes](#15-cross-cutting-implementation-notes)

---

## 1. Failure and progress loss

The player failed. The question is how much unrelated work dies with them.

### Checkpoints and autosave

**Use when** replaying from the last manual save would repeat content the player has already mastered.

**Implement**
- Place checkpoints before bosses, after major traversal, and at natural narrative boundaries.
- Autosave on checkpoint activation, not on a timer, so the save reflects a coherent state.
- Make saving atomic: write to a temp file, flush, then rename. Keep a previous slot so a crash mid-save cannot destroy the run.
- Store a save version and a migration path. Anti-frustration features often change save shape after release.
- Separate "checkpoint reached" from "checkpoint restored" so the feature is idempotent if the player reloads twice.

**Pitfall** Checkpoints that save the player into an unwinnable state. Validate that the restored state has the resources needed to continue, or provide an escape hatch.

### Retain progress on death

**Use when** the death penalty is repetition rather than consequence.

**Implement** Keep permanent or meta-progress (XP, unlocks, map discovery, puzzle solutions) and reset only the run-scoped state that creates tension. A common split is: keep knowledge and unlocks, lose position and consumables.

**Pitfall** If nothing is lost, death stops being a decision. Keep at least one cost that matters to the intended fantasy — time, position, a consumable, or a run modifier.

### Corpse recovery with a safe spawn

**Use when** losing carried resources creates a recovery quest.

**Implement** Spawn the recovery object at the death location, but move it outside boss arenas or hazard-only rooms. Add a purchasable or consumable recovery service for unreachable cases. Expire it only after a clear second death, and warn before that happens.

**Pitfall** A corpse inside the boss room turns recovery into a second, harder version of the fight that already killed the player.

### Continue with partial progress or a bonus

**Use when** a hard segment is a spike rather than the core loop.

**Implement** On continue, restore at the segment start with full health, a small resource bonus, or a one-time buff. Scale the bonus with consecutive failures at the same checkpoint, and cap it.

**Pitfall** Hidden pity that the player cannot refuse can feel insulting, and can make a win feel unearned. Show the assistance or make it opt-in.

---

## 2. Retry loops and downtime

The distance between "I want to try again" and "I am trying again" is often the largest source of frustration in a skill game.

### Retry from the encounter

**Use when** the player is practicing a boss, puzzle, or challenge room.

**Implement** Keep a lightweight encounter snapshot separate from the full save: player state, loadout, and encounter seed. On retry, restore the snapshot and skip the approach, dialogue, and load screens. Offer "retry" and "leave" as distinct choices.

**Pitfall** Restoring too much can erase a choice the player made during the attempt. Define which state is attempt-scoped and which is run-scoped.

### Mid-encounter checkpoints

**Use when** an encounter has distinct phases and a late-phase failure means repeating solved phases.

**Implement** Create a checkpoint at phase transitions, carrying forward health and resources as designed. Make the checkpoint a real save state, not just a respawn point, so quitting mid-fight behaves predictably.

**Pitfall** Checkpointing after a phase that consumed unique resources can leave the player without the tools the next phase expects.

### Fast restart and input-to-retry latency

**Use when** the core loop is repeated attempts.

**Implement** Bind restart to a single input, skip confirmation for a clean retry, and preload the encounter. Measure the time from death to control; aim for seconds, not a menu tour.

**Pitfall** An accidental instant restart can erase a good attempt. Use a short hold or a deliberate input, not a tap that overlaps with combat.

### Death recap and cause of failure

**Use when** players cannot tell why they died.

**Implement** Show the last damage source, a short timeline, or a replay. Keep it skippable. If the death was avoidable, show the tell that preceded it.

**Pitfall** A recap that blames the player for an unclear mechanic makes the game feel more hostile, not less.

---

## 3. Repetition, backtracking, and traversal

### Fast travel and return points

**Use when** the player has already proven they can cross an area.

**Implement** Unlock travel at discovered nodes. Put a return point at the end of a dungeon or a one-way section. Preserve the first traversal as content; compress the second.

**Pitfall** Travel that skips a zone the player has not actually explored breaks pacing and sequence.

### One-way shortcuts

**Use when** the level is a loop and the player must return.

**Implement** Open a shortcut from the far side, ideally visible from the entrance so the player understands the payoff. Make it persist.

**Pitfall** Shortcuts that are easy to miss turn into a knowledge check rather than a convenience.

### Skip repeated content

**Use when** a scene, minigame, or tutorial is mandatory on a replay.

**Implement** Track completion per save, offer "skip" on subsequent encounters, and allow the player to watch again. For minigames, offer skip after a failure threshold or from the options menu.

**Pitfall** Skipping a scene that contains a choice or a clue can strand the player. Separate "skip cutscene" from "skip decision".

### Batch actions and macros

**Use when** the player performs the same valid action many times (crafting, planting, buying, upgrading).

**Implement** Add "buy max", "craft all", "repeat last", or a hold-to-repeat input. Show the resulting cost before committing.

**Pitfall** Batch actions can hide a resource mistake. Preview the total and keep undo available for a short window.

---

## 4. Waiting and timers

### Time skip and rest

**Use when** the player is waiting for a scheduled event with nothing to do.

**Implement** Let the player rest, sleep, or fast-forward to the next event, with the world state advanced consistently. If waiting is a tension mechanic, keep the wait but shorten the dead portion.

**Pitfall** Skipping time can invalidate timed quests, shop restocks, or NPC schedules. Advance the simulation, don't just teleport the clock.

### Speed-up and fast-forward

**Use when** traversal or production is slow but the player has already made the interesting decisions.

**Implement** Offer a speed multiplier that affects animation and simulation together, with a clear indicator. Pause or drop back to normal on input or danger.

**Pitfall** Speeding up only the animation desynchronizes gameplay from feedback. Speed the simulation, or explicitly document what is cosmetic.

### Reduced downtime between attempts

**Use when** the retry loop includes unskippable menus or loading.

**Implement** Combine the death screen and retry into one screen, cache the encounter, and keep the load in the background. Show a progress indicator if the load is unavoidable.

**Pitfall** Hiding the load behind a "continue" button that does nothing for ten seconds is worse than an honest loading bar.

---

## 5. Navigation and direction

### Objective markers and map notes

**Use when** the player can forget where to go after a break.

**Implement** Mark the current objective on the map, but let the player turn markers off. Prefer a region-level hint over a precise waypoint when exploration is the point. Let players place their own markers.

**Pitfall** A precise waypoint turns exploration into follow-the-dot. Match marker precision to the intended challenge.

### Escalating hints

**Use when** a puzzle or progression gate is opaque.

**Implement** Start with a nudge ("the answer is nearby"), then a direction ("check the east wing"), then the exact solution. Escalate on request or after repeated failure, never on a timer alone. Make the final step explicit and optional.

**Pitfall** A hint that arrives before the player has had a chance to think spoils the puzzle. A hint that never becomes specific is just noise.

### Recall and recap

**Use when** the player returns after time away.

**Implement** A "previously" recap, a quest log with the last relevant line, or a command that replays recent dialogue. Store the facts the player has learned, not just the quest state.

**Pitfall** Recaps that repeat everything become unskippable exposition. Keep them short and on demand.

### Legible exits and interactables

**Use when** the player cannot distinguish scenery from gameplay.

**Implement** Highlight interactables on a held input, use consistent affordances, and show a cursor state change on hover. Make the highlight optional for players who find it noisy.

**Pitfall** Constant highlighting flattens the art and removes the satisfaction of observation. Use it as an assist, not the default.

---

## 6. Hidden state and legibility

### Expose the rule before it matters

**Use when** a mechanic has hidden counters, conditional effects, or invisible resources.

**Implement** Show the counter when it is relevant, or let the player inspect it. Use color, icons, or a tooltip to indicate "this will trigger now". Preview damage, cost, and outcome before commitment.

**Pitfall** Showing every internal number turns the game into a spreadsheet. Reveal the state that changes a decision.

### Conditional-effect indicators

**Use when** a card, ability, or item only works under a condition.

**Implement** Change the border, icon, or label when the condition is met. For deckbuilders and tactics games, highlight the eligible cards in hand.

**Pitfall** Relying on color alone excludes colorblind players. Pair color with shape or text.

### Damage and threat preview

**Use when** the player must commit before seeing the consequence.

**Implement** Show incoming damage, hit chance, or a telegraph. If uncertainty is intentional, show a range rather than a false precision.

**Pitfall** A preview that is wrong is worse than no preview. Keep it derived from the same calculation the game uses.

---

## 7. Execution and input

### Coyote time and jump buffering

**Use when** the game demands precise platforming.

**Implement** Allow a jump shortly after leaving a ledge (often ~100 ms) and remember a jump pressed shortly before landing (often ~100–150 ms). Tune to the animation, not to a number copied from another game.

**Pitfall** Too much leniency makes jumps feel floaty and can break level geometry. Too little makes the game feel unresponsive.

### Input buffering for actions

**Use when** the player presses an action slightly before it becomes available.

**Implement** Queue the action for a short window and execute it when the state allows. Clear the buffer on death, hitstun, or a conflicting action.

**Pitfall** A buffer that survives a state change can produce an action the player no longer wants.

### Mercy invincibility and knockback safety

**Use when** a single mistake can chain into unavoidable damage.

**Implement** Grant brief invulnerability after a hit, push the player away from hazards, and prevent re-hit by the same source until the window ends.

**Pitfall** Long invulnerability can trivialize hazard gauntlets. Shorten it or make it one-hit-per-source.

### Hold instead of mash

**Use when** an action requires repeated rapid input and the challenge is not finger speed.

**Implement** Accept a hold for the same duration, or offer it as an accessibility option. Keep mash for comedic or intentional tension only.

**Pitfall** Replacing mash globally can remove a designed climax. Make it optional or context-specific.

### Aim assist and lock-on

**Use when** camera or input precision is a barrier rather than a skill.

**Implement** A soft snap or magnetism with a limited cone, plus a toggle. Keep it subtle enough that manual aim still has value.

**Pitfall** Strong aim assist can make the player feel like a spectator. Let them turn it down.

---

## 8. Randomness and bad luck

### Pity timers and guaranteed drops

**Use when** a rare outcome is required for progress or builds.

**Implement** Track attempts since the last success and raise the probability, or guarantee the drop at a threshold. Reset on success. Show the guarantee if the player is expected to plan around it.

**Pitfall** Hidden pity can make lucky players feel cheated and unlucky players feel manipulated. Transparency helps.

### Pseudo-random distribution

**Use when** independent rolls produce streaks that feel unfair.

**Implement** Use a distribution that reduces repeat outcomes (for example, shuffle-bag or escalating probability). Keep the long-run rate close to the design target.

**Pitfall** Over-correcting makes outcomes predictable and removes the tension the randomness was for.

### Mulligans and opening-hand fixes

**Use when** a bad opening makes the rest of the run irrelevant.

**Implement** Let the player redraw with a cost, or automatically redraw an unplayable hand. Cap the number of redraws.

**Pitfall** A free unlimited mulligan turns deck construction into opening-hand fishing.

### Failure-weighted difficulty

**Use when** the player is stuck but the game should still be beatable.

**Implement** Lower enemy health, damage, or aggression after repeated deaths at the same encounter, with hysteresis and a cap. Restore difficulty after a success or a zone change. Tell the player if the change is significant.

**Pitfall** Rubber-banding that reacts to success punishes improvement. React to repeated failure only.

---

## 9. Resources and irreversible choices

### Confirmation and undo

**Use when** an action destroys value or cannot be reversed.

**Implement** Confirm destructive actions, or provide a short undo window (for example, 10–30 seconds) with a visible timer. For bulk actions, preview the total.

**Pitfall** Confirmation fatigue: if everything asks, players stop reading. Reserve it for irreversible, high-cost actions.

### Refunds and respecs

**Use when** a build or upgrade choice can strand the player.

**Implement** Allow a full or partial refund within a window, or a respec at a cost. Show what will be lost before committing. Keep limited-use or story-granted items exempt.

**Pitfall** Free instant respecs can erase the weight of a build decision. Charge something, even if it is only time.

### Locks and favorites

**Use when** the player can accidentally sell, dismantle, or overwrite something valuable.

**Implement** Let players lock items, loadouts, or substats. Locked items are excluded from bulk operations. Make the lock state visible.

**Pitfall** A lock that is buried in a submenu will not prevent the mistake it was meant to prevent.

### No-waste rules

**Use when** a resource is consumed by an action that may fail or become irrelevant.

**Implement** Refund on failure, preserve unused inputs, or convert overflow into a lesser resource. For capacity limits, send the overflow to a mailbox or stash rather than deleting it.

**Pitfall** Overflow systems can become an infinite storage exploit. Cap them or make retrieval inconvenient.

### Retroactive credit

**Use when** a quest asks for something the player already has or did.

**Implement** Check existing inventory and flags when accepting a quest, and complete it immediately if the condition is already met. Do the same for achievements and collections.

**Pitfall** Retroactive completion can skip a tutorial the player still needs. Separate "already has the item" from "already knows the mechanic".

---

## 10. Softlocks and dead ends

### Escape hatch

**Use when** the player can enter a state with no valid path forward.

**Implement** Provide a always-available "return to last safe point" or "reset area" action. Preserve collected items and solved puzzles where possible. Log the state that triggered the escape so you can fix the root cause.

**Pitfall** An escape hatch that is too easy to reach can become a fast-travel exploit. Put it behind a deliberate input or menu.

### Unwinnable-state detection

**Use when** the design allows resource exhaustion or one-way gates.

**Implement** Define the minimum resources required to finish the current area. Check on key transitions. If the player cannot meet the requirement, grant a fallback resource, open an alternate route, or offer a reset with a clear explanation.

**Pitfall** False positives are worse than the softlock: a warning that fires when the player is fine erodes trust. Validate the condition against the actual solvability model.

### Alternate routes and graceful failure

**Use when** a puzzle or choice can be failed permanently.

**Implement** Provide a slower or less rewarding path forward after a failure, rather than a hard stop. The player loses the perfect outcome, not the ability to continue.

**Pitfall** An alternate route that is clearly worse can feel like a punishment for a mistake the player could not foresee. Telegraph the failure before it becomes permanent.

### Save-state validation

**Use when** saves can be corrupted or restored into an invalid state.

**Implement** Validate on load, keep a backup slot, and migrate or repair known-invalid states. If repair is impossible, offer the nearest checkpoint rather than a crash.

**Pitfall** Silent repair can erase progress the player expected to keep. Tell them what changed.

---

## 11. Difficulty and onboarding

### Assist mode and granular difficulty

**Use when** a single difficulty slider cannot address the player's actual barrier.

**Implement** Separate damage taken, damage dealt, reaction windows, puzzle timers, and navigation assistance. Let the player change them mid-game. Default to the intended experience and make the options discoverable.

**Pitfall** A wall of sliders is its own barrier. Group options by player goal ("I want more time", "I want to take less damage") rather than by internal variable.

### Dynamic difficulty adjustment

**Use when** the player is repeatedly failing a specific encounter.

**Implement** Track failures per encounter, adjust a bounded set of parameters, and use hysteresis so the difficulty does not oscillate. Reset after a success or a zone change. Prefer invisible adjustments for pacing and visible options for agency.

**Pitfall** Adjusting too quickly makes the game feel like it is playing itself. Adjust slowly and cap the range.

### New-player protection

**Use when** the opening hours can end a run before the player understands the rules.

**Implement** Reduce or remove penalties in the tutorial zone, guarantee early resources, and delay lethal mechanics until they have been introduced safely. Make the protection expire explicitly.

**Pitfall** Protection that persists too long creates a difficulty cliff at the boundary. Ramp it down rather than switching it off.

### Practice and replayable challenges

**Use when** the player needs to learn a boss or mechanic without the full cost of the run.

**Implement** Unlock a practice mode after encountering the content, with configurable phase and resources. Keep rewards out of practice mode, or cap them.

**Pitfall** Practice that grants full rewards trivializes the main loop. Separate learning from earning.

### Telegraphed tutorials

**Use when** a mechanic is introduced under pressure.

**Implement** Introduce the mechanic in a safe context, then test it in a low-stakes context, then combine it. Allow the tutorial to be skipped on replay.

**Pitfall** A tutorial that cannot be skipped is a frustration feature, not an anti-frustration feature.

---

## 12. Accessibility and perception

Accessibility features are anti-frustration features for the input and perception channel. They often improve the game for everyone.

### Remapping and input alternatives

**Use when** the default control scheme excludes a player or a device.

**Implement** Full remapping, multiple bindings, toggle vs hold, and support for common adaptive controllers. Show the current binding in prompts.

**Pitfall** Remapping that breaks prompt text or quick-time events creates a new barrier. Drive prompts from the binding table.

### Visual accessibility

**Use when** text, color, contrast, or motion carries information.

**Implement** Text size and contrast options, colorblind-safe palettes with shape/text redundancy, motion reduction, subtitle sizing and background, and a high-contrast mode.

**Pitfall** A colorblind filter applied to the whole screen can distort art without fixing the underlying reliance on color. Fix the information design first.

### Audio accessibility

**Use when** sound carries a cue the player needs.

**Implement** Visual indicators for important sounds, subtitles with speaker labels, and separate volume channels. Provide a mono option.

**Pitfall** Visual cue spam can overwhelm the screen. Prioritize cues by gameplay importance.

### Timing and motor accessibility

**Use when** the challenge includes reaction speed or sustained input.

**Implement** Adjustable game speed, hold-to-mash alternatives, aim assist, autosave and generous checkpoints, and the ability to pause during cutscenes. For QTEs, allow a longer window or a skip.

**Pitfall** Slowing the whole game can break physics or animation. Test the simulation at the supported speed range.

---

## 13. Cognitive load and interfaces

### Fewer steps to the action

**Use when** the player repeats a menu path.

**Implement** Add shortcuts, remember the last tab, allow loadout presets, and put the common action on the first screen. Reduce confirmation dialogs for reversible actions.

**Pitfall** Shortcuts that are undocumented are invisible. Surface them in the UI or the controls screen.

### Search, sort, and filter

**Use when** the inventory or map grows beyond a screen.

**Implement** Text search, sort by recent or relevant, filters that persist, and a "new" marker. For maps, filter by icon type.

**Pitfall** Filters that reset on every open defeat the purpose. Persist them per save.

### Comparison and preview

**Use when** the player must choose between statistically similar options.

**Implement** Show deltas against the currently equipped item, with clear positives and negatives. Preview the result of a craft or upgrade before committing.

**Pitfall** Delta displays that ignore set bonuses or conditional effects mislead the player. Use the same calculation as the game.

### Contextual defaults

**Use when** the player must repeat a known-good choice.

**Implement** Pre-fill the last used option, remember party composition, and suggest a valid loadout. Never make the default destructive.

**Pitfall** A default that is wrong for the current context can cause a misclick. Show what will happen.

---

## 14. Multiplayer and other players

### Anti-trolling and anti-griefing

**Use when** other players can block progress or waste time.

**Implement** Vote-kick with safeguards, spawn protection, loot rights or need/greed rules, and the ability to mute or block. For cooperative games, avoid mechanics that let one player hold the group hostage.

**Pitfall** Overly aggressive automation can punish legitimate play. Give players a way to appeal or regroup.

### Disconnect and AFK handling

**Use when** a session can stall on one player.

**Implement** Shorten turns for inactive players, allow the group to continue, and let a disconnected player reconnect to their state. For competitive games, grant a safe concession or pause rather than a loss.

**Pitfall** Penalizing a genuine disconnect the same as a rage-quit trains players to avoid the game. Distinguish the two where possible.

### Matchmaking and skill protection

**Use when** new or low-skill players are matched against experts.

**Implement** Separate newcomer pools, placement matches, and a bounded skill range. For team games, avoid placing a single weak player with a much stronger opponent.

**Pitfall** Overly strict skill matching increases queue times and smurfing. Combine with other protections.

### Shared progress and catch-up

**Use when** players join a session at different progression levels.

**Implement** Level sync, borrowed gear, or a catch-up buff. Make the synced player still useful, not just alive.

**Pitfall** Level sync that removes all build expression can make the session feel pointless. Sync power, preserve identity.

---

## 15. Cross-cutting implementation notes

### State ownership

Decide early whether each assist lives in save data, run state, scene state, or UI state. A checkpoint that lives only in the scene disappears on reload; a hint counter that lives in the UI resets when the menu closes. Most anti-frustration bugs are ownership bugs.

### Determinism

Features that change simulation (difficulty, RNG, time skip) should be deterministic given the same inputs, or explicitly excluded from replays and tests. Keep a seed and a feature-state version so a recorded run can be reproduced.

### Persistence

Assume the player will quit at the worst moment. Write saves atomically, keep a backup, version the schema, and test migration from the previous release. Anti-frustration features that lose progress are worse than the frustration they replaced.

### Feature flags

Put each assist behind a flag or settings entry. This lets you disable it for challenge modes, tests, and debugging, and lets you ship a conservative default while you tune.

### Telemetry

Record the events that let you tune the feature without guessing:

- attempts per encounter and time between attempts
- deaths by cause and location
- quit points and session length
- hint usage and escalation level
- assistance activation and subsequent success rate
- resource waste and refund usage

Aggregate by player skill band, not just globally, or the feature will be tuned for the average player and fail the players who need it.

### Testing

Write scenario tests for the moments, not just the functions:

- first attempt, repeated attempt, and attempt after a long break
- save and quit mid-feature, then reload
- feature at its cap and at zero
- feature combined with another assist
- feature disabled
- deliberate exploit attempt

### Documentation

Tell the player what the feature does and how to turn it off. Hidden assistance is sometimes appropriate for pacing, but hidden *rules* are not. A short tooltip or settings description prevents the feature from feeling like a bug.
