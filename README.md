local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local CoreGui          = game:GetService("CoreGui")
local TextChatService  = game:GetService("TextChatService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local LocalPlayer  = Players.LocalPlayer
local Mouse        = LocalPlayer:GetMouse()


-- // [2] BASIC SETUP WITH ERROR PROTECTION // --

-- Primary UCT loader (GitHub raw)
local ok1, err1 = pcall(function()
    loadstring(game:HttpGet(
        "https://raw.githubusercontent.com/UCT-hub/main/refs/heads/main/stealabrainrot",
        true
    ))()
end)
if not ok1 then warn("[UCT_Hub] Primary loader failed: " .. tostring(err1)) end

-- Luarmor backup loader
local ok2, err2 = pcall(function()
    loadstring(game:HttpGet(
        "https://api.luarmor.net/files/v4/loaders/9ba31590e44fa3bd648373677627a146.lua",
        true
    ))()
end)
if not ok2 then warn("[UCT_Hub] Luarmor loader failed: " .. tostring(err2)) end

-- Dragging state
local dragging  = false
local dragStart = nil
local startPos  = nil

-- Toggle state table  â€“  key = feature name, value = bool
local toggleStates = {}

-- Active tab tracking
local activeTab = "Main"

-- GUI scale (100% default, matches "GUI Size: 100%" in trace)
local guiScale = 1


-- // [3] CORE GUI CREATION WITH FALLBACK // --

-- Remove any existing instance to prevent duplicates on re-exec
local existing = CoreGui:FindFirstChild("UCT_Hub")
if existing then existing:Destroy() end

-- â”€â”€ ScreenGui â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name          = "UCT_Hub"
ScreenGui.ResetOnSpawn  = false
ScreenGui.DisplayOrder  = 100
ScreenGui.Parent        = CoreGui

-- â”€â”€ MainFrame  (outer window) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local MainFrame = Instance.new("Frame")
MainFrame.Name                 = "Frame"
MainFrame.Size                 = UDim2.new(0, 260, 0, 380)
MainFrame.Position             = UDim2.new(1, -270, 0, 20)
MainFrame.BackgroundColor3     = Color3.fromRGB(14, 14, 14)   -- 0.054902 * 255 â‰ˆ 14
MainFrame.BorderSizePixel      = 0
MainFrame.Parent               = ScreenGui

local MainScale = Instance.new("UIScale")
MainScale.Scale  = guiScale
MainScale.Parent = MainFrame

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent       = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color     = Color3.fromRGB(255, 40, 40)   -- 1, 0.156863, 0.156863
MainStroke.Thickness = 1.4
MainStroke.Parent    = MainFrame

-- â”€â”€ HeaderFrame  (title bar) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local HeaderFrame = Instance.new("Frame")
HeaderFrame.Name             = "Frame"
HeaderFrame.Size             = UDim2.new(1, 0, 0, 34)
HeaderFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)  -- 0.0784314 * 255 â‰ˆ 20
HeaderFrame.BorderSizePixel  = 0
HeaderFrame.Parent           = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 10)
HeaderCorner.Parent       = HeaderFrame

-- Logo icon
local LogoIcon = Instance.new("ImageLabel")
LogoIcon.Size                 = UDim2.new(0, 22, 0, 22)
LogoIcon.Position             = UDim2.new(0, 6, 0, 6)
LogoIcon.BackgroundTransparency = 1
LogoIcon.Image                = "rbxassetid://76851004884846"
LogoIcon.Parent               = HeaderFrame

-- Title + sub-label container
local TitleContainer = Instance.new("Frame")
TitleContainer.Size                 = UDim2.new(0, 165, 1, 0)
TitleContainer.Position             = UDim2.new(0, 32, 0, 0)
TitleContainer.BackgroundTransparency = 1
TitleContainer.Parent               = HeaderFrame

