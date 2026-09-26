---
name: performance
description: Use when loop is slow, frame drops, or many objects render. Handoff to scripting for loop placement, to basics for table reuse syntax.
---

# Performance

## Hot Path Rules

- Do less per frame. Early exit fast.
- Cache `FindFirstChild`, `GetService`, `Character` results.
- Reuse tables. Do not allocate in Heartbeat.
- No string concat in loop. Use cached strings or `table.concat`.

```luau
local parts = {}
for _, p in ipairs(players) do
    parts[#parts + 1] = p.Name
end
local text = table.concat(parts, ",")
```

## Interval Control

```luau
local acc = 0
local INTERVAL = 0.1
RunService.Heartbeat:Connect(function(dt)
    acc += dt
    if acc < INTERVAL then return end
    acc = 0
    -- work
end)
```

Budget guide:
- Heavy scan (player list, descendants): 0.25-0.5s.
- ESP position update: 0.05-0.1s.
- Smooth visuals only: every render, no throttle.

## Cache Pattern

```luau
local cache = {}
local function getHumanoid(char)
    if cache[char] then return cache[char] end
    local h = char:FindFirstChildOfClass("Humanoid")
    cache[char] = h
    return h
end
Players.PlayerRemoving:Connect(function()
    table.clear(cache)
end)
```

Clear cache when player leaves or character respawns.

## Render Budget

- Max ~20 ESP boxes active.
- Hide off-screen, do not destroy/recreate.
- One Drawing object per target, reused.
- Prefer `Heartbeat` for logic, `RenderStepped` only for smooth visuals.
- Batch updates: process N targets per tick when list is long.

```luau
local cursor = 1
local function processBatch(list, perTick)
    for _ = 1, perTick do
        local item = list[cursor]
        if not item then cursor = 1 return end
        -- update item
        cursor += 1
        if cursor > #list then cursor = 1 break end
    end
end
```

## Heartbeat vs RenderStepped

| Use | Choice | Reason |
|-----|--------|--------|
| Logic, scan, config | Heartbeat | Stable rate, no render block |
| Smooth box follow | RenderStepped | Sync with frame |
| Heavy work | Heartbeat + interval | Protect frame time |

## Checklist

- [ ] No object creation inside loop
- [ ] Interval set, accumulator used
- [ ] Cached lookups
- [ ] Render count capped
- [ ] Mobile frame rate acceptable
