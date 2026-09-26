---
name: basics
description: Use for Luau syntax, tables, and functions. Handoff to scripting for runtime structure, to performance for hot-path tuning.
---

# Basics

## Variables

```luau
local speed = 16
local name = "Player1"
local enabled = true
local nothing = nil
```

Always use `local`. Check `nil` before index.
Type hints are optional but help readability:

```luau
local count: number = 0
local label: string = "ESP"
local active: boolean = true
```

## Conditions

```luau
if player and player.Character then
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speed
    end
end
```

Keep nesting max 3 levels. Early return preferred:

```luau
local char = player and player.Character
if not char then return end
local humanoid = char:FindFirstChildOfClass("Humanoid")
if not humanoid then return end
humanoid.WalkSpeed = speed
```

## Loops

```luau
for i, v in ipairs(items) do
    print(i, v)
end
for k, v in pairs(config) do
    print(k, v)
end
local i = 0
while i < 3 do
    i += 1
    if i == 2 then continue end
    print(i)
end
```

Use `ipairs` for arrays, `pairs` for maps. Use `continue` to skip.

## Functions

```luau
local function getHumanoid(targetPlayer)
    local char = targetPlayer and targetPlayer.Character
    if not char then return nil end
    return char:FindFirstChildOfClass("Humanoid")
end
local ok, hum = pcall(getHumanoid, player)
if not ok then warn(hum) return end
```

One function, one job. Return `nil` on missing input.

## Tables

```luau
local config = { ESP = true, Interval = 0.1 }
config.ESP = false
local list = {}
table.insert(list, "a")
table.remove(list, 1)
local joined = table.concat({ "a", "b", "c" }, ",")
```

Use tables for config and cache. Reuse tables in loops.
Never build strings with `..` inside a hot loop. Use `table.concat`.

## Type Guards

```luau
local function isValidConfig(data)
    if type(data) ~= "table" then return false end
    if type(data.ESP) ~= "boolean" then return false end
    if type(data.Interval) ~= "number" then return false end
    return true
end
```

Validate external input: config files, remote args, UI callbacks.

## Luau Idioms

- `+=`, `-=`, `*=`, `/=` supported.
- `continue` supported in loops.
- Type hints optional: `local x: number = 1`.
- `task.wait(0.1)` replaces `wait(0.1)`. Never use bare `wait()`.
- `task.spawn(fn)` replaces `spawn(fn)`.
- `task.delay(1, fn)` for deferred call.
- `table.freeze(config)` to lock defaults after load.

## pcall Pattern

```luau
local ok, result = pcall(function()
    return game:GetService("Players").LocalPlayer
end)
if not ok then warn(result) return end
```

Wrap: service fetch, file read, JSON decode, UI build, remote calls.

## Common Mistakes

1. Global without `local`. Pollutes scope. Fix: add `local`.
2. Deep nested if. Hard to read. Fix: early return.
3. No nil check on `Character`, `Humanoid`, `FindFirstChild` result. Fix: guard clause.
4. `while true do wait() end` for visuals. Fix: RenderStepped/Heartbeat.
5. String concat in loop. Fix: table.concat or cache.
6. Trusting file content without `isValidConfig`. Fix: validate then merge.
7. Calling `FindFirstChild` every frame. Fix: cache handle, refresh on respawn.