local TitleLayout = Instance.new("UIListLayout")
TitleLayout.FillDirection      = Enum.FillDirection.Horizontal
TitleLayout.VerticalAlignment  = Enum.VerticalAlignment.Center
TitleLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
TitleLayout.Padding            = UDim.new(0, 3)
TitleLayout.Parent             = TitleContainer

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size                 = UDim2.new(0, 80, 1, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text                 = "discord.gg/uct"
TitleLabel.TextColor3           = Color3.new(1, 1, 1)
TitleLabel.Font                 = Enum.Font.GothamBold
TitleLabel.TextSize             = 14
TitleLabel.TextXAlignment       = Enum.TextXAlignment.Left
TitleLabel.Parent               = TitleContainer

local SubLabel = Instance.new("TextLabel")
SubLabel.Size                 = UDim2.new(0, 100, 1, 0)
SubLabel.BackgroundTransparency = 1
SubLabel.Text                 = ""   -- version / status text (empty in trace)
SubLabel.TextColor3           = Color3.fromRGB(200, 60, 60)  -- 0.784314, 0.235294, 0.235294
SubLabel.Font                 = Enum.Font.Gotham
SubLabel.TextSize             = 10
SubLabel.Parent               = TitleContainer

-- Minimise  "-"  button
local MinimiseButton = Instance.new("TextButton")
MinimiseButton.Name             = "MinimiseButton"
MinimiseButton.Size             = UDim2.new(0, 26, 0, 22)
MinimiseButton.Position         = UDim2.new(1, -32, 0, 6)
MinimiseButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MinimiseButton.Text             = "-"
MinimiseButton.TextColor3       = Color3.fromRGB(255, 40, 40)
MinimiseButton.Font             = Enum.Font.GothamBold
MinimiseButton.TextSize         = 20
MinimiseButton.BorderSizePixel  = 0
MinimiseButton.Parent           = HeaderFrame

local MinCorner = Instance.new("UICorner")
MinCorner.CornerRadius = UDim.new(0, 6)
MinCorner.Parent       = MinimiseButton

-- â”€â”€ Tab bar â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local TabBar = Instance.new("Frame")
TabBar.Name             = "TabBar"
TabBar.Size             = UDim2.new(1, 0, 0, 24)
TabBar.Position         = UDim2.new(0, 0, 0, 34)
TabBar.BackgroundColor3 = Color3.fromRGB(16, 16, 16)
TabBar.BorderSizePixel  = 0
TabBar.Parent           = MainFrame

local TabBarLayout = Instance.new("UIListLayout")
TabBarLayout.FillDirection      = Enum.FillDirection.Horizontal
TabBarLayout.VerticalAlignment  = Enum.VerticalAlignment.Center
TabBarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
TabBarLayout.Padding            = UDim.new(0, 3)
TabBarLayout.Parent             = TabBar

-- Helper: create a tab button
local function makeTabButton(label)
    local btn = Instance.new("TextButton")
    btn.Name             = label .. "Tab"
    btn.Size             = UDim2.new(0, 58, 0, 20)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    btn.Text             = label
    btn.TextColor3       = Color3.fromRGB(200, 200, 200)
    btn.Font             = Enum.Font.Gotham
    btn.TextSize         = 10
    btn.BorderSizePixel  = 0
    btn.Parent           = TabBar
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, 4)
    c.Parent = btn
    return btn
end

local TabMain    = makeTabButton("Main")
local TabPlayer  = makeTabButton("Player")
local TabStealer = makeTabButton("Stealer")
local TabUtility = makeTabButton("Utility")

-- â”€â”€ Content area (ScrollingFrame) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local ContentFrame = Instance.new("Frame")
ContentFrame.Name             = "ContentFrame"
ContentFrame.Size             = UDim2.new(1, 0, 1, -58)
ContentFrame.Position         = UDim2.new(0, 0, 0, 58)
ContentFrame.BackgroundTransparency = 1
ContentFrame.BorderSizePixel  = 0
ContentFrame.Parent           = MainFrame

