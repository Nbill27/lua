# Coding Rules (Mandatory)

## Required

- Use `local` for all variables and functions.
- Use `task.wait()` for yield. Use `task.spawn()` for parallel work.
- Wrap risky calls in `pcall` (service fetch, player lookup, remote trigger, JSON decode, UI build).
- Use `RunService.RenderStepped` or `RunService.Heartbeat` for visual loops.
- Support save/load config (JSON encode/decode to file).
- Support PC and mobile. Check `UserInputService.TouchEnabled` for touch layout.

## Forbidden

- Do not use `while true do wait() end` for visuals. Use RenderStepped/Heartbeat.
- Do not create globals without stated reason.
- Do not claim absolute safety or perfection.

## Standard Script Structure

1. Load: services, player, character guards.
2. Settings: default config table.
3. Utility: helpers (pcall wrappers, find player, clamp).
4. UI: Wild UI window, tabs, controls, touch sizing.
5. Main Loop: RenderStepped/Heartbeat with interval control.
6. Save/Load: write config JSON, load on start.

## Style

- Modular functions, early return on nil.
- Validate `Player`, `Character`, `Humanoid` before use.
- Limit rendered objects per frame.
