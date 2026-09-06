-- Services & Variables
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local Settings = {
    Aimbot = false,
    LockOnHold = true,
    WallCheck = true,
    TeamCheck = true,
    Smoothness = 0.2,
    TargetPart = "Head",
    
    -- Triggerbot Settings
    Triggerbot = false,
    TriggerbotTeamCheck = true,
    TriggerbotPart = "Head",
    TriggerbotDelay = 0.05,

    -- FOV Settings
    FOVRadius = 250,
    ShowFOVCircle = false,

    -- Visuals
    ESP = false,
    ShowNames = true,
    ShowHealth = true,
    HighlightESP = false,

    -- Hitbox Expansion
    HitboxExpand = false,
    HitboxSize = 5,
    HeadExpand = false,
    HeadSize = 2,

    -- NEW FEATURES TAB SETTINGS
    SilentAim = false,
    Spinbot = false,
    SpinSpeed = 20,
    SpeedHack = false,
    WalkSpeedValue = 32,
    InfJump = false,

    -- UI Colors
    AccentColor = Color3.fromRGB(0, 229, 255),
    ButtonActiveColor = Color3.fromRGB(0, 140, 200),
    RainbowMode = false,
    RainbowSpeed = 0.5
}

local Aiming = false
local Shooting = false
local OriginalSizes = {}
local OriginalMeshScales = {}
local Connections = {}

-- Dynamic UI Color Registries
local DynamicBorders = {}
local DynamicText = {}
local DynamicFills = {}
local ManagedActiveButtons = {}

local function TrackConnection(conn)
    table.insert(Connections, conn)
    return conn
end

-- FOV Circle Drawing Engine
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1.5
FOVCircle.Color = Settings.AccentColor
FOVCircle.Filled = false
FOVCircle.Transparency = 1
FOVCircle.Visible = false

TrackConnection(RunService.RenderStepped:Connect(function()
    if Settings.ShowFOVCircle then
        local MousePos = UserInputService:GetMouseLocation()
        FOVCircle.Position = MousePos
        FOVCircle.Radius = Settings.FOVRadius
        FOVCircle.Color = Settings.AccentColor
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end))

-- GUI Setup (Custom Styled: "PHANTOM // NEXUS")
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PhantomNexusUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 999999999
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 540, 0, 500)
MainFrame.Position = UDim2.new(0.5, -270, 0.5, -250)
MainFrame.BackgroundColor3 = Color3.fromRGB(11, 11, 16)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

-- Invisible Modal Enforcer
local ModalEnforcer = Instance.new("TextButton")
ModalEnforcer.Name = "ModalEnforcer"
ModalEnforcer.Size = UDim2.new(0, 0, 0, 0)
ModalEnforcer.Visible = true
ModalEnforcer.Modal = true
ModalEnforcer.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Settings.AccentColor
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame
table.insert(DynamicBorders, UIStroke)

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

-- Tech Background Grid Effect Decorator
local GridAccent = Instance.new("Frame")
GridAccent.Name = "GridAccent"
GridAccent.Size = UDim2.new(1, 0, 1, 0)
GridAccent.BackgroundTransparency = 0.96
GridAccent.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
GridAccent.BorderSizePixel = 0
GridAccent.ZIndex = 0
GridAccent.Parent = MainFrame

-- Header
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, 0, 0, 42)
Header.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 8)
HeaderCorner.Parent = Header

local HeaderSquareFix = Instance.new("Frame")
HeaderSquareFix.Size = UDim2.new(1, 0, 0, 8)
HeaderSquareFix.Position = UDim2.new(0, 0, 1, -8)
HeaderSquareFix.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
HeaderSquareFix.BorderSizePixel = 0
HeaderSquareFix.Parent = Header

