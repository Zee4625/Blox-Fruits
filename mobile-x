-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TeleportService = game:GetService("TeleportService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- Rebind on respawn
local function rebindCharacter()
    character = player.Character or player.CharacterAdded:Wait()
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    humanoid = character:WaitForChild("Humanoid")
end
player.CharacterAdded:Connect(rebindCharacter)

-- Variables
local flyActive = false
local flightActive = false
local xrayEnabled = false
local spyTarget = nil
local moveSpeed = 39
local flyConnection
local bodyGyro, bodyVelocity

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AloneZeeHUB"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

-- Main Frame (wide and draggable)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 640, 0, 500)
mainFrame.Position = UDim2.new(0, 50, 0, 50)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

-- Gradient Background for mainFrame
local grad = Instance.new("UIGradient", mainFrame)
grad.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(170, 0, 255)),    -- Purple
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 0, 0)),    -- Red
    ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 0))     -- Yellow
}
grad.Rotation = 45

-- Title Bar Frame
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundTransparency = 0.9
titleBar.Parent = mainFrame

-- RGB Neon Title Label
local titleLabel = Instance.new("TextLabel")
titleLabel.Text = "Alone x ZeeHUB"
titleLabel.Size = UDim2.new(1, -80, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 28
titleLabel.TextStrokeColor3 = Color3.new(1,1,1)
titleLabel.TextStrokeTransparency = 0
titleLabel.Parent = titleBar

-- Function to cycle RGB colors in the title
local hue = 0
RunService.Heartbeat:Connect(function(delta)
    hue = (hue + delta*0.2) % 1
    local r, g, b = Color3.fromHSV(hue, 1, 1):ToHSV()
    -- To create red-purple-yellow cycling, manipulate colors manually:
    -- But for simplicity, use fromHSV cycling with saturation and value 1
    titleLabel.TextColor3 = Color3.fromHSV(hue, 0.8, 1)
end)

-- Close Button (Red X)
local closeButton = Instance.new("TextButton")
closeButton.Size = UDim2.new(0, 35, 0, 30)
closeButton.Position = UDim2.new(1, -40, 0, 5)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 30, 30)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.new(1,1,1)
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 22
closeButton.AutoButtonColor = false
closeButton.Parent = titleBar

closeButton.MouseEnter:Connect(function()
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
end)
closeButton.MouseLeave:Connect(function()
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 30, 30)
end)
closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Minimize Button (Yellow U)
local minimizeButton = Instance.new("TextButton")
minimizeButton.Size = UDim2.new(0, 35, 0, 30)
minimizeButton.Position = UDim2.new(1, -80, 0, 5)
minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 210, 30)
minimizeButton.Text = "U"
minimizeButton.TextColor3 = Color3.new(0,0,0)
minimizeButton.Font = Enum.Font.GothamBold
minimizeButton.TextSize = 22
minimizeButton.AutoButtonColor = false
minimizeButton.Parent = titleBar

local minimized = false
minimizeButton.MouseEnter:Connect(function()
    minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 255, 80)
end)
minimizeButton.MouseLeave:Connect(function()
    minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 210, 30)
end)
minimizeButton.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        mainFrame.Size = UDim2.new(0, 200, 0, 40)
        for _, v in pairs(mainFrame:GetChildren()) do
            if v ~= titleBar then
                v.Visible = false
            end
        end
    else
        mainFrame.Size = UDim2.new(0, 640, 0, 500)
        for _, v in pairs(mainFrame:GetChildren()) do
            if v ~= titleBar then
                v.Visible = true
            end
        end
    end
end)

-- Container for buttons and toggles below title bar
local contentFrame = Instance.new("ScrollingFrame")
contentFrame.Size = UDim2.new(1, -20, 1, -50)
contentFrame.Position = UDim2.new(0, 10, 0, 45)
contentFrame.BackgroundTransparency = 1
contentFrame.ScrollBarThickness = 6
contentFrame.Parent = mainFrame

local uiLayout = Instance.new("UIListLayout")
uiLayout.SortOrder = Enum.SortOrder.LayoutOrder
uiLayout.Padding = UDim.new(0, 8)
uiLayout.Parent = contentFrame

