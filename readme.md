local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "neko scripts",
   Icon = 0,
   LoadingTitle = "Rayfield Interface Suite",
   LoadingSubtitle = "by Sirius",
   ShowText = "Rayfield",
   Theme = "Bloom",
   ToggleUIKeybind = "K",
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,
   ConfigurationSaving = {
      Enabled = true,
      FolderName = nil,
      FileName = "Big Hub"
   },
   Discord = {
      Enabled = true,
      Invite = "2ZebVBdFU",
      RememberJoins = true
   },
   KeySystem = true,
   KeySettings = {
      Title = "Untitled",
      Subtitle = "Key System",
      Note = "join the discord for key",
      FileName = "Key",
      SaveKey = true,
      GrabKeyFromSite = false,
      Key = {"neva"}
   }
})

local Tab = Window:CreateTab("Main", 4483362458)

-- ✅ FOV Slider
Tab:CreateSlider({
   Name = "Field of View",
   Range = {70, 120},
   Increment = 1,
   Suffix = "°",
   CurrentValue = workspace.CurrentCamera.FieldOfView,
   Flag = "FOVSlider",
   Callback = function(Value)
      workspace.CurrentCamera.FieldOfView = Value
   end,
})

-- ✅ Variables
local flying = false
local flyConnection
local noclipping = false
local noclipConnection
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local goUp = false
local goDown = false

-- ✅ On-screen GUI buttons for mobile
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyButtons"
screenGui.ResetOnSpawn = false
screenGui.Parent = game.Players.LocalPlayer.PlayerGui

-- Up Button
local upButton = Instance.new("TextButton")
upButton.Size = UDim2.new(0, 80, 0, 80)
upButton.Position = UDim2.new(1, -100, 0.7, -90)
upButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
upButton.BackgroundTransparency = 0.3
upButton.Text = "⬆ UP"
upButton.TextColor3 = Color3.fromRGB(255, 255, 255)
upButton.TextScaled = true
upButton.Font = Enum.Font.GothamBold
upButton.Parent = screenGui

local upCorner = Instance.new("UICorner")
upCorner.CornerRadius = UDim.new(0, 12)
upCorner.Parent = upButton

-- Down Button
local downButton = Instance.new("TextButton")
downButton.Size = UDim2.new(0, 80, 0, 80)
downButton.Position = UDim2.new(1, -100, 0.7, 10)
downButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
downButton.BackgroundTransparency = 0.3
downButton.Text = "⬇ DN"
downButton.TextColor3 = Color3.fromRGB(255, 255, 255)
downButton.TextScaled = true
downButton.Font = Enum.Font.GothamBold
downButton.Parent = screenGui

local downCorner = Instance.new("UICorner")
downCorner.CornerRadius = UDim.new(0, 12)
downCorner.Parent = downButton

screenGui.Enabled = false

-- Hold up/down
upButton.MouseButton1Down:Connect(function() goUp = true; goDown = false end)
upButton.MouseButton1Up:Connect(function() goUp = false end)
downButton.MouseButton1Down:Connect(function() goDown = true; goUp = false end)
downButton.MouseButton1Up:Connect(function() goDown = false end)

-- ✅ Fly Functions
local function startFly()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    local humanoid = character:WaitForChild("Humanoid")

    humanoid.PlatformStand = true
    screenGui.Enabled = true

    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.zero
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Parent = humanoidRootPart

    local bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bodyGyro.P = 1e4
    bodyGyro.Parent = humanoidRootPart

    local flySpeed = 50
    local camera = workspace.CurrentCamera

    flyConnection = RunService.RenderStepped:Connect(function()
        local moveDir = humanoid.MoveDirection
        local verticalVelocity = 0

        if UIS:IsKeyDown(Enum.KeyCode.Space) then
            verticalVelocity = flySpeed
        elseif UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
            verticalVelocity = -flySpeed
        end

        if goUp then verticalVelocity = flySpeed
        elseif goDown then verticalVelocity = -flySpeed end

        if moveDir.Magnitude > 0 then
            bodyVelocity.Velocity = Vector3.new(moveDir.X * flySpeed, verticalVelocity, moveDir.Z * flySpeed)
        else
            bodyVelocity.Velocity = Vector3.new(0, verticalVelocity, 0)
        end

        bodyGyro.CFrame = camera.CFrame
    end)
end

local function stopFly()
    local player = game.Players.LocalPlayer
    local character = player.Character
    if not character then return end

    local humanoid = character:WaitForChild("Humanoid")
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

    humanoid.PlatformStand = false
    screenGui.Enabled = false

    if humanoidRootPart:FindFirstChild("BodyVelocity") then
        humanoidRootPart.BodyVelocity:Destroy()
    end
    if humanoidRootPart:FindFirstChild("BodyGyro") then
        humanoidRootPart.BodyGyro:Destroy()
    end

    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end

    goUp = false
    goDown = false
end

-- ✅ Noclip Functions
local function startNoclip()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()

    noclipConnection = RunService.Stepped:Connect(function()
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end)
end

local function stopNoclip()
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end

    local player = game.Players.LocalPlayer
    local character = player.Character
    if not character then return end

    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CanCollide = true
        end
    end
end

-- ✅ Fly Toggle Button
local FlyButton = Tab:CreateButton({
    Name = "Fly Toggle",
    Callback = function()
        flying = not flying
        if flying then startFly() else stopFly() end
    end,
})

-- ✅ Noclip Toggle Button
local NoclipButton = Tab:CreateButton({
    Name = "Noclip Toggle",
    Callback = function()
        noclipping = not noclipping
        if noclipping then startNoclip() else stopNoclip() end
    end,
})