local HeaderBar = Instance.new("Frame")
HeaderBar.Name = "HeaderBar"
HeaderBar.Size = UDim2.new(1, 0, 0, 2)
HeaderBar.Position = UDim2.new(0, 0, 1, 0)
HeaderBar.BackgroundColor3 = Settings.AccentColor
HeaderBar.BorderSizePixel = 0
HeaderBar.Parent = Header
table.insert(DynamicFills, {Type = "Background", Element = HeaderBar})

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -16, 1, 0)
Title.Position = UDim2.new(0, 12, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "⚡ PHANTOM // NEXUS v2.8 [SECURE]"
Title.TextColor3 = Settings.AccentColor
Title.TextSize = 12
Title.Font = Enum.Font.Code
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Header
table.insert(DynamicText, Title)

local StatusIndicator = Instance.new("TextLabel")
StatusIndicator.Size = UDim2.new(0, 100, 1, 0)
StatusIndicator.Position = UDim2.new(1, -110, 0, 0)
StatusIndicator.BackgroundTransparency = 1
StatusIndicator.Text = "● ACTIVE"
StatusIndicator.TextColor3 = Color3.fromRGB(40, 240, 120)
StatusIndicator.TextSize = 10
StatusIndicator.Font = Enum.Font.Code
StatusIndicator.TextXAlignment = Enum.TextXAlignment.Right
StatusIndicator.Parent = Header

-- RightShift Toggle Keybind
TrackConnection(UserInputService.InputBegan:Connect(function(input, gpe)
    if input.KeyCode == Enum.KeyCode.RightShift then
        MainFrame.Visible = not MainFrame.Visible
    end
end))

-- Dragging Logic
local Dragging, DragInput, DragStart, StartPos

TrackConnection(Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        Dragging = true
        DragStart = input.Position
        StartPos = MainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                Dragging = false
            end
        end)
    end
end))

TrackConnection(Header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        DragInput = input
    end
end))

TrackConnection(UserInputService.InputChanged:Connect(function(input)
    if input == DragInput and Dragging then
        local Delta = input.Position - DragStart
        MainFrame.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset + Delta.X, StartPos.Y.Scale, StartPos.Y.Offset + Delta.Y)
    end
end))

-- Tab Navigation Container
local TabBar = Instance.new("Frame")
TabBar.Name = "TabBar"
TabBar.Size = UDim2.new(1, -16, 0, 32)
TabBar.Position = UDim2.new(0, 8, 0, 52)
TabBar.BackgroundTransparency = 1
TabBar.Parent = MainFrame

local TabListLayout = Instance.new("UIListLayout")
TabListLayout.FillDirection = Enum.FillDirection.Horizontal
TabListLayout.SortOrder = Enum.SortOrder.LayoutOrder
TabListLayout.Padding = UDim.new(0, 4)
TabListLayout.Parent = TabBar

-- Content Container
local ContentContainer = Instance.new("Frame")
ContentContainer.Name = "ContentContainer"
ContentContainer.Size = UDim2.new(1, -16, 1, -96)
ContentContainer.Position = UDim2.new(0, 8, 0, 88)
ContentContainer.BackgroundTransparency = 1
ContentContainer.Parent = MainFrame

local Tabs = {}
local TabButtons = {}

local function CreateTab(tabName)
    local tabBtn = Instance.new("TextButton")
    tabBtn.Size = UDim2.new(0.155, 0, 1, 0)
    tabBtn.BackgroundColor3 = Color3.fromRGB(16, 16, 24)
    tabBtn.Text = tabName
    tabBtn.TextColor3 = Color3.fromRGB(140, 140, 150)
    tabBtn.Font = Enum.Font.Code
    tabBtn.TextSize = 10
    tabBtn.Parent = TabBar

    local tabBtnCorner = Instance.new("UICorner")
    tabBtnCorner.CornerRadius = UDim.new(0, 6)
    tabBtnCorner.Parent = tabBtn

    local scrollPage = Instance.new("ScrollingFrame")
    scrollPage.Name = tabName .. "Page"
    scrollPage.Size = UDim2.new(1, 0, 1, 0)
    scrollPage.BackgroundTransparency = 1
    scrollPage.BorderSizePixel = 0
    scrollPage.ScrollBarThickness = 3
    scrollPage.ScrollBarImageColor3 = Settings.AccentColor
    scrollPage.Visible = false
    scrollPage.Parent = ContentContainer
    table.insert(DynamicFills, {Type = "ScrollBar", Element = scrollPage})

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Padding = UDim.new(0, 6)
    pageLayout.Parent = scrollPage

    Tabs[tabName] = scrollPage
    TabButtons[tabName] = tabBtn

    TrackConnection(tabBtn.MouseButton1Click:Connect(function()
        for name, page in pairs(Tabs) do
            page.Visible = (name == tabName)
            TabButtons[name].BackgroundColor3 = (name == tabName) and Settings.ButtonActiveColor or Color3.fromRGB(16, 16, 24)
            TabButtons[name].TextColor3 = (name == tabName) and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(140, 140, 150)
        end
    end))

    return scrollPage