-- Helper function: create toggle button
local function CreateToggle(text, tip, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 38)
    frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
    frame.BorderSizePixel = 0
    frame.Parent = contentFrame
    frame.AutomaticSize = Enum.AutomaticSize.Y

    local label = Instance.new("TextLabel")
    label.Text = text
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 18
    label.TextColor3 = Color3.fromRGB(200, 200, 200)
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(0.8, 0, 1, 0)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 60, 0, 24)
    toggleBtn.Position = UDim2.new(1, -65, 0, 7)
    toggleBtn.BackgroundColor3 = default and Color3.fromRGB(0, 200, 50) or Color3.fromRGB(60, 60, 60)
    toggleBtn.Text = default and "ON" or "OFF"
    toggleBtn.TextColor3 = Color3.new(1,1,1)
    toggleBtn.Font = Enum.Font.GothamBold
    toggleBtn.TextSize = 16
    toggleBtn.AutoButtonColor = false
    toggleBtn.Parent = frame

    local enabled = default
    toggleBtn.MouseButton1Click:Connect(function()
        enabled = not enabled
        toggleBtn.BackgroundColor3 = enabled and Color3.fromRGB(0, 200, 50) or Color3.fromRGB(60, 60, 60)
        toggleBtn.Text = enabled and "ON" or "OFF"
        callback(enabled)
    end)
    return frame
end

-- Helper function: create button
local function CreateButton(text, tip, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = text
    btn.Parent = contentFrame

    btn.MouseEnter:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    end)
    btn.MouseLeave:Connect(function()
        btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end)
    btn.MouseButton1Click:Connect(function()
        callback()
    end)
    return btn
end

-- Fly Speed Slider UI
local flySpeedFrame = Instance.new("Frame")
flySpeedFrame.Size = UDim2.new(1, 0, 0, 60)
flySpeedFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
flySpeedFrame.BorderSizePixel = 0
flySpeedFrame.Parent = contentFrame

local flySpeedLabel = Instance.new("TextLabel")
flySpeedLabel.Text = "Fly Speed"
flySpeedLabel.Size = UDim2.new(1, 0, 0, 20)
flySpeedLabel.Position = UDim2.new(0, 0, 0, 5)
flySpeedLabel.Font = Enum.Font.GothamBold
flySpeedLabel.TextSize = 18
flySpeedLabel.TextColor3 = Color3.fromRGB(0, 255, 0)
flySpeedLabel.BackgroundTransparency = 1
flySpeedLabel.Parent = flySpeedFrame

local sliderFrame = Instance.new("Frame")
sliderFrame.Size = UDim2.new(1, -40, 0, 30)
sliderFrame.Position = UDim2.new(0, 20, 0, 25)
sliderFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
sliderFrame.BorderSizePixel = 0
sliderFrame.Parent = flySpeedFrame

local sliderBar = Instance.new("Frame")
sliderBar.Size = UDim2.new(moveSpeed/100, 0, 1, 0)
sliderBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
sliderBar.BorderSizePixel = 0
sliderBar.Parent = sliderFrame

local sliderButton = Instance.new("TextButton")
sliderButton.Size = UDim2.new(0, 20, 1, 0)
sliderButton.Position = UDim2.new(moveSpeed/100, -10, 0, 0)
sliderButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
sliderButton.Text = ""
sliderButton.AutoButtonColor = false
sliderButton.Parent = sliderFrame

local dragging = false
local function updateSpeed(inputPosX)
    local relativePos = math.clamp(inputPosX - sliderFrame.AbsolutePosition.X, 0, sliderFrame.AbsoluteSize.X)
    local percent = relativePos / sliderFrame.AbsoluteSize.X
    sliderBar.Size = UDim2.new(percent, 0, 1, 0)
    sliderButton.Position = UDim2.new(percent, -10, 0, 0)
    moveSpeed = math.floor(percent * 100)
    if moveSpeed < 1 then moveSpeed = 1 end
end

sliderButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
    end
end)
sliderButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        updateSpeed(input.Position.X)
    end
end)