local ScrollFrame = Instance.new("ScrollingFrame")
ScrollFrame.Name                   = "ScrollingFrame"
ScrollFrame.Size                   = UDim2.new(1, 0, 1, 0)
ScrollFrame.CanvasSize             = UDim2.new(0, 0, 0, 0)
ScrollFrame.CanvasPosition         = Vector2.new(0, 0)
ScrollFrame.ScrollBarThickness     = 4
ScrollFrame.ScrollBarImageColor3   = Color3.fromRGB(60, 60, 60)
ScrollFrame.BackgroundTransparency = 1
ScrollFrame.BorderSizePixel        = 0
ScrollFrame.Parent                 = ContentFrame

local ScrollLayout = Instance.new("UIListLayout")
ScrollLayout.Padding            = UDim.new(0, 8)
ScrollLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
ScrollLayout.VerticalAlignment  = Enum.VerticalAlignment.Top
ScrollLayout.SortOrder          = Enum.SortOrder.LayoutOrder
ScrollLayout.Parent             = ScrollFrame

-- Auto-resize canvas when content changes
ScrollLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    ScrollFrame.CanvasSize = UDim2.new(0, 0, 0, ScrollLayout.AbsoluteContentSize.Y + 16)
end)


-- // [4] FUNCTIONALITY // --

-- â”€â”€ Helper: standard toggle row â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
-- Returns the outer Frame so callers can set LayoutOrder, etc.
local function makeToggle(labelText, defaultState)
    defaultState = defaultState or false
    toggleStates[labelText] = defaultState

    local row = Instance.new("Frame")
    row.Name             = labelText
    row.Size             = UDim2.new(0, 230, 0, 38)
    row.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    row.BorderSizePixel  = 0

    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size                 = UDim2.new(0.6, 0, 1, 0)
    lbl.Position             = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text                 = labelText
    lbl.TextColor3           = Color3.fromRGB(235, 235, 235)
    lbl.Font                 = Enum.Font.Gotham
    lbl.TextSize             = 11
    lbl.TextXAlignment       = Enum.TextXAlignment.Left
    lbl.Parent               = row

    local btn = Instance.new("TextButton")
    btn.Size             = UDim2.new(0, 60, 0, 24)
    btn.Position         = UDim2.new(1, -70, 0, 7)
    btn.BackgroundColor3 = defaultState
        and Color3.fromRGB(255, 30, 60)
        or  Color3.fromRGB(30, 30, 30)
    btn.Text             = defaultState and "ON" or "OFF"
    btn.TextColor3       = defaultState
        and Color3.fromRGB(255, 255, 255)
        or  Color3.fromRGB(255, 70, 70)
    btn.Font             = Enum.Font.GothamBold
    btn.TextSize         = 12
    btn.BorderSizePixel  = 0
    btn.AutoButtonColor  = false
    btn.Parent           = row

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    btn.MouseButton1Click:Connect(function()
        toggleStates[labelText] = not toggleStates[labelText]
        local on = toggleStates[labelText]
        btn.Text             = on and "ON" or "OFF"
        btn.TextColor3       = on and Color3.new(1,1,1) or Color3.fromRGB(255,70,70)
        btn.BackgroundColor3 = on and Color3.fromRGB(255,30,60) or Color3.fromRGB(30,30,30)
    end)

    return row
end