end

local function CreateToggleButton(parentPage, text, defaultState, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -6, 0, 30)
    btn.BackgroundColor3 = defaultState and Settings.ButtonActiveColor or Color3.fromRGB(20, 20, 28)
    btn.Text = "   [ " .. (defaultState and "ENABLED" or "DISABLED") .. " ]  " .. text
    btn.TextColor3 = defaultState and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(170, 170, 180)
    btn.Font = Enum.Font.Code
    btn.TextSize = 11
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Parent = parentPage

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    local state = defaultState
    local btnObj = {State = function() return state end, Button = btn}
    table.insert(ManagedActiveButtons, btnObj)

    TrackConnection(btn.MouseButton1Click:Connect(function()
        state = not state
        btn.BackgroundColor3 = state and Settings.ButtonActiveColor or Color3.fromRGB(20, 20, 28)
        btn.Text = "   [ " .. (state and "ENABLED" or "DISABLED") .. " ]  " .. text
        btn.TextColor3 = state and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(170, 170, 180)
        callback(state)
    end))
end

local function CreateSlider(parentPage, text, min, max, default, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -6, 0, 46)
    frame.BackgroundColor3 = Color3.fromRGB(18, 18, 26)
    frame.Parent = parentPage

    local frameCorner = Instance.new("UICorner")
    frameCorner.CornerRadius = UDim.new(0, 6)
    frameCorner.Parent = frame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -12, 0, 18)
    label.Position = UDim2.new(0, 6, 0, 4)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. tostring(default)
    label.TextColor3 = Color3.fromRGB(210, 210, 220)
    label.Font = Enum.Font.Code
    label.TextSize = 11
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local sliderBack = Instance.new("Frame")
    sliderBack.Size = UDim2.new(1, -12, 0, 10)
    sliderBack.Position = UDim2.new(0, 6, 0, 26)
    sliderBack.BackgroundColor3 = Color3.fromRGB(30, 30, 42)
    sliderBack.Parent = frame

    local backCorner = Instance.new("UICorner")
    backCorner.CornerRadius = UDim.new(0, 4)
    backCorner.Parent = sliderBack

    local fill = Instance.new("Frame")
    local startPercent = (default - min) / (max - min)
    fill.Size = UDim2.new(startPercent, 0, 1, 0)
    fill.BackgroundColor3 = Settings.AccentColor
    fill.Parent = sliderBack
    table.insert(DynamicFills, {Type = "Background", Element = fill})

    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(0, 4)
    fillCorner.Parent = fill

    local sliding = false
    local function UpdateInput(input)
        local pos = math.clamp((input.Position.X - sliderBack.AbsolutePosition.X) / sliderBack.AbsoluteSize.X, 0, 1)
        fill.Size = UDim2.new(pos, 0, 1, 0)
        local val = math.floor((min + (max - min) * pos) * 100) / 100
        label.Text = text .. ": " .. tostring(val)
        callback(val)
    end

    TrackConnection(sliderBack.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            sliding = true
            UpdateInput(input)
        end
    end))

    TrackConnection(UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            sliding = false
        end
    end))

    TrackConnection(UserInputService.InputChanged:Connect(function(input)
        if sliding and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
            UpdateInput(input)
        end
    end))
end

local function CreateActionBtn(parentPage, text, bgColor, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -6, 0, 30)
    btn.BackgroundColor3 = bgColor
    btn.Text = "   " .. text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Font = Enum.Font.Code
    btn.TextSize = 11
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Parent = parentPage

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    TrackConnection(btn.MouseButton1Click:Connect(function()
        callback(btn)
    end))
    return btn
end