-- Fly Functionality
local function toggleFly(state)
    flyActive = state
    if flyConnection then flyConnection:Disconnect() end
    if not flyActive then
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVelocity then bodyVelocity:Destroy() end
        flySpeedFrame.Visible = false
        return
    end

    flySpeedFrame.Visible = true

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.P = 9e4
    bodyGyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
    bodyGyro.cframe = humanoidRootPart.CFrame
    bodyGyro.Parent = humanoidRootPart

    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.velocity = Vector3.new(0, 0, 0)
    bodyVelocity.maxForce = Vector3.new(9e9, 9e9, 9e9)
    bodyVelocity.Parent = humanoidRootPart

    flyConnection = RunService.Heartbeat:Connect(function()
        if not flyActive then return end
        bodyGyro.cframe = workspace.CurrentCamera.CFrame

        local moveVec = Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then
            moveVec += workspace.CurrentCamera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then
            moveVec -= workspace.CurrentCamera.CFrame.LookVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then
            moveVec -= workspace.CurrentCamera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then
            moveVec += workspace.CurrentCamera.CFrame.RightVector
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            moveVec += Vector3.new(0, 1, 0)
        end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
            moveVec -= Vector3.new(0, 1, 0)
        end

        if moveVec.Magnitude > 0 then
            bodyVelocity.velocity = moveVec.Unit * moveSpeed
        else
            bodyVelocity.velocity = Vector3.new(0, 0, 0)
        end
    end)
end

-- Speed Mode (walk speed override)
local flightConnection
local function toggleSpeedMode(state)
    flightActive = state
    if flightConnection then flightConnection:Disconnect() end

    if flightActive then
        humanoid.WalkSpeed = moveSpeed
        flightConnection = RunService.Heartbeat:Connect(function()
            humanoid.WalkSpeed = moveSpeed
        end)
    else
        flightConnection = RunService.Heartbeat:Connect(function()
            humanoid.WalkSpeed = 16
        end)
    end
end

-- Jump Boost toggle
local jumpBoostConnection
local function toggleJumpBoost(state)
    if jumpBoostConnection then jumpBoostConnection:Disconnect() end
    if state then
        jumpBoostConnection = RunService.Heartbeat:Connect(function()
            humanoid.JumpPower = 75
        end)
    else
        humanoid.JumpPower = 50
    end
end

-- X-Ray toggle
local function toggleXray(state)
    xrayEnabled = state
    for _, part in pairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" and not part:IsDescendantOf(character) then
            if xrayEnabled then
                part.Transparency = 0.5
                if not part:FindFirstChild("XrayOriginalCanCollide") then
                    local origCanCollide = Instance.new("BoolValue")
                    origCanCollide.Name = "XrayOriginalCanCollide"
                    origCanCollide.Value = part.CanCollide
                    origCanCollide.Parent = part
                end
                part.CanCollide = false
            else
                if part:FindFirstChild("XrayOriginalCanCollide") then
                    part.CanCollide = part.XrayOriginalCanCollide.Value
                    part.XrayOriginalCanCollide:Destroy()
                else
                    part.CanCollide = true
                end
                part.Transparency = 0
            end
        end
    end
end

-- ESP Highlight for all players (white)
local highlights = {}

local function createESP(plr)
    if plr == player then return end
    local ch = plr.Character
    if ch and ch:FindFirstChild("HumanoidRootPart") then
        local highlight = Instance.new("Highlight")
        highlight.Adornee = ch
        highlight.FillColor = Color3.new(1,1,1)
        highlight.FillTransparency = 0.6
        highlight.OutlineColor = Color3.new(1,1,1)
        highlight.OutlineTransparency = 0
        highlight.Parent = ch
        highlights[plr] = highlight
    end
end

local function removeESP(plr)
    if highlights[plr] then
        highlights[plr]:Destroy()
        highlights[plr] = nil
    end
end

local espEnabled = false
local function toggleESP(state)
    espEnabled = state
    if espEnabled then
        for _, plr in pairs(Players:GetPlayers()) do
            createESP(plr)
        end
        Players.PlayerAdded:Connect(function(plr)
            if espEnabled then
                plr.CharacterAdded:Connect(function()
                    wait(0.1)
                    createESP(plr)
                end)
            end
        end)
        Players.PlayerRemoving:Connect(function(plr)
            removeESP(plr)
        end)
    else
        for plr, hl in pairs(highlights) do
            hl:Destroy()
        end
        highlights = {}
    end
end

-- Teleport to player (Spy player)
local function teleportToPlayer(plrName)
    local target = Players:FindFirstChild(plrName)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        humanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0,5,0)
        spyTarget = target
    else
        warn("Player not found or no HumanoidRootPart!")
    end
end

-- Button and Toggles Creation --