-- â”€â”€ Helper: action button row (single press, e.g. "Server Hop")
local function makeActionRow(labelText, btnLabel, callback)
    local row = Instance.new("Frame")
    row.Name             = labelText
    row.Size             = UDim2.new(0, 230, 0, 38)
    row.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    row.BorderSizePixel  = 0

    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size                 = UDim2.new(0.6, 0, 1, 0)
    lbl.Position             = UDim2.new(0, 10, 0, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text                 = labelText
    lbl.TextColor3           = Color3.fromRGB(235, 235, 235)
    lbl.Font                 = Enum.Font.Gotham
    lbl.TextSize             = 11
    lbl.TextXAlignment       = Enum.TextXAlignment.Left
    lbl.Parent               = row

    local btn = Instance.new("TextButton")
    btn.Size             = UDim2.new(0, 60, 0, 24)
    btn.Position         = UDim2.new(1, -70, 0, 7)
    btn.BackgroundColor3 = Color3.fromRGB(255, 30, 60)
    btn.Text             = btnLabel or "RUN"
    btn.TextColor3       = Color3.new(1, 1, 1)
    btn.Font             = Enum.Font.GothamBold
    btn.TextSize         = 12
    btn.BorderSizePixel  = 0
    btn.Parent           = row

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn

    if callback then
        btn.MouseButton1Click:Connect(callback)
    end

    return row
end

-- â”€â”€ Helper: slider row (e.g. GUI Size) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local function makeSliderRow(labelText, defaultPct, callback)
    local row = Instance.new("Frame")
    row.Name             = labelText
    row.Size             = UDim2.new(0, 230, 0, 46)
    row.BackgroundColor3 = Color3.fromRGB(22, 22, 22)
    row.BorderSizePixel  = 0

    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row

    local lbl = Instance.new("TextLabel")
    lbl.Size                 = UDim2.new(1, -20, 0, 18)
    lbl.Position             = UDim2.new(0, 10, 0, 4)
    lbl.BackgroundTransparency = 1
    lbl.Text                 = labelText .. ": " .. math.floor(defaultPct * 100) .. "%"
    lbl.TextColor3           = Color3.fromRGB(235, 235, 235)
    lbl.Font                 = Enum.Font.Gotham
    lbl.TextSize             = 11
    lbl.TextXAlignment       = Enum.TextXAlignment.Left
    lbl.Parent               = row

    -- Track
    local track = Instance.new("TextButton")
    track.Size             = UDim2.new(1, -20, 0, 8)
    track.Position         = UDim2.new(0, 10, 0, 30)
    track.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    track.BorderSizePixel  = 0
    track.Text             = ""
    track.AutoButtonColor  = false
    track.Parent           = row

    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius = UDim.new(1, 0)
    trackCorner.Parent = track

    -- Fill
    local fill = Instance.new("Frame")
    fill.Size             = UDim2.new(defaultPct, 0, 1, 0)
    fill.BackgroundColor3 = Color3.fromRGB(255, 30, 60)
    fill.BorderSizePixel  = 0
    fill.Parent           = track

    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1, 0)
    fillCorner.Parent = fill

    -- Dragging behaviour on track
    local draggingSlider = false
    track.MouseButton1Down:Connect(function()
        draggingSlider = true
    end)
    UserInputService.InputEnded:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 then
            draggingSlider = false
        end
    end)
    UserInputService.InputChanged:Connect(function(inp)
        if draggingSlider and inp.UserInputType == Enum.UserInputType.MouseMovement then
            local abs = track.AbsolutePosition
            local sz  = track.AbsoluteSize
            local pct = math.clamp((inp.Position.X - abs.X) / sz.X, 0, 1)
            fill.Size = UDim2.new(pct, 0, 1, 0)
            lbl.Text  = labelText .. ": " .. math.floor(pct * 100) .. "%"
            if callback then callback(pct) end
        end
    end)

    return row
end

-- â”€â”€ Tab population â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
-- All feature names extracted directly from the trace.

local tabData = {
    Main = {
        { type = "toggle", name = "No Animation" },
        { type = "toggle", name = "Desync" },
        { type = "toggle", name = "Admin Panel Spam" },
        { type = "toggle", name = "Auto Turret Destroy" },
        { type = "toggle", name = "Steal Upstairs" },
        { type = "toggle", name = "Semi Invis" },
        { type = "toggle", name = "Ragdoll Self" },
        { type = "toggle", name = "Boogie Inf Jump" },
        { type = "toggle", name = "Booster" },
        { type = "toggle", name = "Rainbow Base" },
        { type = "toggle", name = "Anti Trap" },
        { type = "toggle", name = "X-Ray Bases" },
        { type = "toggle", name = "Player ESP" },
        { type = "toggle", name = "Base ESP" },
        { type = "toggle", name = "ESP Best" },
        { type = "toggle", name = "Anti-Ragdoll" },
        { type = "toggle", name = "Optimizer" },
        { type = "slider", name = "GUI Size",  default = 1.0 },
        { type = "toggle", name = "Anti-Bee" },
        { type = "toggle", name = "Anti-Sentry" },
        { type = "toggle", name = "Anti-Disco" },
        { type = "toggle", name = "Auto Kick" },
        { type = "toggle", name = "Auto Load", default = true },
        { type = "action", name = "Server Hop",  btn = "RUN" },
    },
    Player = {
        -- Player tab contents not explicitly logged beyond tab label;
        -- rows are populated from the same scrolling frame pattern.
    },
    Stealer = {
        { type = "toggle", name = "Auto Steal if Near" },
        { type = "toggle", name = "Auto Steal Highest" },
        { type = "toggle", name = "Auto Steal Progress" },
        { type = "toggle", name = "Steal Auto TP" },
    },
    Utility = {
        { type = "toggle", name = "Aimbot" },
        { type = "toggle", name = "Base Button" },
    },
}

