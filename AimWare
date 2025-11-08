-- PracticeTrainerClient Full (LocalScript)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Load Rayfield
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/UI-Interface/CustomFIeld/main/RayField.lua'))()

-- Default state: all toggles OFF
local state = {
    esp = false,
    box = false,
    skeleton = false,
    health = false,
    line = false,
    wallCheck = false,
    aimbot = false,
    silentAim = false,
    aimbotPower = 25,
    fovSize = 150,
}

-- Create Rayfield Window
local Window = Rayfield:CreateWindow({
    Name = "Practice Trainer",
    LoadingTitle = "Practice Trainer",
    LoadingSubtitle = "Rayfield UI",
    ConfigurationSaving = {Enabled = false},
    Discord = {Enabled = false},
    KeySystem = false
})

local MainSection = Window:CreateSection("Main")
local EspSection = Window:CreateSection("ESP")
local AimSection = Window:CreateSection("Aimbot")

-- ESP Toggles
EspSection:CreateToggle({Name = "ESP Master", CurrentValue = false, Callback = function(v) state.esp=v end})
EspSection:CreateToggle({Name = "Box ESP", CurrentValue = false, Callback = function(v) state.box=v end})
EspSection:CreateToggle({Name = "Line ESP", CurrentValue = false, Callback = function(v) state.line=v end})
EspSection:CreateToggle({Name = "Skeleton ESP", CurrentValue = false, Callback = function(v) state.skeleton=v end})
EspSection:CreateToggle({Name = "Health ESP", CurrentValue = false, Callback = function(v) state.health=v end})
EspSection:CreateToggle({Name = "Wall Check", CurrentValue = false, Callback = function(v) state.wallCheck=v end})

-- Aimbot toggles & sliders
AimSection:CreateToggle({Name="Aimbot", CurrentValue=false, Callback=function(v) state.aimbot=v end})
AimSection:CreateToggle({Name="Silent Aim", CurrentValue=false, Callback=function(v) state.silentAim=v end})
AimSection:CreateSlider({Name="Aimbot Power", Range={0,100}, Increment=1, CurrentValue=25, Callback=function(v) state.aimbotPower=v end})
AimSection:CreateSlider({Name="FOV Size", Range={50,500}, Increment=1, CurrentValue=150, Callback=function(v) state.fovSize=v end})

-- Global state access
_G.PracticeTrainerConfig = state

-- --- Helper Functions ---
local function getHead(model)
    return model:FindFirstChild("Head") or model:FindFirstChildWhichIsA("BasePart")
end

local highlights, billboards, lines, skeletons = {}, {}, {}, {}

local function ensureVisuals(target)
    if highlights[target] then return end
    local h = Instance.new("Highlight")
    h.Adornee = target
    h.Enabled = false
    h.FillTransparency = 0.6
    h.Parent = game:GetService("CoreGui")
    highlights[target] = h

    local head = getHead(target)
    if head then
        -- Health Billboard
        local bb = Instance.new("BillboardGui")
        bb.Size = UDim2.new(0,80,0,24)
        bb.StudsOffset = Vector3.new(0,2,0)
        bb.AlwaysOnTop = true
        bb.Adornee = head
        local txt = Instance.new("TextLabel", bb)
        txt.Size = UDim2.fromScale(1,1)
        txt.BackgroundTransparency = 1
        txt.TextScaled = true
        txt.Text = ""
        txt.Font = Enum.Font.SourceSansBold
        txt.TextColor3 = Color3.new(1,1,1)
        bb.Parent = game:GetService("CoreGui")
        billboards[target] = txt

        -- Line ESP
        local line = Instance.new("Part")
        line.Anchored = true
        line.CanCollide = false
        line.Size = Vector3.new(0.2,0.2,0.2)
        line.Transparency = 0.5
        line.Color = Color3.new(1,0,0)
        line.Parent = game:GetService("CoreGui")
        lines[target] = line

        -- Skeleton (basic)
        skeletons[target] = {}
        for _, partName in pairs({"Torso","Left Arm","Right Arm","Left Leg","Right Leg","Head"}) do
            local part = target:FindFirstChild(partName)
            if part then
                local s = Instance.new("BoxHandleAdornment")
                s.Adornee = part
                s.Size = part.Size
                s.AlwaysOnTop = true
                s.ZIndex = 2
                s.Transparency = 0.6
                s.Color = Color3.fromRGB(255,0,0)
                s.Parent = game:GetService("CoreGui")
                table.insert(skeletons[target],s)
            end
        end
    end
end

