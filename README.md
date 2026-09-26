# lua-toolkit

Simple agent kit for Lua/Luau client-side scripting. PC + mobile. Default UI: Wild UI.

## Structure

lua-toolkit/
├── config.json
├── README.md
├── rules.md
├── routing.md
├── skills/
│   ├── basics.md
│   ├── scripting.md
│   ├── performance.md
│   ├── api-lookup.md
│   └── networking.md
└── roles/
    ├── lead.md
    ├── writer.md
    └── checker.md

## Agents

| Role | Mode | Job |
|------|------|-----|
| lead | primary | Receive request, read routing, delegate, integrate, report |
| writer | subagent | Write script per spec |
| checker | subagent | Review script, score 1-10 |

## Skills

| Skill | When to use |
|-------|-------------|
| basics | Luau syntax, tables, functions |
| scripting | Client runtime structure, UI, ESP with Drawing, config, mobile |
| performance | Hot path, cache, interval control |
| api-lookup | Roblox API reference lookup |
| networking | RemoteEvent/RemoteFunction from client side |

## Routing

1. Read `routing.md` first.
2. Read listed skills for your role.
3. If skill file missing, ask user. Do not invent.
4. Global rules in `rules.md` apply to all output.

## Usage

1. Copy `lua-toolkit/` to project root.
2. Load `config.json` in agent framework.
3. Call `lead` with user request.
4. `lead` delegates to `writer`, then `checker`, then reports.
