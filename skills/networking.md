---
name: networking
description: Use when script must read or trigger RemoteEvent/RemoteFunction from client. Handoff to scripting for loop wiring, to performance for call frequency.
---

# Networking (Client Side)

## Concepts

- `RemoteEvent`: one-way. Client calls `:FireServer(args)`.
- `RemoteFunction`: request-response. Client calls `:InvokeServer(args)`.
- Direction here is client to server only. No server logic in this kit.

## Find Remote

```luau
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local events = ReplicatedStorage:WaitForChild("Events", 5)
local remote = events and events:WaitForChild("Action", 5)
if not remote then warn("remote missing") return end
```

Always timeout WaitForChild. Guard nil. Re-resolve after respawn if needed.

## Validate Args

```luau
local function validAction(name)
    if type(name) ~= "string" then return false end
    if #name == 0 or #name > 64 then return false end
    return true
end
```

Never send oversized tables or mixed types. Keep args small and typed.

## Trigger Safely (Event)

```luau
local lastCall = 0
local COOLDOWN = 0.5
local function fireAction(name)
    if not validAction(name) then return end
    local now = os.clock()
    if now - lastCall < COOLDOWN then return end
    lastCall = now
    local ok, err = pcall(function()
        remote:FireServer(name)
    end)
    if not ok then warn(err) end
end
```

Do not call every frame. Cooldown 0.5s default. Increase for heavy actions.

## Trigger Safely (Function)

```luau
local function callAction(name)
    if not validAction(name) then return nil end
    local ok, result = pcall(function()
        return remote:InvokeServer(name)
    end)
    if not ok then warn(result) return nil end
    return result
end
```

InvokeServer can yield. Never call inside RenderStepped. Use task.spawn when unsure.

## Read Remote Events

Client can listen if remote fires to client:

```luau
local conn
conn = remote.OnClientEvent:Connect(function(...)
    local ok, err = pcall(function(...)
        print(...)
    end, ...)
    if not ok then warn(err) end
end)
-- cleanup: conn:Disconnect()
```

Disconnect on script disable. Guard callback with pcall.

## Limits To Know

- Server may ignore or reject malformed args.
- Do not spam. Add cooldown (e.g. 0.5s between calls).
- Validate remote exists before each session.
- Never assume call succeeds. Always handle nil/timeout.
- Keep frequency low. Handoff to `performance` for batching strategy.
