-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Disable Aladia PvP Camera Script
task.spawn(function()
    while true do
        local camScript = LocalPlayer:FindFirstChild("PlayerScripts") and LocalPlayer.PlayerScripts:FindFirstChild("CVCCamera")
        if camScript and camScript:FindFirstChild("main") then
            camScript.main.Disabled = true
        end
        task.wait(1)
    end
end)

-- Load Rayfield UI
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Window = Rayfield:CreateWindow({
    Name = "#6WILD MAINCHEAT",
    LoadingTitle = "Loading WILD ST. UI...",
    LoadingSubtitle = "BY WILD. #4EO",
    ConfigurationSaving = { Enabled = false }
})

-- Tabs
local MainTab = Window:CreateTab("Main", 4483362458)
local VisualsTab = Window:CreateTab("Visuals", 4483362458)
local CreditsTab = Window:CreateTab("Credits", 4483362458)

-- Credits
CreditsTab:CreateParagraph({
    Title = "Made By",
    Content = "40KPAKSHITT on Discord\nScripted & Branded using Rayfield UI\nJoin: https://discord.gg/YH6fxQa7uz"
})

-- Variables
local aimEnabled = false
local wallCheckEnabled = false
local fovRadius = 100
local fovCircle = nil
local highlight = nil
local rainbowHighlight = false
local ESPEnabled = false
local playerESP = {}

-- Fly Variables
local flyEnabled = false
local flySpeed = 50
local bodyVelocity

-- Toggles
MainTab:CreateToggle({
    Name = "AIM ASSIST [WILD]",
    CurrentValue = false,
    Callback = function(state)
        aimEnabled = state
    end
})

MainTab:CreateToggle({
    Name = "WALL CHECK [WILD]",
    CurrentValue = false,
    Callback = function(state)
        wallCheckEnabled = state
    end
})

MainTab:CreateToggle({
    Name = "ESP [WILD]",
    CurrentValue = false,
    Callback = function(state)
        ESPEnabled = state
        if not ESPEnabled then
            for _, box in pairs(playerESP) do
                box:Remove()
            end
            playerESP = {}
        end
    end
})

MainTab:CreateToggle({
    Name = "Fly [WILD]",
    CurrentValue = false,
    Callback = function(state)
        flyEnabled = state
        local char = LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            if flyEnabled then
                bodyVelocity = Instance.new("BodyVelocity")
                bodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
                bodyVelocity.Velocity = Vector3.new(0,0,0)
                bodyVelocity.Parent = char.HumanoidRootPart
            else
                if bodyVelocity then
                    bodyVelocity:Destroy()
                    bodyVelocity = nil
                end
            end
        end
    end
})

-- Drop Bomb on All Players
MainTab:CreateButton({
    Name = "Drop Bomb on All Players",
    Callback = function()
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                local bomb = Instance.new("Part")
                bomb.Size = Vector3.new(2,2,2)
                bomb.Shape = Enum.PartType.Ball
                bomb.BrickColor = BrickColor.new("Really red")
                bomb.Material = Enum.Material.Neon
                bomb.Anchored = false
                bomb.CanCollide = true
                bomb.Position = hrp.Position + Vector3.new(0,5,0)

                local explosion = Instance.new("Explosion")
                explosion.BlastRadius = 10
                explosion.BlastPressure = 500000

                bomb.Touched:Connect(function(hit)
                    if hit and hit.Parent ~= LocalPlayer.Character then
                        explosion.Position = bomb.Position
                        explosion.Parent = workspace
                        bomb:Destroy()
                    end
                end)

                bomb.Parent = workspace
            end
        end
    end
})

-- Visuals
VisualsTab:CreateToggle({
    Name = "Show FOV Circle [WILD]",
    CurrentValue = false,
    Callback = function(state)
        if state and not fovCircle then
            fovCircle = Drawing.new("Circle")
            fovCircle.Thickness = 2
            fovCircle.Filled = false
            fovCircle.Radius = fovRadius
            fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            fovCircle.Visible = true
        elseif fovCircle then
            fovCircle.Visible = state
        end
    end
})

VisualsTab:CreateToggle({
    Name = "Rainbow Highlights [WILD]",
    CurrentValue = false,
    Callback = function(state)
        rainbowHighlight = state
    end
})

-- Functions
local function isVisible(part)
    local origin = Camera.CFrame.Position
    local direction = (part.Position - origin)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = {LocalPlayer.Character}
    local result = workspace:Raycast(origin, direction, params)
    return not result or result.Instance:IsDescendantOf(part.Parent)
end

local function getTarget()
    local closest, shortest = nil, fovRadius
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local humanoid = plr.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local part = plr.Character:FindFirstChild("Head") or plr.Character:FindFirstChild("HumanoidRootPart")
                if part then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen then
                        local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                        if dist < shortest and (not wallCheckEnabled or isVisible(part)) then
                            closest, shortest = plr, dist
                        end
                    end
                end
            end
        end
    end
    return closest
end

-- Main Loop
local currentTarget = nil
RunService.RenderStepped:Connect(function()
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

    -- Update FOV circle
    if fovCircle then
        local screenSize = Camera.ViewportSize
        fovCircle.Position = Vector2.new(screenSize.X/2, screenSize.Y/2)
        fovCircle.Radius = fovRadius
    end

    -- AIM Assist
    if aimEnabled then
        local target = getTarget()
        if target and target.Character then
            local part = target.Character:FindFirstChild("HumanoidRootPart") or target.Character:FindFirstChild("Head")
            if part then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, part.Position + Vector3.new(0,1.4,0))
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    highlight.FillTransparency = 0.5
                    highlight.OutlineTransparency = 1
                    highlight.Parent = game.CoreGui
                end
                if target ~= currentTarget then
                    currentTarget = target
                    highlight.Adornee = target.Character
                end
                if rainbowHighlight then
                    highlight.FillColor = Color3.fromHSV(tick()%5/5,1,1)
                else
                    highlight.FillColor = Color3.fromRGB(0,255,0)
                end
            end
        else
            currentTarget = nil
            if highlight then highlight.Adornee = nil end
        end
    else
        currentTarget = nil
        if highlight then highlight.Adornee = nil end
    end

    -- ESP
    if ESPEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = plr.Character.HumanoidRootPart
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    if not playerESP[plr] then
                        local box = Drawing.new("Square")
                        box.Color = Color3.fromRGB(255,0,0)
                        box.Thickness = 2
                        box.Filled = false
                        playerESP[plr] = box
                    end
                    local size = Vector2.new(30,50)
                    playerESP[plr].Position = Vector2.new(screenPos.X - size.X/2, screenPos.Y - size.Y/2)
                    playerESP[plr].Size = size
                    playerESP[plr].Visible = true
                elseif playerESP[plr] then
                    playerESP[plr].Visible = false
                end
            elseif playerESP[plr] then
                playerESP[plr].Visible = false
            end
        end
    end

    -- Fly
    if flyEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and bodyVelocity then
        local hrp = LocalPlayer.Character.HumanoidRootPart
        local moveVector = Vector3.new(0,0,0)
        local speed = flySpeed
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVector += Camera.CFrame.LookVector*speed end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVector -= Camera.CFrame.LookVector*speed end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVector -= Camera.CFrame.RightVector*speed end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVector += Camera.CFrame.RightVector*speed end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVector += Vector3.new(0,speed,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVector -= Vector3.new(0,speed,0) end
        bodyVelocity.Velocity = moveVector
    end
end)