local function removeVisuals(target)
    if highlights[target] then highlights[target]:Destroy(); highlights[target]=nil end
    if billboards[target] then billboards[target].Parent:Destroy(); billboards[target]=nil end
    if lines[target] then lines[target]:Destroy(); lines[target]=nil end
    if skeletons[target] then
        for _, s in pairs(skeletons[target]) do s:Destroy() end
        skeletons[target]=nil
    end
end

local function isVisible(part)
    local origin = Camera.CFrame.Position
    local dir = part.Position - origin
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
    rayParams.IgnoreWater = true
    local result = Workspace:Raycast(origin, dir, rayParams)
    return not result or result.Instance:IsDescendantOf(part)
end

local function screenDistance(pos)
    local sp, onScreen = Camera:WorldToViewportPoint(pos)
    if not onScreen then return math.huge, sp end
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    return (Vector2.new(sp.X, sp.Y)-center).Magnitude, sp
end

local function findBestTarget()
    local folder = Workspace:FindFirstChild("PracticeTargets")
    if not folder then return nil, nil end
    local best, bestDist, bestScreen = nil, math.huge, nil
    for _, t in ipairs(folder:GetChildren()) do
        local head = getHead(t)
        if head then
            local dist, screenPos = screenDistance(head.Position)
            if dist <= state.fovSize and dist < bestDist then
                best = t
                bestDist = dist
                bestScreen = screenPos
            end
        end
    end
    return best, bestScreen
end

local function aimAtTarget(target, dt)
    if not target then return end
    local head = getHead(target)
    if not head then return end
    local cf = CFrame.new(Camera.CFrame.Position, head.Position)
    local power = math.clamp(state.aimbotPower/100,0,1)
    Camera.CFrame = Camera.CFrame:Lerp(cf, math.clamp(4*power*dt,0,1))
end

local function hitTarget(target)
    if not target then return end
    local hv = target:FindFirstChild("Health")
    if hv then
        hv.Value = math.max(0,hv.Value-25)
        if highlights[target] then
            highlights[target].Enabled = true
            delay(0.12,function() if highlights[target] then highlights[target].Enabled=false end end)
        end
    end
end

-- --- Main loop ---
RunService.RenderStepped:Connect(function(dt)
    if not LocalPlayer:FindFirstChild("InPractice") or not LocalPlayer.InPractice.Value then
        for k,_ in pairs(highlights) do removeVisuals(k) end
        return
    end

    local folder = Workspace:FindFirstChild("PracticeTargets")
    if not folder then return end

    local known = {}
    for _, t in ipairs(folder:GetChildren()) do
        known[t]=true
        ensureVisuals(t)
    end
    for k,_ in pairs(highlights) do if not known[k] then removeVisuals(k) end end

    for t,h in pairs(highlights) do
        local head = getHead(t)
        if head then
            h.Enabled = state.esp
            if state.wallCheck then
                h.FillColor = isVisible(head) and Color3.fromRGB(255,0,0) or Color3.fromRGB(0,0,0)
                h.OutlineColor = h.FillColor
            else
                h.FillColor = Color3.fromRGB(80,160,255)
                h.OutlineColor = Color3.fromRGB(40,80,180)
            end

            if billboards[t] and state.health then
                local hv = t:FindFirstChild("Health")
                billboards[t].Text = hv and hv.Value.." HP" or ""
            else
                if billboards[t] then billboards[t].Text="" end
            end

            if lines[t] and state.line then
                lines[t].CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
                lines[t].Size = Vector3.new(0.2,0.2,(head.Position-Camera.CFrame.Position).Magnitude)
            end

            if skeletons[t] then
                for _, s in pairs(skeletons[t]) do
                    s.Enabled = state.skeleton
                end
            end
        end
    end

    if state.aimbot then
        local best,_ = findBestTarget()
        if best then aimAtTarget(best, dt) end
    end
end)

-- Mouse click for Silent Aim
Mouse.Button1Down:Connect(function()
    if not LocalPlayer:FindFirstChild("InPractice") or not LocalPlayer.InPractice.Value then return end
    if state.silentAim then
        local best,_ = findBestTarget()
        if best then hitTarget(best) end
    else
        local origin = Camera.CFrame.Position
        local dir = Camera.CFrame.LookVector*1000
        local rayParams = RaycastParams.new()
        rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
        rayParams.FilterType = Enum.RaycastFilterType.Blacklist
        rayParams.IgnoreWater = true
        local result = Workspace:Raycast(origin, dir, rayParams)
        if result and result.Instance then
            local model = result.Instance:FindFirstAncestorOfClass("Model")
            if model and model.Parent==Workspace:FindFirstChild("PracticeTargets") then
                local hv = model:FindFirstChild("Health")
                if hv then hv.Value = math.max(0,hv.Value-20) end
            end
        end
    end
end)