-- Pages Initialization
local AimbotPage = CreateTab("Aimbot")
local TriggerbotPage = CreateTab("Triggerbot")
local VisualsPage = CreateTab("Visuals")
local HitboxPage = CreateTab("Hitbox")
local NewFeaturesPage = CreateTab("New Features")
local UISettingsPage = CreateTab("UI Settings")

Tabs["Aimbot"].Visible = true
TabButtons["Aimbot"].BackgroundColor3 = Settings.ButtonActiveColor
TabButtons["Aimbot"].TextColor3 = Color3.fromRGB(255, 255, 255)

-- Theme Update Engine
local function ApplyColorToUI(color)
    Settings.AccentColor = color
    for _, stroke in ipairs(DynamicBorders) do
        stroke.Color = color
    end
    for _, label in ipairs(DynamicText) do
        label.TextColor3 = color
    end
    for _, fill in ipairs(DynamicFills) do
        if fill.Type == "ScrollBar" then
            fill.Element.ScrollBarImageColor3 = color
        else
            fill.Element.BackgroundColor3 = color
        end
    end
end

local function UpdateButtonActiveTheme(color)
    Settings.ButtonActiveColor = color
    for _, item in ipairs(ManagedActiveButtons) do
        if item.State() then
            item.Button.BackgroundColor3 = color
        end
    end
    for name, page in pairs(Tabs) do
        if page.Visible then
            TabButtons[name].BackgroundColor3 = color
        end
    end
end

-- Active Rainbow Loop Engine
local HueCounter = 0
TrackConnection(RunService.RenderStepped:Connect(function(delta)
    if Settings.RainbowMode then
        HueCounter = (HueCounter + delta * Settings.RainbowSpeed) % 1
        local RainbowColor = Color3.fromHSV(HueCounter, 0.9, 1)
        ApplyColorToUI(RainbowColor)
    end
end))

-- Aimbot Tab Controls
CreateToggleButton(AimbotPage, "Aimbot Engine", Settings.Aimbot, function(v) Settings.Aimbot = v end)
CreateToggleButton(AimbotPage, "Lock on RMB", Settings.LockOnHold, function(v) Settings.LockOnHold = v end)

CreateActionBtn(AimbotPage, "[ TARGET PART ] Currently: HEAD", Color3.fromRGB(22, 22, 32), function(btn)
    if Settings.TargetPart == "Head" then
        Settings.TargetPart = "Torso"
        btn.Text = "   [ TARGET PART ] Currently: TORSO"
    else
        Settings.TargetPart = "Head"
        btn.Text = "   [ TARGET PART ] Currently: HEAD"
    end
end)

CreateToggleButton(AimbotPage, "Team Check", Settings.TeamCheck, function(v) Settings.TeamCheck = v end)
CreateToggleButton(AimbotPage, "Wall Check", Settings.WallCheck, function(v) Settings.WallCheck = v end)
CreateToggleButton(AimbotPage, "Show FOV Circle", Settings.ShowFOVCircle, function(v) Settings.ShowFOVCircle = v end)

CreateSlider(AimbotPage, "Lock Smoothness (Low -> High)", 0.01, 1.0, Settings.Smoothness, function(v)
    Settings.Smoothness = v
end)

CreateSlider(AimbotPage, "FOV Radius Size", 50, 500, Settings.FOVRadius, function(v)
    Settings.FOVRadius = v
end)

-- Triggerbot Tab Controls
CreateToggleButton(TriggerbotPage, "Triggerbot Engine", Settings.Triggerbot, function(v) Settings.Triggerbot = v end)
CreateToggleButton(TriggerbotPage, "Team Check", Settings.TriggerbotTeamCheck, function(v) Settings.TriggerbotTeamCheck = v end)

CreateActionBtn(TriggerbotPage, "[ TARGET PART ] Currently: HEAD", Color3.fromRGB(22, 22, 32), function(btn)
    if Settings.TriggerbotPart == "Head" then
        Settings.TriggerbotPart = "Torso"
        btn.Text = "   [ TARGET PART ] Currently: TORSO"
    elseif Settings.TriggerbotPart == "Torso" then
        Settings.TriggerbotPart = "Any"
        btn.Text = "   [ TARGET PART ] Currently: ANY"
    else
        Settings.TriggerbotPart = "Head"
        btn.Text = "   [ TARGET PART ] Currently: HEAD"
    end
end)

