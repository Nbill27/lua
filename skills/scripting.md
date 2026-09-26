---
name: scripting
description: Use for client-side runtime structure, UI wiring, and main loops. Handoff to api-lookup for property/event names, to performance for frame budget, to networking for remotes.
---

# Scripting (Client-Side)

## Services

```luau
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local Workspace = game:GetService("Workspace")
local player = Players.LocalPlayer
```

Wrap service fetch in pcall if environment uncertain.
Cache service handles at top. Never call `GetService` inside a loop.

## Player Guards

```luau
if not player then return end
local function getCharacter(p)
    p = p or player
    if not p then return nil end
    return p.Character
end
local function getHumanoid(char)
    if not char then return nil end
    return char:FindFirstChildOfClass("Humanoid")
end
local function getRoot(char)
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart")
end
```

Reconnect on respawn:

```luau
player.CharacterAdded:Connect(function(char)
    char:WaitForChild("HumanoidRootPart", 5)
end)
```

## Full Script Structure

1. Load 2. Settings 3. Utility 4. UI 5. Main Loop 6. Save/Load

```luau
-- 1. Load
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local player = Players.LocalPlayer
if not player then return end

-- 2. Settings
local Config = {
    ESP = true,
    Interval = 0.1,
    MaxBoxes = 20,
    TeamCheck = true,
}

-- 3. Utility
local function getCharacter(p)
    p = p or player
    return p and p.Character or nil
end
local function sameTeam(a, b)
    if not Config.TeamCheck then return false end
    if not a or not b then return false end
    return a.Team ~= nil and a.Team == b.Team
end

-- 4. UI (Wild UI default)
local okUI, UI = pcall(function()
    return loadstring(game:HttpGet("https://example.com/wild-ui.lua"))()
end)
local window = nil
if okUI and UI then
    local okWin, win = pcall(function()
        return UI:CreateWindow({
            Title = "Tool",
            Mobile = UserInputService.TouchEnabled,
        })
    end)
    if okWin then window = win end
else
    warn("UI lib failed to load")
end

-- 5. Main Loop
local acc = 0
RunService.Heartbeat:Connect(function(dt)
    acc += dt
    if acc < Config.Interval then return end
    acc = 0
    local char = getCharacter()
    if not char then return end
    -- per-tick work here
end)

-- 6. Save/Load
local CONFIG_FILE = "lua-toolkit-config.json"
local function saveConfig()
    local ok, json = pcall(function()
        return HttpService:JSONEncode(Config)
    end)
    if ok and writefile then
        pcall(writefile, CONFIG_FILE, json)
    end
end
local function loadConfig()
    if not (readfile and isfile) then return end
    local okFile, has = pcall(isfile, CONFIG_FILE)
    if not okFile or not has then return end
    local ok, json = pcall(readfile, CONFIG_FILE)
    if not ok then return end
    local ok2, data = pcall(function()
        return HttpService:JSONDecode(json)
    end)
    if ok2 and type(data) == "table" then
        for k, v in pairs(data) do
            if Config[k] ~= nil and type(v) == type(Config[k]) then
                Config[k] = v
            end
        end
    end
end
loadConfig()
```

## UI Integration (Wild UI)

- One window, tabs per feature group.
- Toggle writes to Config + calls saveConfig.
- Touch: larger buttons when `TouchEnabled` true.
- Guard every UI callback with pcall so one broken toggle never kills loop.

```luau
local function bindToggle(name, default, onChange)
    local state = Config[name]
    if state == nil then state = default end
    return function(value)
        local ok, err = pcall(onChange, value)
        if not ok then warn(name, err) return end
        Config[name] = value
        saveConfig()
    end
end
```

## ESP Pattern (Drawing)

Reusable boxes, team filter, camera projection, hard render cap:

```luau
local camera = Workspace.CurrentCamera
local drawings: {} = {}
local function getBox(targetPlayer)
    local box = drawings[targetPlayer]
    if box then return box end
    local ok, obj = pcall(function()
        return Drawing.new("Square")
    end)
    if not ok then return nil end
    obj.Visible = false
    obj.Thickness = 1
    drawings[targetPlayer] = obj
    return obj
end
local function hideAll()
    for _, obj in pairs(drawings) do
        pcall(function() obj.Visible = false end)
    end
end
local function updateESP()
    if not Config.ESP then hideAll() return end
    local count = 0
    for _, target in ipairs(Players:GetPlayers()) do
        if count >= Config.MaxBoxes then break end
        if target == player then continue end
        if sameTeam(player, target) then continue end
        local char = target.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not root then continue end
        local pos, onScreen = camera:WorldToViewportPoint(root.Position)
        if not onScreen then continue end
        local box = getBox(target)
        if not box then continue end
        box.Position = Vector2.new(pos.X - 25, pos.Y - 50)
        box.Size = Vector2.new(50, 100)
        box.Visible = true
        count += 1
    end
end
Players.PlayerRemoving:Connect(function(target)
    local box = drawings[target]
    if box then
        pcall(function() box:Remove() end)
        drawings[target] = nil
    end
end)
```

Rules: never create Drawing per frame. Hide off-screen. Cap at MaxBoxes.

## Mobile Support

```luau
local isTouch = UserInputService.TouchEnabled
local btnSize = isTouch and UDim2.new(0, 120, 0, 48) or UDim2.new(0, 100, 0, 32)
local fontSize = isTouch and 18 or 14
```

Test layout at 360px width. Keep text readable.
Larger hit area on touch. Avoid hover-only controls.

## Save/Load Notes

- JSON only. Validate type before merge.
- Never overwrite Config with unknown keys.
- Call `saveConfig` on toggle change, not every frame.

## Debugging

- `warn()` for recoverable, `error()` only fatal.
- Guard: player, character, humanoid, UI lib result.
- Isolate failing block with pcall, log message.
- Repro steps: minimal config + interval 1s.
- Disable ESP first when diagnosing frame drops, then UI, then loop.
