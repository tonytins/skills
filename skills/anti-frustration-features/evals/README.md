# Trigger evals

`trigger-evals.json` contains 20 realistic prompts: 10 that should load this skill, and 10 near-miss negatives that should not. It is formatted for the skill-creator's `run_loop.py`:

```bash
cd /path/to/skill-creator
python3 -m scripts.run_loop \
  --eval-set /path/to/anti-frustration-features/evals/trigger-evals.json \
  --skill-path /path/to/anti-frustration-features \
  --model <model-id> \
  --runs-per-query 3 \
  --max-iterations 5 \
  --results-dir ./trigger-results
```

## What the set covers

The positives are deliberately multi-issue, because a one-line "should I add coyote time?" is the kind of question a model may answer from general knowledge even when a skill would help. They cover:

- playtest churn and "the game is exhausting"
- long runbacks, checkpoints, and retry state
- duplicate rares and pity timers
- softlocks and escape hatches
- accessibility options
- dynamic difficulty and rubber-banding
- co-op griefing and AFK handling
- inventory tedium and accidental sales
- escalating hints

The negatives are adjacent but out of scope: save serialization, a health-bar bug, making a boss harder, general puzzle design, intentional horror scarcity, achievements, economy rebalancing, rendering performance, a deliberate rage game, and a dialogue-tree implementation.

## Status

The description has been checked statically: the frontmatter parses as YAML, the skill-creator parser reads the full description, and OpenCode advertises the skill from `~/.agents/skills/anti-frustration-features`.

The trigger rate has **not** been measured yet. The first OpenCode runs were invalid because the skill was only in the repository's `skills/` directory, which is not one of OpenCode's discovery sources; the model sometimes found the file by exploring the repo, but that is not a trigger. After installing the skill into `~/.agents/skills`, the free model provider hit its usage limit (`provider.free-limit`), and the Claude CLI was also rate-limited. Rerun the loop when quota is available.

When running through OpenCode instead of `claude`, check the JSON event stream for a `skill` tool call whose input contains `anti-frustration-features`, or a `read` of the skill's `SKILL.md`. Do not count a final answer that merely mentions the skill name.