CreateSlider(TriggerbotPage, "Shot Delay (Seconds)", 0.0, 0.5, Settings.TriggerbotDelay, function(v)
    Settings.TriggerbotDelay = v
end)

-- Visuals Tab Controls
CreateToggleButton(VisualsPage, "ESP Overlay", Settings.ESP, function(v)
    Settings.ESP = v
    if not v then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Hypershot_ESP_Billboard") then
                plr.Character.Hypershot_ESP_Billboard:Destroy()
            end
        end
    end
end)
CreateToggleButton(VisualsPage, "Highlight All Players", Settings.HighlightESP, function(v)
    Settings.HighlightESP = v
    if not v then
        for _, plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Hypershot_ESP_Highlight") then
                plr.Character.Hypershot_ESP_Highlight:Destroy()
            end
        end
    end
end)
CreateToggleButton(VisualsPage, "Show Names", Settings.ShowNames, function(v) Settings.ShowNames = v end)
CreateToggleButton(VisualsPage, "Show Health", Settings.ShowHealth, function(v) Settings.ShowHealth = v end)

-- Hitbox Tab Controls
CreateToggleButton(HitboxPage, "Expand Body Hitbox", Settings.HitboxExpand, function(v) Settings.HitboxExpand = v end)
CreateSlider(HitboxPage, "Body Hitbox Size", 1, 20, Settings.HitboxSize, function(v)
    Settings.HitboxSize = v
end)

CreateToggleButton(HitboxPage, "Expand Head Hitbox", Settings.HeadExpand, function(v) Settings.HeadExpand = v end)
CreateSlider(HitboxPage, "Head Hitbox Scale", 1, 10, Settings.HeadSize, function(v)
    Settings.HeadSize = v
end)

-- NEW FEATURES TAB CONTROLS
CreateToggleButton(NewFeaturesPage, "Silent Aim (Camera Snap On Shot)", Settings.SilentAim, function(v)
    Settings.SilentAim = v
end)

CreateToggleButton(NewFeaturesPage, "Spinbot Anti-Aim", Settings.Spinbot, function(v)
    Settings.Spinbot = v
end)

CreateSlider(NewFeaturesPage, "Spin Speed", 5, 100, Settings.SpinSpeed, function(v)
    Settings.SpinSpeed = v
end)

CreateToggleButton(NewFeaturesPage, "Speed Hack", Settings.SpeedHack, function(v)
    Settings.SpeedHack = v
    if not v and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
    end
end)

CreateSlider(NewFeaturesPage, "WalkSpeed Value", 16, 120, Settings.WalkSpeedValue, function(v)
    Settings.WalkSpeedValue = v
end)

CreateToggleButton(NewFeaturesPage, "Infinite Jump", Settings.InfJump, function(v)
    Settings.InfJump = v
end)

-- UI Settings Tab Controls
CreateToggleButton(UISettingsPage, "Rainbow UI Theme", Settings.RainbowMode, function(v)
    Settings.RainbowMode = v
end)

CreateActionBtn(UISettingsPage, "UI Accent: Cyber Cyan", Color3.fromRGB(0, 140, 200), function()
    Settings.RainbowMode = false
    ApplyColorToUI(Color3.fromRGB(0, 229, 255))
end)
CreateActionBtn(UISettingsPage, "UI Accent: Matrix Green", Color3.fromRGB(20, 140, 60), function()
    Settings.RainbowMode = false
    ApplyColorToUI(Color3.fromRGB(40, 255, 110))
end)
CreateActionBtn(UISettingsPage, "UI Accent: Plasma Purple", Color3.fromRGB(110, 30, 160), function()
    Settings.RainbowMode = false
    ApplyColorToUI(Color3.fromRGB(180, 70, 255))
end)
CreateActionBtn(UISettingsPage, "UI Accent: Laser Crimson", Color3.fromRGB(160, 30, 45), function()
    Settings.RainbowMode = false
    ApplyColorToUI(Color3.fromRGB(255, 55, 80))
end)
CreateActionBtn(UISettingsPage, "UI Accent: Solar Amber", Color3.fromRGB(170, 100, 10), function()
    Settings.RainbowMode = false
    ApplyColorToUI(Color3.fromRGB(255, 170, 30))
end)

