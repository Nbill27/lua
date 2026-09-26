---
name: api-lookup
description: Use as dictionary for services, properties, and events. Handoff to scripting for wiring, to networking for remote objects.
---

# API Lookup

## Core Services

- `Players`, `RunService`, `UserInputService`, `HttpService`, `Workspace`, `Lighting`, `GuiService`, `ReplicatedStorage`.

```luau
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
```

## Player / Character

- `Players.LocalPlayer`, `.Character`, `CharacterAdded:Wait()`.
- `Humanoid`: `WalkSpeed`, `JumpPower`, `Health`, `Died`, `StateChanged`.
- `HumanoidRootPart`: `CFrame`, `Position`, `Velocity`.
- `Team`, `DisplayName`, `UserId`.

Quick check:

```luau
local char = player.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if hum then print(hum.WalkSpeed, hum.Health) end
```

Respawn guard:

```luau
local char = player.Character or player.CharacterAdded:Wait()
local root = char:WaitForChild("HumanoidRootPart", 5)
```

## UserInputService

- Props: `TouchEnabled`, `KeyboardEnabled`, `MouseEnabled`, `GamepadEnabled`.
- Events: `InputBegan`, `InputEnded`, `JumpRequest`.

```luau
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.E then print("E") end
    if input.UserInputType == Enum.UserInputType.Touch then print("touch") end
end)
```

## RunService

- `Heartbeat`, `RenderStepped`, `Stepped`. Use Heartbeat for logic.
- Callback receives `deltaTime`. Accumulate for throttle.

## Workspace / Camera

- `Workspace:GetDescendants()`, `FindFirstChild(name, recursive)`.
- `CurrentCamera`: `WorldToViewportPoint(pos)` returns `Vector3, bool`.
- `Vector3`: `Magnitude`, `Unit`. `CFrame`: `Position`, `LookVector`.
- `Vector2`: screen coords for Drawing.

```luau
local cam = Workspace.CurrentCamera
local pos, onScreen = cam:WorldToViewportPoint(root.Position)
if onScreen then print(pos.X, pos.Y) end
```

## Drawing API

- Types: `Square`, `Circle`, `Line`, `Text`.
- Props: `Visible`, `Position`, `Size`, `Color`, `Thickness`, `Transparency`.
- Always `Remove()` on cleanup. Never create per frame.

```luau
local box = Drawing.new("Square")
box.Size = Vector2.new(50, 100)
box.Visible = false
```

## HttpService

- `JSONEncode`, `JSONDecode`. Always pcall both.
- File funcs (`writefile`, `readfile`, `isfile`) depend on runtime. Guard nil.

## Enum Quick Ref

- `Enum.KeyCode.E`, `Enum.UserInputType.Touch`, `Enum.HumanoidStateType.Jumping`.
- Compare with `==`, never with string name.

## UDim2 / Gui Quick Ref

- `UDim2.new(0, 120, 0, 48)` for touch buttons, smaller for PC.
- `UDim.new(0, 14)` for padding. `Color3.fromRGB(30, 30, 30)` for panels.
- Anchor UI to corners so 360px mobile layout stays readable.

## Excluded

Server-side stores and cross-server messaging omitted. Client scope only.
When remote object needed, handoff to `networking`.
