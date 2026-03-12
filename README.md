# autoresearch-claude-code

Autonomous experiment loop for Claude Code. Port of [pi-autoresearch](https://github.com/davebcn87/pi-autoresearch) as a pure skill (no MCP server).

Continuously optimizes any measurable target (test speed, bundle size, training loss, etc.) by running experiments, measuring results, keeping winners, discarding losers, and looping forever until interrupted.

## Install

```bash
git clone <this-repo> ~/autoresearch-claude-code
cd ~/autoresearch-claude-code
./install.sh
```

Then add the hook to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {"type": "command", "command": "~/.claude/hooks/autoresearch-context.sh"}
        ]
      }
    ]
  }
}
```

## Usage

In Claude Code:

```
/autoresearch optimize test suite runtime
```

This will:
1. Ask/infer: goal, command, metric, files in scope, constraints
2. Create a git branch `autoresearch/<goal>-<date>`
3. Write `autoresearch.md` (session doc) and `autoresearch.sh` (benchmark script)
4. Run a baseline, then loop forever optimizing

### Commands

- `/autoresearch <goal>` — Start a new experiment loop
- `/autoresearch` — Resume an existing loop (if `autoresearch.md` exists)
- `/autoresearch off` — Pause autoresearch mode

### Steering

Send a message while an experiment is running to steer the next experiment. The agent will finish the current experiment, log it, then incorporate your idea.

### Ideas backlog

The agent writes promising but complex ideas to `autoresearch.ideas.md`. On resume, it reads this file for inspiration.

## How it works

The original pi-autoresearch used 3 registered MCP tools. This port encodes all that logic as skill instructions that tell Claude to use its built-in Bash/Read/Write tools:

| Original | Claude Code |
|---|---|
| `init_experiment` tool | Agent writes config header to `autoresearch.jsonl` |
| `run_experiment` tool | Agent runs `bash -c "./autoresearch.sh"` with timing |
| `log_experiment` tool | Agent appends result JSON, runs `git commit` if keep |
| TUI dashboard | `autoresearch-dashboard.md` file |
| `before_agent_start` hook | `UserPromptSubmit` hook injects context |
| Steer queueing | Skill instructions |

### State format

All state lives in `autoresearch.jsonl`:

```jsonl
{"type":"config","name":"optimize tests","metricName":"duration_s","metricUnit":"s","bestDirection":"lower"}
{"run":1,"commit":"abc1234","metric":42.3,"metrics":{},"status":"keep","description":"baseline","timestamp":1234567890,"segment":0}
{"run":2,"commit":"def5678","metric":39.1,"metrics":{},"status":"keep","description":"parallelize setup","timestamp":1234567891,"segment":0}
```

## Uninstall

```bash
cd ~/autoresearch-claude-code
./uninstall.sh
```

Then remove the hook entry from `~/.claude/settings.json`.