CreateActionBtn(UISettingsPage, "Active Button: Deep Cyan", Color3.fromRGB(0, 100, 150), function()
    UpdateButtonActiveTheme(Color3.fromRGB(0, 140, 200))
end)
CreateActionBtn(UISettingsPage, "Active Button: Forest Green", Color3.fromRGB(15, 100, 50), function()
    UpdateButtonActiveTheme(Color3.fromRGB(20, 140, 70))
end)
CreateActionBtn(UISettingsPage, "Active Button: Deep Violet", Color3.fromRGB(80, 25, 120), function()
    UpdateButtonActiveTheme(Color3.fromRGB(110, 40, 160))
end)
CreateActionBtn(UISettingsPage, "Active Button: Dark Orange", Color3.fromRGB(140, 60, 15), function()
    UpdateButtonActiveTheme(Color3.fromRGB(175, 75, 20))
end)

local function UnloadScript()
    pcall(function() FOVCircle:Remove() end)
    for _, conn in ipairs(Connections) do
        if conn then conn:Disconnect() end
    end
    pcall(function() RunService:UnbindFromRenderStep("HypershotMouseMoverelLock") end)

    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Character then
            if plr.Character:FindFirstChild("Hypershot_ESP_Billboard") then
                plr.Character.Hypershot_ESP_Billboard:Destroy()
            end
            if plr.Character:FindFirstChild("Hypershot_ESP_Highlight") then
                plr.Character.Hypershot_ESP_Highlight:Destroy()
            end
        end
    end

    for part, originalSize in pairs(OriginalSizes) do
        if part and part.Parent then
            part.Size = originalSize
            if part.Name == "HumanoidRootPart" or part.Name == "Torso" or part.Name == "UpperTorso" then part.Transparency = 1 end
        end
    end

    for mesh, originalScale in pairs(OriginalMeshScales) do
        if mesh and mesh.Parent then mesh.Scale = originalScale end
    end

    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = 16
    end

    ScreenGui:Destroy()
end

CreateActionBtn(UISettingsPage, "[ UNLOAD ] Terminate Script", Color3.fromRGB(160, 30, 30), UnloadScript)

-- Core Gameplay & Targeting Loops
TrackConnection(UserInputService.InputBegan:Connect(function(input, gpe)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then Aiming = true end
    if Settings.InfJump and input.KeyCode == Enum.KeyCode.Space and not gpe then
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end))

TrackConnection(UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then Aiming = false end
end))

local function IsEnemy(player)
    if not Settings.TeamCheck then return true end
    if player.Team and LocalPlayer.Team then return player.Team ~= LocalPlayer.Team end
    return true
end

local function IsVisible(TargetPart)
    if not Settings.WallCheck then return true end
    local IgnoreList = {Camera}
    if LocalPlayer.Character then table.insert(IgnoreList, LocalPlayer.Character) end

    local RaycastParams = RaycastParams.new()
    RaycastParams.FilterType = Enum.RaycastFilterType.Exclude
    RaycastParams.FilterDescendantsInstances = IgnoreList
    RaycastParams.IgnoreWater = true

    local Direction = TargetPart.Position - Camera.CFrame.Position
    local Result = workspace:Raycast(Camera.CFrame.Position, Direction, RaycastParams)
    return Result == nil or Result.Instance:IsDescendantOf(TargetPart.Parent)
end

local function GetClosestTarget()
    local MousePos = UserInputService:GetMouseLocation()
    local ClosestPart = nil
    local ShortestDistance = Settings.FOVRadius

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsEnemy(player) and player.Character then
            local char = player.Character
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local TargetPart = nil

                if Settings.TargetPart == "Torso" then
                    TargetPart = char:FindFirstChild("HumanoidRootPart") 
                        or char:FindFirstChild("UpperTorso") 
                        or char:FindFirstChild("Torso")
                else
                    TargetPart = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
                end

                if TargetPart then
                    local ScreenPos, OnScreen = Camera:WorldToViewportPoint(TargetPart.Position)
                    if OnScreen then
                        local Dist = (Vector2.new(ScreenPos.X, ScreenPos.Y) - MousePos).Magnitude
                        if Dist < ShortestDistance and IsVisible(TargetPart) then
                            ShortestDistance = Dist
                            ClosestPart = TargetPart
                        end
                    end
                end
            end
        end
    end
    return ClosestPart
