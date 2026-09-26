# lua-toolkit

Agent kit for writing client-side Lua/Luau scripts with AI help. Three roles (`lead`, `writer`, `checker`) share five knowledge files (`skills/`) so output stays consistent: Wild UI by default, PC + mobile layout, `pcall` on risky calls, JSON config save/load.

## Who Is This For

- You write gameplay helper scripts and want structured AI output instead of random snippets.
- You use an agent framework (OpenCode or similar) that can load `config.json` with per-agent instructions.

## Prerequisites

- Agent framework that supports `config.json` with global + per-agent instructions.
- Basic Luau knowledge (variables, functions, tables). See `skills/basics.md` for a refresher.

## Structure

```text
lua-toolkit/
├── config.json        # Main config: global rules + 3 agents + skill wiring
├── README.md          # This file
├── rules.md           # Mandatory coding rules (local, task.wait, pcall, no placeholders)
├── routing.md         # Which role must read which skill, in what order
├── skills/            # Knowledge base, auto-loaded per role
│   ├── basics.md      # Luau syntax, tables, pcall, common mistakes
│   ├── scripting.md   # Full 6-stage script template + ESP + Wild UI + mobile
│   ├── performance.md # Interval throttle, cache, 20-box render budget
│   ├── api-lookup.md  # Service / property / event dictionary
│   └── networking.md  # RemoteEvent/RemoteFunction from client, cooldown 0.5s
└── roles/             # Agent prompts
    ├── lead.md        # Orchestrator: clarify, delegate, retry max 2x, report
    ├── writer.md      # Builds the script (defaults + self-check)
    └── checker.md     # Reviews: PASS / REVISE / REJECT + score 1-10
```

Total: 12 files, 2 subfolders. No hidden folders. Flat by design.

## How It Works

```text
You → lead → writer → checker → lead → you
```

1. You send a request to `lead` (example below).
2. `lead` reads `routing.md`, then `skills/scripting.md` + `skills/api-lookup.md`. Asks back if the request is vague.
3. `lead` forwards a build contract to `writer` (goal, feature list, constraints).
4. `writer` reads all 5 skills, builds a complete runnable script using the 6-stage template: Load → Settings → Utility → UI → Main Loop → Save/Load.
5. `checker` reviews against `rules.md` + `scripting.md` + `performance.md`. Returns verdict PASS / REVISE / REJECT with score and line-specific fixes.
6. Score 6-7 or REVISE: one retry with the issue list (max 2 rounds). Score ≤ 5 twice: `lead` stops and reports the blocker instead of looping.
7. `lead` delivers: summary, full script, usage steps, limits, checker score.

Skill files are never guessed. If a listed skill is missing, the agent must stop and ask you.

## Installation

### Option A: OpenCode / compatible framework

1. Copy the `lua-toolkit/` folder to your project root.
2. Point the framework at `D:\lua-toolkit\config.json` (or your copy path).
3. Confirm 3 agents register: `lead` (primary), `writer` + `checker` (subagents).

### Option B: Manual (any AI chat)

1. Paste `rules.md` + `routing.md` into the chat first.
2. Paste the role file you need (`roles/writer.md` for building, `roles/checker.md` for review).
3. Paste the skill files listed in that role's "Required Skills to Read" section.

## Usage

Send a concrete request to `lead`:

```text
Build a Wild UI tool with ESP toggle (max 20 boxes, team check on),
interval 0.1s, PC + mobile layout, config saved to lua-toolkit-config.json.
```

What you get back:

1. **Summary** — features built + assumptions (e.g. Wild UI URL placeholder replaced with yours).
2. **Full script** — single runnable block, no TODOs.
3. **Usage** — controls, config file name, mobile notes.
4. **Performance notes** — interval value, render count, cache used.
5. **Limits** — what the script does not handle + checker score/verdict.

Vague requests trigger one clarification round first (behavior, scope, UI), then the build starts.

## Roles and Skills Map

| Role | Must read | Why |
|------|-----------|-----|
| `lead` | `routing.md`, `scripting.md`, `api-lookup.md` | Scope the job correctly before delegating |
| `writer` | `routing.md` + all 5 skills | Full context: syntax, template, speed, API, remotes |
| `checker` | `routing.md`, `rules.md`, `scripting.md`, `performance.md` | Verify structure, error handling, frame budget |

Handoff order: `basics` → `scripting` → `performance` → `api-lookup` → `networking`.

## Defaults You Should Know

| Setting | Default | Change where |
|---------|---------|--------------|
| UI library | Wild UI | `skills/scripting.md`, writer request |
| Platform | PC + mobile (`TouchEnabled` check) | Request text |
| Main loop | `Heartbeat` + accumulator, `Interval = 0.1` | `Config.Interval` |
| ESP cap | `MaxBoxes = 20`, one Drawing per target | `Config.MaxBoxes` |
| Config file | `lua-toolkit-config.json` (validated merge) | `CONFIG_FILE` |
| Remote cooldown | `0.5s` between calls | `COOLDOWN` in `skills/networking.md` |
| Error handling | `pcall` on service, file, JSON, UI, remote | `rules.md` |

## Customizing

- **Different UI library:** replace the Wild UI loader in `skills/scripting.md` and the writer defaults in `roles/writer.md`.
- **Different cooldown or cap:** edit the numbers in `skills/networking.md` / `skills/performance.md`, then mirror them in `roles/writer.md` so agents stay consistent.
- **New skill:** add the `.md` file under `skills/`, list it in `routing.md` + `config.json` instructions, and mention it in the relevant role prompt.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Agent invents API names | Skipped `api-lookup.md` | Remind it to read `routing.md` first |
| Script has TODOs | Writer skipped self-check | Ask writer to run its handoff checklist |
| Score without verdict | Old checker prompt | Pull latest `roles/checker.md` (verdict is line 1) |
| Endless revise loop | Retry cap ignored | Lead caps at 2 rounds, then reports blocker |
| Mobile layout broken | Touch sizing missing | Check `btnSize` / `fontSize` block in `skills/scripting.md` |