-- ESP toggle
CreateToggle("ESP (White Highlight)", "Highlights all players with white glow", false, function(state)
    toggleESP(state)
end)

-- Fly toggle
CreateToggle("Fly Mode", "Fly using WASD + Space + Ctrl", false, function(state)
    toggleFly(state)
end)

-- Speed mode toggle
CreateToggle("Speed Mode", "Increase walk speed", false, function(state)
    toggleSpeedMode(state)
end)

-- Jump Boost toggle
CreateToggle("Jump Boost", "Higher jumps", false, function(state)
    toggleJumpBoost(state)
end)

-- X-Ray toggle
CreateToggle("X-Ray", "Makes map parts semi-transparent", false, function(state)
    toggleXray(state)
end)

-- Spy Player teleport button & input
local spyFrame = Instance.new("Frame")
spyFrame.Size = UDim2.new(1, 0, 0, 50)
spyFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
spyFrame.BorderSizePixel = 0
spyFrame.Parent = contentFrame

local spyLabel = Instance.new("TextLabel")
spyLabel.Text = "Teleport to Player:"
spyLabel.Font = Enum.Font.GothamBold
spyLabel.TextSize = 18
spyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
spyLabel.BackgroundTransparency = 1
spyLabel.Size = UDim2.new(0.5, 0, 1, 0)
spyLabel.Position = UDim2.new(0, 5, 0, 0)
spyLabel.Parent = spyFrame

local spyInput = Instance.new("TextBox")
spyInput.PlaceholderText = "PlayerName"
spyInput.Font = Enum.Font.Gotham
spyInput.TextSize = 18
spyInput.TextColor3 = Color3.fromRGB(0, 0, 0)
spyInput.BackgroundColor3 = Color3.fromRGB(240, 240, 240)
spyInput.Size = UDim2.new(0.35, 0, 0.7, 0)
spyInput.Position = UDim2.new(0.5, 0, 0.15, 0)
spyInput.ClearTextOnFocus = false
spyInput.Parent = spyFrame

local spyButton = Instance.new("TextButton")
spyButton.Text = "Teleport"
spyButton.Font = Enum.Font.GothamBold
spyButton.TextSize = 18
spyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
spyButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
spyButton.Size = UDim2.new(0.12, 0, 0.7, 0)
spyButton.Position = UDim2.new(0.85, 0, 0.15, 0)
spyButton.Parent = spyFrame

spyButton.MouseEnter:Connect(function()
    spyButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
end)
spyButton.MouseLeave:Connect(function()
    spyButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
end)
spyButton.MouseButton1Click:Connect(function()
    local targetName = spyInput.Text
    if targetName ~= "" then
        teleportToPlayer(targetName)
    end
end)

-- Wallhack toggle (makes players semi-transparent)
local wallhackEnabled = false
local wallhackOriginals = {}

local function toggleWallhack(state)
    wallhackEnabled = state
    if wallhackEnabled then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr ~= player and plr.Character then
                for _, part in pairs(plr.Character:GetChildren()) do
                    if part:IsA("BasePart") then
                        wallhackOriginals[part] = part.Transparency
                        part.Transparency = 0.5
                    end
                end
            end
        end
        Players.PlayerAdded:Connect(function(plr)
            if wallhackEnabled then
                plr.CharacterAdded:Connect(function(char)
                    wait(0.1)
                    for _, part in pairs(char:GetChildren()) do
                        if part:IsA("BasePart") then
                            wallhackOriginals[part] = part.Transparency
                            part.Transparency = 0.5
                        end
                    end
                end)
            end
        end)
        Players.PlayerRemoving:Connect(function(plr)
            if plr.Character then
                for _, part in pairs(plr.Character:GetChildren()) do
                    if part:IsA("BasePart") and wallhackOriginals[part] then
                        part.Transparency = wallhackOriginals[part]
                        wallhackOriginals[part] = nil
                    end
                end
            end
        end)
    else
        for part, oldTransparency in pairs(wallhackOriginals) do
            if part and part.Parent then
                part.Transparency = oldTransparency
            end
        end
        wallhackOriginals = {}
    end
end

CreateToggle("Wallhack", "Make other players semi-transparent", false, function(state)
    toggleWallhack(state)
end)

-- Finished
print("Alone x ZeeHUB loaded successfully.")