end

-- Triggerbot Raycast Function
local function GetTargetUnderCrosshair()
    local MouseLocation = UserInputService:GetMouseLocation()
    local UnitRay = Camera:ViewportPointToRay(MouseLocation.X, MouseLocation.Y)
    
    local RaycastParams = RaycastParams.new()
    RaycastParams.FilterType = Enum.RaycastFilterType.Exclude
    
    if LocalPlayer.Character then
        RaycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    end

    local Result = workspace:Raycast(UnitRay.Origin, UnitRay.Direction * 1000, RaycastParams)

    if Result and Result.Instance then
        local HitPart = Result.Instance
        local Model = HitPart:FindFirstAncestorOfClass("Model")
        
        if Model then
            local TargetPlayer = Players:GetPlayerFromCharacter(Model)
            if TargetPlayer and TargetPlayer ~= LocalPlayer then
                if Settings.TriggerbotTeamCheck and TargetPlayer.Team and LocalPlayer.Team and TargetPlayer.Team == LocalPlayer.Team then
                    return nil
                end
                
                local Humanoid = Model:FindFirstChildOfClass("Humanoid")
                if Humanoid and Humanoid.Health > 0 then
                    if Settings.TriggerbotPart == "Head" then
                        if HitPart.Name == "Head" then return TargetPlayer end
                    elseif Settings.TriggerbotPart == "Torso" then
                        if HitPart.Name == "UpperTorso" or HitPart.Name == "Torso" or HitPart.Name == "HumanoidRootPart" then return TargetPlayer end
                    elseif Settings.TriggerbotPart == "Any" then
                        return TargetPlayer
                    end
                end
            end
        end
    end
    return nil
end

-- Mouse Cursor Tracking Lock (using mousemoverel)
RunService:BindToRenderStep("HypershotMouseMoverelLock", Enum.RenderPriority.Camera.Value + 1, function()
    if Settings.Aimbot and (not Settings.LockOnHold or Aiming) then
        local TargetPart = GetClosestTarget()
        if TargetPart then
            local ScreenPos, OnScreen = Camera:WorldToViewportPoint(TargetPart.Position)
            if OnScreen then
                local MousePos = UserInputService:GetMouseLocation()
                local DeltaX = ScreenPos.X - MousePos.X
                local DeltaY = ScreenPos.Y - MousePos.Y
                
                if mousemoverel then
                    local Scale = math.clamp(Settings.Smoothness, 0.01, 1.0)
                    mousemoverel(DeltaX * Scale, DeltaY * Scale)
                end
            end
        end
    end
end)

-- Triggerbot Execution Loop
TrackConnection(RunService.RenderStepped:Connect(function()
    if not Settings.Triggerbot then return end

    local Target = GetTargetUnderCrosshair()
    
    if Target and not Shooting then
        Shooting = true
        task.spawn(function()
            if Settings.TriggerbotDelay > 0 then task.wait(Settings.TriggerbotDelay) end
            
            if Settings.Triggerbot and GetTargetUnderCrosshair() == Target then
                -- Silent Aim: Snap camera directly to target before shooting
                if Settings.SilentAim then
                    local char = Target.Character
                    if char then
                        local part = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
                        if part then Camera.CFrame = CFrame.new(Camera.CFrame.Position, part.Position) end
                    end
                end

                if mouse1press then
                    mouse1press()
                    task.wait(0.05)
                    mouse1release()
                elseif mouse1click then
                    mouse1click()
                end
            end
            
            Shooting = false
        end)
    end
end))

-- New Features Handlers (Spinbot & Speed Hack)
TrackConnection(RunService.RenderStepped:Connect(function()
    -- Spinbot Engine
    if Settings.Spinbot and LocalPlayer.Character then
        local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if root then
            root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(Settings.SpinSpeed), 0)
        end
    end

    -- Speed Hack Engine
    if Settings.SpeedHack and LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = Settings.WalkSpeedValue
        end
    end
end))

