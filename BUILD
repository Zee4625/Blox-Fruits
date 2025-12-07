--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
-- Client-only Block Builder with stacking support
-- Paste this LocalScript into StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local ContextActionService = game:GetService("ContextActionService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- ===== Utilities =====
local function new(className, props)
    local obj = Instance.new(className)
    for k,v in pairs(props or {}) do obj[k] = v end
    return obj
end

local function hsvColor(h) return Color3.fromHSV(h%1, 0.8, 0.95) end
local function clamp(n, a, b) return (n < a and a) or (n > b and b) or n end
local function roundTo(v, snap) return math.floor((v + (snap * 0.5)) / snap) * snap end
local function roundVector(v, snap) return Vector3.new(roundTo(v.X, snap), roundTo(v.Y, snap), roundTo(v.Z, snap)) end

-- ===== Client folder for parts =====
local clientFolderName = "ClientBlocks_" .. LocalPlayer.Name
local clientFolder = workspace:FindFirstChild(clientFolderName) or new("Folder", {Name = clientFolderName, Parent = workspace})

-- ===== UI (compact) =====
local gui = new("ScreenGui", {Parent = LocalPlayer:WaitForChild("PlayerGui"), Name = "CompactBuildUI", ResetOnSpawn = false})
gui.IgnoreGuiInset = true

local frame = new("Frame", {
    Parent = gui,
    AnchorPoint = Vector2.new(1, 0),
    Position = UDim2.new(1, -12, 0, 12),
    Size = UDim2.new(0, 220, 0, 220),
    BackgroundColor3 = Color3.fromRGB(10,10,10),
    BorderSizePixel = 0
})
new("UICorner", {Parent = frame, CornerRadius = UDim.new(0, 10)})
local stroke = new("UIStroke", {Parent = frame, Color = Color3.fromRGB(50,50,50), Thickness = 1})

local tabCol = new("Frame", {Parent = frame, Size = UDim2.new(0, 70, 1, 0), BackgroundTransparency = 1})
new("UIListLayout", {Parent = tabCol, Padding = UDim.new(0, 6), FillDirection = Enum.FillDirection.Vertical})

local content = new("Frame", {Parent = frame, Position = UDim2.new(0, 78, 0, 8), Size = UDim2.new(1, -86, 1, -16), BackgroundTransparency = 1})
local scroll = new("ScrollingFrame", {Parent = content, Size = UDim2.new(1, 0, 1, 0), CanvasSize = UDim2.new(0,0), ScrollBarThickness = 6, BackgroundTransparency = 1})
local listLayout = new("UIListLayout", {Parent = scroll, Padding = UDim.new(0,6)})

local function clearContent()
    for _,c in ipairs(scroll:GetChildren()) do
        if c:IsA("TextButton") or c:IsA("TextLabel") or c:IsA("Frame") then
            c:Destroy()
        end
    end
end

local function addLabel(text)
    local lbl = new("TextLabel", {
        Parent = scroll,
        Size = UDim2.new(1, -6, 0, 20),
        BackgroundTransparency = 1,
        Text = text,
        TextColor3 = Color3.fromRGB(200,200,200),
        Font = Enum.Font.GothamBold,
        TextSize = 13,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    return lbl
end

local function addButton(text, callback)
    local b = new("TextButton", {
        Parent = scroll,
        Size = UDim2.new(1, -6, 0, 26),
        BackgroundColor3 = Color3.fromRGB(30,30,30),
        Text = text,
        TextColor3 = Color3.new(1,1,1),
        Font = Enum.Font.Gotham,
        TextSize = 13,
        AutoButtonColor = true
    })
    new("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)})
    b.MouseButton1Click:Connect(function() pcall(callback) end)
    return b
end

-- ===== Build state =====
local state = {
    mode = "BUILD", -- BUILD | EDIT
    ghostSize = Vector3.new(4,1,4),
    ghostColor = Color3.fromRGB(150,150,150),
    snap = 1,
    rotSnap = 15,
    scaleSnap = 0.5,
    collision = true
}

-- ===== Ghost preview =====
local ghost = new("Part", {
    Name = "GhostPreview",
    Anchored = true,
    CanCollide = false,
    Transparency = 0.6,
    Color = state.ghostColor,
    Material = Enum.Material.SmoothPlastic,
    Size = state.ghostSize,
    Parent = workspace
})
local ghostBox = new("SelectionBox", {Parent = ghost, Adornee = ghost, LineThickness = 0.02, Color3 = Color3.fromRGB(255,255,255)})

local hue = 0
RunService.RenderStepped:Connect(function(dt)
    hue = (hue + dt * 0.2) % 1
    ghostBox.Color3 = hsvColor(hue)
    stroke.Color = hsvColor((hue + 0.5) % 1)
end)

-- ===== Raycast params =====
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude

local function getIgnoreList()
    -- IMPORTANT: do NOT exclude clientFolder here so raycasts can hit client blocks (stacking)
    local ignore = {ghost}
    local char = LocalPlayer.Character
    if char then table.insert(ignore, char) end
    return ignore
end

local lastNormal = Vector3.new(0,1,0)
local function mouseTarget()
    rayParams.FilterDescendantsInstances = getIgnoreList()
    local origin = Camera.CFrame.Position
    local dir = (Mouse.Hit.Position - origin)
    if dir.Magnitude == 0 then dir = Camera.CFrame.LookVector * 1000 end
    local result = workspace:Raycast(origin, dir.Unit * 1000, rayParams)
    if result then
        return result.Position, result.Normal, result.Instance
    end
    -- fallback to mouse hit
    return Mouse.Hit.Position, Vector3.new(0,1,0), nil
end

local function updateGhost()
    local pos, normal, hitInstance = mouseTarget()
    lastNormal = normal or Vector3.new(0,1,0)
    -- Snap position to grid
    local snapped = roundVector(pos, state.snap)
    -- Offset so the ghost sits on top of the surface (half height)
    local offset = lastNormal.Unit * (state.ghostSize.Y / 2)
    -- If we hit a part, align to its surface exactly (so stacking works)
    if hitInstance and hitInstance:IsA("BasePart") then
        -- compute surface point: move along normal by half of ghost height
        ghost.CFrame = CFrame.new(snapped + offset, snapped + offset + Camera.CFrame.LookVector)
    else
        ghost.CFrame = CFrame.new(snapped + offset, snapped + offset + Camera.CFrame.LookVector)
    end
end

RunService.RenderStepped:Connect(function()
    if state.mode == "BUILD" then
        updateGhost()
    end
end)

-- ===== Place block at ghost =====
local function placeBlockAtGhost()
    local block = Instance.new("Part")
    block.Size = state.ghostSize
    block.Material = Enum.Material.SmoothPlastic
    block.Color = state.ghostColor
    block.Anchored = true
    block.CanCollide = state.collision
    block.Name = "ClientBlock"
    block.Parent = clientFolder
    block.CFrame = ghost.CFrame

    local sel = Instance.new("SelectionBox")
    sel.Adornee = block
    sel.LineThickness = 0.02
    sel.Color3 = Color3.fromRGB(255,255,255)
    sel.Parent = block

    return block
end

-- ===== Place block at player's feet (gray block) =====
local function placeBlockAtFeet()
    local char = LocalPlayer.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
    if not hrp then return end

    local block = Instance.new("Part")
    block.Size = Vector3.new(4, 1, 4)
    block.Material = Enum.Material.SmoothPlastic
    block.Color = Color3.fromRGB(160,160,160)
    block.Anchored = true
    block.CanCollide = true
    block.Name = "ClientBlock"
    block.Parent = clientFolder

    -- Position block so top aligns with feet (place under player)
    local feetY = hrp.Position.Y - (hrp.Size.Y / 2)
    local halfY = block.Size.Y / 2
    local pos = Vector3.new(hrp.Position.X, feetY - halfY - 0.1, hrp.Position.Z)
    block.CFrame = CFrame.new(pos)

    local sel = Instance.new("SelectionBox")
    sel.Adornee = block
    sel.LineThickness = 0.02
    sel.Color3 = Color3.fromRGB(255,255,255)
    sel.Parent = block

    return block
end

-- ===== Editing =====
local selectedBlock = nil

local function selectNearestBlock()
    rayParams.FilterDescendantsInstances = getIgnoreList()
    local origin = Camera.CFrame.Position
    local dir = (Mouse.Hit.Position - origin)
    if dir.Magnitude == 0 then dir = Camera.CFrame.LookVector * 1000 end
    local result = workspace:Raycast(origin, dir.Unit * 1000, rayParams)
    if result then
        local closest, dist = nil, math.huge
        for _,p in ipairs(clientFolder:GetChildren()) do
            if p:IsA("BasePart") then
                local d = (p.Position - result.Position).Magnitude
                if d < dist then closest = p dist = d end
            end
        end
        selectedBlock = closest
        return selectedBlock
    end
    selectedBlock = nil
    return nil
end

local function nudgeSelected(delta)
    if selectedBlock then
        local newPos = selectedBlock.Position + delta
        newPos = roundVector(newPos, state.snap)
        selectedBlock.CFrame = CFrame.new(newPos, newPos + selectedBlock.CFrame.LookVector)
    end
end

local function rotateSelected(axis, deg)
    if not selectedBlock then return end
    local inc = math.rad(roundTo(deg, state.rotSnap))
    if axis == "X" then
        selectedBlock.CFrame = selectedBlock.CFrame * CFrame.Angles(inc, 0, 0)
    elseif axis == "Y" then
        selectedBlock.CFrame = selectedBlock.CFrame * CFrame.Angles(0, inc, 0)
    elseif axis == "Z" then
        selectedBlock.CFrame = selectedBlock.CFrame * CFrame.Angles(0, 0, inc)
    end
end

local function scaleSelected(delta)
    if not selectedBlock then return end
    local newSize = selectedBlock.Size + delta
    newSize = Vector3.new(
        clamp(roundTo(newSize.X, state.scaleSnap), 0.5, 100),
        clamp(roundTo(newSize.Y, state.scaleSnap), 0.5, 100),
        clamp(roundTo(newSize.Z, state.scaleSnap), 0.5, 100)
    )
    selectedBlock.Size = newSize
end

local function deleteSelected()
    if selectedBlock then
        selectedBlock:Destroy()
        selectedBlock = nil
    end
end

-- ===== Input bindings =====
local layerOffset = 0
local function applyLayerOffset()
    if state.mode == "BUILD" then
        local cf = ghost.CFrame
        ghost.CFrame = cf + lastNormal.Unit * layerOffset
    end
end

local function onPlaceOrSelect(actionName, inputState, inputObject)
    if inputState ~= Enum.UserInputState.Begin then return end
    if state.mode == "BUILD" then
        placeBlockAtGhost()
    elseif state.mode == "EDIT" then
        selectNearestBlock()
    end
end

ContextActionService:BindAction("PlaceOrSelect", onPlaceOrSelect, true, Enum.UserInputType.MouseButton1)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if state.mode == "BUILD" then
        if input.KeyCode == Enum.KeyCode.Q then
            layerOffset = layerOffset - state.snap
            applyLayerOffset()
        elseif input.KeyCode == Enum.KeyCode.E then
            layerOffset = layerOffset + state.snap
            applyLayerOffset()
        elseif input.KeyCode == Enum.KeyCode.R then
            ghost.CFrame = ghost.CFrame * CFrame.Angles(0, math.rad(state.rotSnap), 0)
        elseif input.KeyCode == Enum.KeyCode.T then
            ghost.CFrame = ghost.CFrame * CFrame.Angles(0, -math.rad(state.rotSnap), 0)
        elseif input.KeyCode == Enum.KeyCode.F then
            state.collision = not state.collision
        end
    elseif state.mode == "EDIT" then
        local step = state.snap
        if input.KeyCode == Enum.KeyCode.W then nudgeSelected(Camera.CFrame.LookVector * step) end
        if input.KeyCode == Enum.KeyCode.S then nudgeSelected(-Camera.CFrame.LookVector * step) end
        if input.KeyCode == Enum.KeyCode.A then nudgeSelected(-Camera.CFrame.RightVector * step) end
        if input.KeyCode == Enum.KeyCode.D then nudgeSelected(Camera.CFrame.RightVector * step) end
        if input.KeyCode == Enum.KeyCode.PageUp then nudgeSelected(Vector3.new(0, step, 0)) end
        if input.KeyCode == Enum.KeyCode.PageDown then nudgeSelected(Vector3.new(0, -step, 0)) end
        if input.KeyCode == Enum.KeyCode.Z then rotateSelected("X", state.rotSnap) end
        if input.KeyCode == Enum.KeyCode.X then rotateSelected("Y", state.rotSnap) end
        if input.KeyCode == Enum.KeyCode.C then rotateSelected("Z", state.rotSnap) end
        if input.KeyCode == Enum.KeyCode.Equals or input.KeyCode == Enum.KeyCode.Plus then scaleSelected(Vector3.new(state.scaleSnap, state.scaleSnap, state.scaleSnap)) end
        if input.KeyCode == Enum.KeyCode.Minus then scaleSelected(Vector3.new(-state.scaleSnap, -state.scaleSnap, -state.scaleSnap)) end
        if input.KeyCode == Enum.KeyCode.Delete or input.KeyCode == Enum.KeyCode.Backspace then deleteSelected() end
    end
end)

-- ===== Tabs and content =====
local tabButtons = {}
local function addTab(name)
    local tb = new("TextButton", {
        Parent = tabCol,
        Size = UDim2.new(1, 0, 0, 28),
        BackgroundColor3 = Color3.fromRGB(20,20,20),
        Text = name,
        TextColor3 = Color3.new(1,1,1),
        Font = Enum.Font.Gotham,
        TextSize = 13
    })
    new("UICorner", {Parent = tb, CornerRadius = UDim.new(0,6)})
    tabButtons[name] = tb
    return tb
end

local function setTab(name)
    for k,v in pairs(tabButtons) do
        v.BackgroundColor3 = (k == name) and Color3.fromRGB(40,40,40) or Color3.fromRGB(20,20,20)
    end
    clearContent()
    if name == "BUILD" then
        state.mode = "BUILD"
        addLabel("Build mode")
        addLabel("Left-click: place block at ghost")
        addLabel("Q/E: layer offset | R/T: rotate ghost | F: toggle collision")
        addLabel("Ghost size: " .. string.format("(%.1f, %.1f, %.1f)", state.ghostSize.X, state.ghostSize.Y, state.ghostSize.Z))
        addButton("Place block at ghost", function() placeBlockAtGhost() end)
        addButton("Place gray block at my feet", function() placeBlockAtFeet() end)
        addButton("Cycle ghost size", function()
            local sizes = {Vector3.new(2,1,2), Vector3.new(4,1,4), Vector3.new(6,1,6)}
            local idx = 1
            for i,s in ipairs(sizes) do if s == state.ghostSize then idx = i break end end
            idx = (idx % #sizes) + 1
            state.ghostSize = sizes[idx]
            ghost.Size = state.ghostSize
        end)
        addButton("Toggle collision for new blocks", function() state.collision = not state.collision end)
        addButton("Tilt ghost +", function() ghost.CFrame = ghost.CFrame * CFrame.Angles(math.rad(state.rotSnap), 0, 0) end)
        addButton("Tilt ghost -", function() ghost.CFrame = ghost.CFrame * CFrame.Angles(math.rad(-state.rotSnap), 0, 0) end)

    elseif name == "EDIT" then
        state.mode = "EDIT"
        addLabel("Edit mode")
        addLabel("Left-click: select block under mouse")
        addLabel("WASD: move | PageUp/PageDown: vertical")
        addLabel("Z/X/C: rotate X/Y/Z | +/-: scale | Delete: remove")
        addButton("Select block under mouse", function() selectNearestBlock() end)
        addButton("Move +X", function() nudgeSelected(Vector3.new(state.snap,0,0)) end)
        addButton("Move -X", function() nudgeSelected(Vector3.new(-state.snap,0,0)) end)
        addButton("Move +Y", function() nudgeSelected(Vector3.new(0,state.snap,0)) end)
        addButton("Move -Y", function() nudgeSelected(Vector3.new(0,-state.snap,0)) end)
        addButton("Move +Z", function() nudgeSelected(Vector3.new(0,0,state.snap)) end)
        addButton("Move -Z", function() nudgeSelected(Vector3.new(0,0,-state.snap)) end)
        addButton("Rotate Y +"..state.rotSnap.."°", function() rotateSelected("Y", state.rotSnap) end)
        addButton("Rotate Y -"..state.rotSnap.."°", function() rotateSelected("Y", -state.rotSnap) end)
        addButton("Scale +", function() scaleSelected(Vector3.new(state.scaleSnap, state.scaleSnap, state.scaleSnap)) end)
        addButton("Scale -", function() scaleSelected(Vector3.new(-state.scaleSnap, -state.scaleSnap, -state.scaleSnap)) end)
        addButton("Delete selected", function() deleteSelected() end)

    elseif name == "SETTINGS" then
        addLabel("Settings")
        addButton("Grid snap +0.5", function() state.snap = clamp(state.snap + 0.5, 0.5, 20) end)
        addButton("Grid snap -0.5", function() state.snap = clamp(state.snap - 0.5, 0.5, 20) end)
        addButton("Rotation snap +5°", function() state.rotSnap = clamp(state.rotSnap + 5, 5, 90) end)
        addButton("Rotation snap -5°", function() state.rotSnap = clamp(state.rotSnap - 5, 5, 90) end)
        addButton("Scale snap +0.25", function() state.scaleSnap = clamp(state.scaleSnap + 0.25, 0.1, 10) end)
        addButton("Scale snap -0.25", function() state.scaleSnap = clamp(state.scaleSnap - 0.25, 0.1, 10) end)
        addButton("Clear all client blocks", function()
            for _,p in ipairs(clientFolder:GetChildren()) do if p:IsA("BasePart") then p:Destroy() end end
            selectedBlock = nil
        end)
        addButton("Toggle UI visibility", function() frame.Visible = not frame.Visible end)
    end

    -- update canvas size
    RunService.Heartbeat:Wait()
    if listLayout.AbsoluteContentSize then
        scroll.CanvasSize = UDim2.new(0,0,0, listLayout.AbsoluteContentSize.Y + 12)
    end
end

-- Create tabs
local buildTab = addTab("BUILD")
local editTab = addTab("EDIT")
local settingsTab = addTab("SETTINGS")

buildTab.MouseButton1Click:Connect(function() setTab("BUILD") end)
editTab.MouseButton1Click:Connect(function() setTab("EDIT") end)
settingsTab.MouseButton1Click:Connect(function() setTab("SETTINGS") end)

setTab("BUILD") -- default

-- Keep ghost visible only in BUILD mode
RunService.RenderStepped:Connect(function()
    ghost.Transparency = (state.mode == "BUILD") and 0.6 or 1
    ghost.CanCollide = false
end)

print("[ClientBuildUI] Loaded. You can now stack client-only blocks on top of other client blocks.")