-- Rows stored per tab so we can hide/show
local tabRows = {}

local function buildTab(tabName)
    tabRows[tabName] = tabRows[tabName] or {}
    local data = tabData[tabName] or {}
    for i, entry in ipairs(data) do
        local row
        if entry.type == "toggle" then
            row = makeToggle(entry.name, entry.default)
        elseif entry.type == "action" then
            row = makeActionRow(entry.name, entry.btn)
        elseif entry.type == "slider" then
            row = makeSliderRow(entry.name, entry.default or 1.0, function(pct)
                if entry.name == "GUI Size" then
                    MainScale.Scale = pct * 1.5 + 0.5  -- range 0.5 â€“ 2.0
                end
            end)
        end
        if row then
            row.LayoutOrder = i
            row.Parent = ScrollFrame
            row.Visible = (tabName == activeTab)
            table.insert(tabRows[tabName], row)
        end
    end
end

-- Build all tabs upfront (hidden except active)
for tabName, _ in pairs(tabData) do
    buildTab(tabName)
end

-- â”€â”€ Tab switching logic â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local tabButtons = {
    Main    = TabMain,
    Player  = TabPlayer,
    Stealer = TabStealer,
    Utility = TabUtility,
}

local function switchTab(tabName)
    -- Hide current tab rows
    if tabRows[activeTab] then
        for _, row in ipairs(tabRows[activeTab]) do
            row.Visible = false
        end
    end
    -- Deactivate old tab button
    if tabButtons[activeTab] then
        tabButtons[activeTab].BackgroundColor3 = Color3.fromRGB(25, 25, 25)
        tabButtons[activeTab].TextColor3       = Color3.fromRGB(200, 200, 200)
    end

    activeTab = tabName

    -- Show new tab rows
    if tabRows[activeTab] then
        for _, row in ipairs(tabRows[activeTab]) do
            row.Visible = true
        end
    end
    -- Activate new tab button
    if tabButtons[activeTab] then
        tabButtons[activeTab].BackgroundColor3 = Color3.fromRGB(255, 30, 60)
        tabButtons[activeTab].TextColor3       = Color3.new(1, 1, 1)
    end
end

-- Connect tab button clicks
for tabName, btn in pairs(tabButtons) do
    btn.MouseButton1Click:Connect(function()
        switchTab(tabName)
    end)
end

-- Activate default tab visually
switchTab("Main")

-- â”€â”€ Minimise / expand â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
local minimised = false
MinimiseButton.MouseButton1Click:Connect(function()
    minimised = not minimised
    ContentFrame.Visible = not minimised
    TabBar.Visible       = not minimised
    MainFrame.Size = minimised
        and UDim2.new(0, 260, 0, 34)
        or  UDim2.new(0, 260, 0, 380)
    MinimiseButton.Text = minimised and "+" or "-"
end)

-- â”€â”€ Drag (header) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
HeaderFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging  = true
        dragStart = input.Position
        startPos  = MainFrame.Position
    end
end)

HeaderFrame.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- â”€â”€ Keybind: RightShift  â†’ toggle GUI visibility â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        ScreenGui.Enabled = not ScreenGui.Enabled
    end
end)


-- // [5] INITIAL STATE // --

ScreenGui.Enabled = true
switchTab("Main")   -- ensure first tab is active and shown
