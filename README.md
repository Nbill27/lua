# lua-toolkit

Agent kit for writing client-side Lua/Luau scripts with AI help. Three roles (`lead`, `writer`, `checker`) share five knowledge files (`skills/`) so output stays consistent: Wild UI by default, PC + mobile layout, `pcall` on risky calls, JSON config save/load.

## Who Is This For

- You write gameplay helper scripts and want structured AI output instead of random snippets.
- You use an agent framework (OpenCode or similar) that can load `config.json` with per-agent instructions.

## Prerequisites

- Git (to clone this repo).
- Agent framework that supports `config.json` with global + per-agent instructions (OpenCode recommended).
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

## Installation (Step by Step)

### Option A: OpenCode (recommended, auto-load skills)

1. Clone the repo:
   ```powershell
   git clone https://github.com/Nbill27/lua.git lua-toolkit
   Set-Location lua-toolkit
   ```
2. Open the folder in OpenCode (or point your framework at it).
3. Confirm the framework loads `config.json`. It must register:
   - `lead` as primary agent with instructions `routing.md`, `skills/scripting.md`, `skills/api-lookup.md`.
   - `writer` as subagent with all 5 skills.
   - `checker` as subagent with `rules.md`, `scripting.md`, `performance.md`.
   - Global instructions `rules.md` for every agent.
4. Verify the wiring with a smoke test. Ask `lead`:
   ```text
   List the skills you must read for your role and the order.
   ```
   Correct answer mentions `routing.md` first, then `scripting.md` + `api-lookup.md`. If it invents other files, the config did not load — re-check the `config.json` path.
5. Done. Go to Usage below and send your first build request to `lead`.

### Option B: Manual (any AI chat, no framework)

1. Clone or download the repo so you have all 12 files locally.
2. Start a new chat. Paste these two files first, in order:
   - `rules.md` (global coding rules).
   - `routing.md` (role-to-skill map).
3. Decide which role you need:
   - Building something → paste `roles/writer.md`, then paste every skill in its "Required Skills to Read" section.
   - Reviewing a script → paste `roles/checker.md`, then paste `rules.md` + `skills/scripting.md` + `skills/performance.md`.
   - End-to-end (build + review) → do the writer round first, then open the checker round with the writer output.
4. Send your request (see Usage example). If the model says a skill is missing, paste that file — never let it guess the content.

## Roles Explained

### `lead` — The Orchestrator (primary agent)

- **Task:** understand your request, split it into build steps, delegate to `writer`, send the result to `checker`, enforce max 2 retry rounds, deliver the final package.
- **Reads:** `routing.md` first, then `skills/scripting.md` + `skills/api-lookup.md`.
- **Writes:** no code. Only plans, contracts, and the final report.
- **Input from you:** goal + feature list + constraints (example: "ESP toggle, 20 boxes, mobile layout").
- **Output to you:** summary, full script, usage steps, limits, checker score/verdict.
- **Use when:** every new task starts here. Never skip `lead` for multi-step work.
- **Stops when:** request is vague (asks 1 clarification round), checker rejects twice (reports blocker instead of looping forever).

### `writer` — The Builder (subagent)

- **Task:** produce one complete runnable script from the `lead` contract.
- **Reads:** `routing.md` + all 5 skills (`basics`, `scripting`, `performance`, `api-lookup`, `networking`).
- **Follows:** 6-stage template (Load → Settings → Utility → UI → Main Loop → Save/Load), defaults table below, self-check checklist before handoff.
- **Input:** build contract from `lead` (goal, features, constraints).
- **Output:** feature summary, full script block (no TODOs, no fake URLs), usage, performance notes (interval, render count), known limits.
- **Use when:** `lead` delegates a build, or you manually need fresh code.
- **Never:** claims perfection, writes destructive code, ignores mobile, leaves placeholders.

### `checker` — The Reviewer (subagent)

- **Task:** verify the writer output against `rules.md` + `scripting.md` + `performance.md`.
- **Reads:** `routing.md`, `rules.md`, `skills/scripting.md`, `skills/performance.md`.
- **Checks:** quality (locals, structure, no placeholders), error handling (`pcall`, nil guards, JSON validation), performance (interval, render cap, cache, mobile), safety (no wipe, no spam, cooldown on remotes).
- **Input:** full writer script + original goal.
- **Output (in order):** verdict `PASS` / `REVISE` / `REJECT` on line 1, score 1-10 with reason, issue list with file/line + severity, ready-to-paste fix per blocker/major, retry note for next round.
- **Use when:** every writer draft must pass here before reaching you.
- **Score guide:** 9-10 pass, 7-8 small revise, 5-6 structural revise, 1-4 reject.

## Skills Explained

All skill files are in English. Each has frontmatter (`name`, `description`) stating when to use it and where to hand off next.

### `skills/basics.md` — Luau language refresher

- **Contains:** `local` variables, type hints, conditions with early return, `ipairs`/`pairs`/`while` loops, functions, tables, `table.concat` vs `..` concat, type-guard validators, `task.wait`/`task.spawn`/`task.delay` idioms, `pcall` wrapper pattern, 7 common mistakes with fixes.
- **Use when:** writing or reviewing any Luau code, fixing nil-index crashes, cleaning globals or deep nesting.
- **Hands off to:** `scripting` for runtime wiring, `performance` for hot-path tuning.

### `skills/scripting.md` — Client runtime template (the core skill)

- **Contains:** service bootstrap, player/character/humanoid guards, respawn reconnect, full 6-stage runnable template (Load → Settings → Utility → Wild UI → Heartbeat loop → JSON save/load with type-validated merge), Wild UI toggle binding, ESP pattern (Drawing cache, team check, `WorldToViewportPoint`, 20-box cap, cleanup on leave), touch sizing (`120x48` touch vs `100x32` PC), debugging ladder.
- **Use when:** building or reviewing any script in this kit. `lead` and `writer` and `checker` all read it.
- **Hands off to:** `api-lookup` for exact property/event names, `performance` for frame budget, `networking` for remotes.

### `skills/performance.md` — Frame-budget guard

- **Contains:** hot-path rules (early exit, cache, no alloc in Heartbeat), interval accumulator template with budget table (heavy scan 0.25-0.5s, ESP 0.05-0.1s), humanoid cache with clear-on-leave, 20-box render budget, N-per-tick batch processor, Heartbeat-vs-RenderStepped decision table, 5-box checklist.
- **Use when:** loop feels slow, frame drops, ESP list grows, or `checker` flags uncapped rendering.
- **Hands off to:** `scripting` for loop placement, `basics` for table-reuse syntax.

### `skills/api-lookup.md` — Property and event dictionary

- **Contains:** core services list, player/character/humanoid fields, respawn guard snippet, UserInputService props + events with touch example, RunService events, camera projection (`WorldToViewportPoint`), Vector3/CFrame/Vector2 notes, Drawing API (types, props, cleanup), HttpService JSON + file-func guards, Enum and UDim2 quick refs.
- **Use when:** you need an exact name (property, event, enum) instead of guessing.
- **Hands off to:** `scripting` for wiring, `networking` for remote objects. Server-side stores omitted by design.

### `skills/networking.md` — Client-side remote guide

- **Contains:** RemoteEvent vs RemoteFunction, `WaitForChild` with timeout, arg validator (string, length cap), `FireServer` with 0.5s cooldown, `InvokeServer` with `pcall` (never in RenderStepped), `OnClientEvent` listener with disconnect, limits list (server may reject, validate before session, handle nil/timeout).
- **Use when:** the script must read or trigger a remote from the client side.
- **Hands off to:** `scripting` for loop wiring, `performance` for call batching.

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