-- Unified Heartbeat Loop for Hitboxes and Visuals
TrackConnection(RunService.Heartbeat:Connect(function()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local char = player.Character
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            local isAlive = not humanoid or humanoid.Health > 0
            local enemyCheck = IsEnemy(player)

            local head = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
            local root = char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")

            -- Body Hitbox Expansion
            if root then
                if not OriginalSizes[root] then OriginalSizes[root] = root.Size end
                if Settings.HitboxExpand and isAlive and enemyCheck then
                    root.Size = Vector3.new(Settings.HitboxSize, Settings.HitboxSize, Settings.HitboxSize)
                    root.Transparency = 0.5
                    root.CanCollide = false
                else
                    root.Size = OriginalSizes[root]
                    root.Transparency = 1
                end
            end

            -- Head Hitbox Expansion
            if head then
                if not OriginalSizes[head] then OriginalSizes[head] = head.Size end
                if Settings.HeadExpand and isAlive and enemyCheck then
                    head.Size = OriginalSizes[head] * Settings.HeadSize
                    head.CanCollide = false
                else
                    head.Size = OriginalSizes[head]
                end

                for _, child in ipairs(head:GetDescendants()) do
                    if child:IsA("SpecialMesh") or child:IsA("BlockMesh") or child:IsA("CylinderMesh") then
                        if not OriginalMeshScales[child] then OriginalMeshScales[child] = child.Scale end
                        if Settings.HeadExpand and isAlive and enemyCheck then
                            child.Scale = OriginalMeshScales[child] * Settings.HeadSize
                        else
                            child.Scale = OriginalMeshScales[child]
                        end
                    end
                end
            end

            -- ESP Text Billboard
            local textAdornee = head or root
            if Settings.ESP and isAlive and enemyCheck and textAdornee then
                local billboard = char:FindFirstChild("Hypershot_ESP_Billboard")
                if not billboard then
                    billboard = Instance.new("BillboardGui")
                    billboard.Name = "Hypershot_ESP_Billboard"
                    billboard.Adornee = textAdornee
                    billboard.Size = UDim2.new(0, 150, 0, 40)
                    billboard.StudsOffset = Vector3.new(0, 2, 0)
                    billboard.AlwaysOnTop = true

                    local textLabel = Instance.new("TextLabel")
                    textLabel.Name = "InfoLabel"
                    textLabel.Size = UDim2.new(1, 0, 1, 0)
                    textLabel.BackgroundTransparency = 1
                    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    textLabel.TextStrokeTransparency = 0
                    textLabel.TextSize = 12
                    textLabel.TextXAlignment = Enum.TextXAlignment.Center
                    textLabel.Font = Enum.Font.Code
                    textLabel.Parent = billboard
                    billboard.Parent = char
                end

                local label = billboard:FindFirstChild("InfoLabel")
                if label then
                    local displayText = Settings.ShowNames and player.Name or ""
                    if Settings.ShowHealth and humanoid then
                        local hpText = "[" .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth) .. " HP]"
                        displayText = (displayText ~= "" and displayText .. " " or "") .. hpText
                    end
                    label.Text = displayText
                    billboard.Enabled = true
                end
            else
                if char:FindFirstChild("Hypershot_ESP_Billboard") then
                    char.Hypershot_ESP_Billboard.Enabled = false
                end
            end

            -- Highlight ESP
            if Settings.HighlightESP and isAlive and enemyCheck then
                local highlight = char:FindFirstChild("Hypershot_ESP_Highlight")
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "Hypershot_ESP_Highlight"
                    highlight.Adornee = char
                    highlight.FillColor = Settings.AccentColor
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineTransparency = 0
                    highlight.Parent = char
                else
                    highlight.FillColor = Settings.AccentColor
                    highlight.Enabled = true
                end
            else
                if char:FindFirstChild("Hypershot_ESP_Highlight") then
                    char.Hypershot_ESP_Highlight.Enabled = false
                end
            end
        end
    end
end))
