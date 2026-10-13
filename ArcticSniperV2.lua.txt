--[[
    AS | ARCTIC SNIPER V2
    Standalone GUI | Delta-style executor environment
    Scanner and Brainrot feed are demo/test tools.
    No automatic code redemption or live-game scraping.
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")

local OLD = PlayerGui:FindFirstChild("ArcticSniperV2")
if OLD then OLD:Destroy() end

local C = {
    BG = Color3.fromRGB(7, 13, 25),
    PANEL = Color3.fromRGB(13, 24, 42),
    PANEL2 = Color3.fromRGB(19, 36, 59),
    BLUE = Color3.fromRGB(86, 190, 255),
    ICE = Color3.fromRGB(190, 235, 255),
    WHITE = Color3.fromRGB(240, 248, 255),
    MUTED = Color3.fromRGB(135, 163, 190),
    GREEN = Color3.fromRGB(83, 225, 168),
    RED = Color3.fromRGB(255, 105, 125),
}

local gui = Instance.new("ScreenGui")
gui.Name = "ArcticSniperV2"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999
gui.Parent = PlayerGui

local function make(class, props, parent)
    local obj = Instance.new(class)
    for k, v in pairs(props or {}) do
        obj[k] = v
    end
    obj.Parent = parent
    return obj
end

local function round(obj, radius)
    make("UICorner", {
        CornerRadius = UDim.new(0, radius or 10)
    }, obj)
end

local function stroke(obj, color, thickness, transparency)
    make("UIStroke", {
        Color = color or C.BLUE,
        Thickness = thickness or 1,
        Transparency = transparency or 0.5
    }, obj)
end

local function label(parent, text, size, position, fontSize, color)
    return make("TextLabel", {
        BackgroundTransparency = 1,
        Text = text,
        Size = size,
        Position = position,
        Font = Enum.Font.Gotham,
        TextSize = fontSize or 13,
        TextColor3 = color or C.WHITE,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        TextWrapped = true,
    }, parent)
end

local function button(parent, text, size, position, callback)
    local b = make("TextButton", {
        Size = size,
        Position = position,
        BackgroundColor3 = C.PANEL2,
        Text = text,
        TextColor3 = C.WHITE,
        TextSize = 12,
        Font = Enum.Font.GothamSemibold,
        AutoButtonColor = false,
    }, parent)
    round(b, 8)
    stroke(b, C.BLUE, 1, 0.72)

    b.MouseEnter:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.12), {
            BackgroundColor3 = Color3.fromRGB(28, 54, 82)
        }):Play()
    end)

    b.MouseLeave:Connect(function()
        TweenService:Create(b, TweenInfo.new(0.12), {
            BackgroundColor3 = C.PANEL2
        }):Play()
    end)

    b.Activated:Connect(function()
        if callback then callback() end
    end)

    return b
end

local function input(parent, placeholder, size, position)
    local box = make("TextBox", {
        Size = size,
        Position = position,
        BackgroundColor3 = C.BG,
        Text = "",
        PlaceholderText = placeholder,
        PlaceholderColor3 = C.MUTED,
        TextColor3 = C.WHITE,
        TextSize = 13,
        Font = Enum.Font.Gotham,
        ClearTextOnFocus = false,
    }, parent)
    round(box, 8)
    stroke(box, C.BLUE, 1, 0.8)
    return box
end

local main = make("Frame", {
    Name = "Main",
    Size = UDim2.new(0, 720, 0, 480),
    Position = UDim2.new(0.5, -360, 0.5, -240),
    BackgroundColor3 = C.BG,
    BorderSizePixel = 0,
}, gui)
round(main, 14)
stroke(main, C.BLUE, 1.5, 0.3)

local scale = make("UIScale", {
    Scale = 1
}, main)

local function updateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end
    local viewport = camera.ViewportSize
    scale.Scale = math.min(1, viewport.X / 750, viewport.Y / 510)
end

updateScale()
if workspace.CurrentCamera then
    workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(updateScale)
end

local header = make("Frame", {
    Size = UDim2.new(1, 0, 0, 58),
    BackgroundColor3 = C.PANEL,
    BorderSizePixel = 0,
}, main)
round(header, 14)

make("Frame", {
    Size = UDim2.new(1, 0, 0, 15),
    Position = UDim2.new(0, 0, 1, -15),
    BackgroundColor3 = C.PANEL,
    BorderSizePixel = 0,
}, header)

local accent = make("Frame", {
    Size = UDim2.new(1, -28, 0, 2),
    Position = UDim2.new(0, 14, 1, -1),
    BackgroundColor3 = C.BLUE,
    BorderSizePixel = 0,
}, header)

label(header, "❄", UDim2.new(0, 38, 0, 40),
    UDim2.new(0, 14, 0, 7), 28, C.BLUE)

label(header, "ARCTIC SNIPER", UDim2.new(0, 240, 0, 24),
    UDim2.new(0, 55, 0, 9), 17, C.ICE)

label(header, "V2  •  FROST EDITION", UDim2.new(0, 230, 0, 18),
    UDim2.new(0, 57, 0, 32), 10, C.MUTED)

local close = button(header, "✕", UDim2.new(0, 34, 0, 32),
    UDim2.new(1, -46, 0, 13), function()
        gui:Destroy()
    end)
close.BackgroundColor3 = Color3.fromRGB(75, 28, 43)

local mini = button(gui, "❄ AS", UDim2.new(0, 72, 0, 38),
    UDim2.new(0, 18, 0.5, -19), function()
        main.Visible = not main.Visible
    end)
mini.Visible = false

-- Drag window
do
    local dragging = false
    local dragStart, startPosition

    header.InputBegan:Connect(function(io)
        if io.UserInputType == Enum.UserInputType.MouseButton1
            or io.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = io.Position
            startPosition = main.Position
        end
    end)

    UserInputService.InputChanged:Connect(function(io)
        if dragging and (
            io.UserInputType == Enum.UserInputType.MouseMovement
            or io.UserInputType == Enum.UserInputType.Touch
        ) then
            local delta = io.Position - dragStart
            main.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    UserInputService.InputEnded:Connect(function(io)
        if io.UserInputType == Enum.UserInputType.MouseButton1
            or io.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

local sidebar = make("Frame", {
    Size = UDim2.new(0, 155, 1, -74),
    Position = UDim2.new(0, 10, 0, 66),
    BackgroundColor3 = C.PANEL,
    BorderSizePixel = 0,
}, main)
round(sidebar, 10)

label(sidebar, "MAIN MENU", UDim2.new(1, -20, 0, 25),
    UDim2.new(0, 12, 0, 8), 10, C.MUTED)

local content = make("Frame", {
    Size = UDim2.new(1, -185, 1, -74),
    Position = UDim2.new(0, 175, 0, 66),
    BackgroundTransparency = 1,
}, main)

local pageNames = {
    "Home",
    "Scanner",
    "Redeemer",
    "Brainrots",
    "Discord",
    "Settings",
    "Activity Log",
}

local pages = {}
local navButtons = {}
local logs = {}
local activePage = "Home"
local scanEnabled = false
local spawnEnabled = false
local autoSubmit = false
local notifications = true

local function addLog(message)
    local timestamp = os.date("%H:%M:%S")
    table.insert(logs, 1, "[" .. timestamp .. "] " .. message)
    if #logs > 80 then table.remove(logs) end

    if pages["Activity Log"] then
        local logPage = pages["Activity Log"]
        local list = logPage:FindFirstChild("LogList")
        if list then
            for _, child in ipairs(list:GetChildren()) do
                if child:IsA("TextLabel") then child:Destroy() end
            end

            for i = 1, math.min(#logs, 22) do
                label(list, logs[i], UDim2.new(1, -12, 0, 25),
                    UDim2.new(0, 6, 0, (i - 1) * 27), 11, C.ICE)
            end
            list.CanvasSize = UDim2.new(0, 0, 0, math.min(#logs, 22) * 27)
        end
    end
end

local function newPage(name)
    local p = make("ScrollingFrame", {
        Name = name,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = C.BLUE,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false,
    }, content)

    make("UIListLayout", {
        Padding = UDim.new(0, 10),
        SortOrder = Enum.SortOrder.LayoutOrder,
    }, p)

    make("UIPadding", {
        PaddingTop = UDim.new(0, 2),
        PaddingBottom = UDim.new(0, 12),
        PaddingLeft = UDim.new(0, 2),
        PaddingRight = UDim.new(0, 8),
    }, p)

    pages[name] = p
    return p
end

local function card(parent, title, description, height)
    local frame = make("Frame", {
        Size = UDim2.new(1, -4, 0, height or 95),
        BackgroundColor3 = C.PANEL,
        BorderSizePixel = 0,
    }, parent)
    round(frame, 10)
    stroke(frame, C.BLUE, 1, 0.83)

    label(frame, title, UDim2.new(1, -24, 0, 25),
        UDim2.new(0, 12, 0, 8), 14, C.ICE)

    if description then
        label(frame, description, UDim2.new(1, -24, 0, 38),
            UDim2.new(0, 12, 0, 34), 11, C.MUTED)
    end

    return frame
end

local function switch(parent, title, initial, callback)
    local frame = make("Frame", {
        Size = UDim2.new(1, -4, 0, 45),
        BackgroundColor3 = C.PANEL,
        BorderSizePixel = 0,
    }, parent)
    round(frame, 9)

    label(frame, title, UDim2.new(1, -100, 1, 0),
        UDim2.new(0, 12, 0, 0), 12, C.WHITE)

    local on = initial
    local b = button(frame, on and "ON" or "OFF",
        UDim2.new(0, 58, 0, 29),
        UDim2.new(1, -70, 0.5, -14), nil)

    local function refresh()
        b.Text = on and "ON" or "OFF"
        b.BackgroundColor3 = on and Color3.fromRGB(20, 76, 94) or C.PANEL2
        b.TextColor3 = on and C.GREEN or C.MUTED
    end

    b.Activated:Connect(function()
        on = not on
        refresh()
        if callback then callback(on) end
    end)

    refresh()
    return frame, function() return on end
end

local function goTo(name)
    activePage = name
    for pageName, p in pairs(pages) do
        p.Visible = pageName == name
    end
    for pageName, b in pairs(navButtons) do
        b.BackgroundColor3 = pageName == name and C.PANEL2 or C.PANEL
        b.TextColor3 = pageName == name and C.ICE or C.MUTED
    end
end

for i, name in ipairs(pageNames) do
    local b = button(sidebar, name, UDim2.new(1, -16, 0, 36),
        UDim2.new(0, 8, 0, 39 + (i - 1) * 43), function()
            goTo(name)
        end)
    b.TextXAlignment = Enum.TextXAlignment.Left
    b.Text = "   " .. name
    b.BackgroundColor3 = C.PANEL
    b.TextColor3 = C.MUTED
    navButtons[name] = b
end

local footer = label(sidebar, "ARCTIC SYSTEMS\nv2.0 • ICE BLUE",
    UDim2.new(1, -20, 0, 36), UDim2.new(0, 12, 1, -44), 9, C.MUTED)
footer.TextYAlignment = Enum.TextYAlignment.Bottom

-- HOME
do
    local p = newPage("Home")

    local welcome = card(p, "WELCOME TO ARCTIC SNIPER",
        "A frosty control panel for your scanner tools and notifications.", 112)

    label(welcome, "❄  SYSTEM READY", UDim2.new(1, -24, 0, 20),
        UDim2.new(0, 12, 1, -28), 11, C.GREEN)

    local stats = card(p, "QUICK STATUS", nil, 95)
    label(stats, "Scanner: DEMO MODE", UDim2.new(0.5, -16, 0, 24),
        UDim2.new(0, 12, 0, 12), 11, C.ICE)
    label(stats, "Spawn feed: TEST MODE", UDim2.new(0.5, -16, 0, 24),
        UDim2.new(0.5, 0, 0, 12), 11, C.ICE)
    label(stats, "Redeemer: Manual / preview only", UDim2.new(1, -24, 0, 24),
        UDim2.new(0, 12, 0, 47), 11, C.MUTED)

    local actions = card(p, "QUICK ACTIONS", nil, 88)
    button(actions, "OPEN SCANNER", UDim2.new(0.48, -4, 0, 34),
        UDim2.new(0, 10, 0, 42), function() goTo("Scanner") end)
    button(actions, "SPAWN FEED", UDim2.new(0.48, -4, 0, 34),
        UDim2.new(0.52, 0, 0, 42), function() goTo("Brainrots") end)

    card(p, "IMPORTANT", 
        "This standalone UI cannot see private game events by itself. Use the demo buttons to test the interface.", 80)
end

-- SCANNER
do
    local p = newPage("Scanner")

    card(p, "CODE SCANNER",
        "Paste announcement text to test detection of phrases such as 'use code' or 'the code is'.", 82)

    local textBox = input(p, "Paste announcement text here...",
        UDim2.new(1, -4, 0, 42), UDim2.new(0, 0, 0, 0))
    textBox.LayoutOrder = 2

    local result = card(p, "SCAN RESULT", "Waiting for text...", 85)
    result.LayoutOrder = 3

    button(result, "TEST SCAN", UDim2.new(0, 112, 0, 30),
        UDim2.new(1, -124, 1, -40), function()
            local text = textBox.Text
            if text == "" then
                label(result, "Enter some text first.", UDim2.new(1, -24, 0, 20),
                    UDim2.new(0, 12, 0, 36), 11, C.RED)
                addLog("Scanner test: empty input")
                return
            end

            local lower = string.lower(text)
            local found = string.match(lower, "use code")
                or string.match(lower, "the code is")
                or string.match(lower, "code:")

            local status = result:FindFirstChild("ScanStatus")
            if status then status:Destroy() end

            status = label(result,
                found and "Potential code announcement detected." or "No code phrase detected.",
                UDim2.new(1, -24, 0, 22), UDim2.new(0, 12, 0, 36),
                11, found and C.GREEN or C.MUTED)
            status.Name = "ScanStatus"

            addLog(found and "Test scanner detected a code phrase" or "Test scan completed")
        end)

    switch(p, "Scanner enabled (demo toggle)", false, function(value)
        scanEnabled = value
        addLog("Scanner demo " .. (value and "enabled" or "disabled"))
    end)

    local info = card(p, "HOW IT WORKS",
        "This tests text you provide. It does not read other players' private messages or game announcements automatically.", 90)
end

-- REDEEMER
do
    local p = newPage("Redeemer")

    card(p, "CODE REDEEMER",
        "Enter a code to validate the input format and preview it. This does not redeem codes in-game.", 82)

    local codeBox = input(p, "Enter code...",
        UDim2.new(1, -4, 0, 42), UDim2.new(0, 0, 0, 0))
    codeBox.LayoutOrder = 2

    local preview = card(p, "CODE PREVIEW", "No code entered.", 80)
    preview.LayoutOrder = 3

    button(preview, "CHECK", UDim2.new(0, 88, 0, 30),
        UDim2.new(1, -100, 1, -39), function()
            local value = string.gsub(codeBox.Text, "^%s*(.-)%s*$", "%1")
            local old = preview:FindFirstChild("CodeStatus")
            if old then old:Destroy() end

            local message
            local color = C.MUTED

            if value == "" then
                message = "Enter a code first."
                color = C.RED
            elseif #value > 64 then
                message = "Code is too long."
                color = C.RED
            elseif string.find(value, "%s") then
                message = "Code contains spaces."
                color = C.RED
            else
                message = "Input looks valid: " .. value
                color = C.GREEN
            end

            local status = label(preview, message,
                UDim2.new(1, -24, 0, 22), UDim2.new(0, 12, 0, 34), 11, color)
            status.Name = "CodeStatus"
            addLog("Code format checked")
        end)

    switch(p, "Auto-submit preference", false, function(value)
        autoSubmit = value
        addLog("Auto-submit preference " .. (value and "enabled" or "disabled"))
    end)

    card(p, "SAFETY NOTE",
        "The auto-submit toggle only saves your UI preference. It does not submit or redeem anything.", 75)
end

-- BRAINROTS
do
    local p = newPage("Brainrots")

    card(p, "BRAINROT SPAWN FEED",
        "Use the test event to preview how a spawn notification would look.", 82)

    local feed = card(p, "LATEST EVENT", "No test events yet.", 92)
    local feedText = label(feed, "Waiting for a test event...",
        UDim2.new(1, -24, 0, 24), UDim2.new(0, 12, 0, 36), 12, C.MUTED)

    button(feed, "TEST SPAWN", UDim2.new(0, 120, 0, 30),
        UDim2.new(1, -132, 1, -40), function()
            feedText.Text = "TEST • Brainrot appeared (sample event)"
            feedText.TextColor3 = C.GREEN
            addLog("Sample Brainrot spawn event")
        end)

    switch(p, "Spawn notifications (demo)", false, function(value)
        spawnEnabled = value
        addLog("Spawn feed demo " .. (value and "enabled" or "disabled"))
    end)

    card(p, "LIVE EVENTS",
        "Live spawn detection requires a supported, authorized event source. This panel does not inspect or exploit game internals.", 88)
end

-- DISCORD
do
    local p = newPage("Discord")

    card(p, "DISCORD NOTIFICATIONS",
        "Preview the message format you could send through your own bot or server backend.", 82)

    local discordBox = input(p, "Notification text...",
        UDim2.new(1, -4, 0, 42), UDim2.new(0, 0, 0, 0))
    discordBox.LayoutOrder = 2

    local preview = card(p, "MESSAGE PREVIEW",
        "❄ ARCTIC SNIPER\nNo message created yet.", 115)
    preview.LayoutOrder = 3

    button(preview, "PREVIEW", UDim2.new(0, 100, 0, 30),
        UDim2.new(1, -112, 1, -40), function()
            local value = discordBox.Text
            if value == "" then value = "Sample notification" end

            local old = preview:FindFirstChild("DiscordMessage")
            if old then old:Destroy() end

            local msg = label(preview,
                "❄ ARCTIC SNIPER\n" .. value,
                UDim2.new(1, -24, 0, 50),
                UDim2.new(0, 12, 0, 34), 11, C.ICE)
            msg.Name = "DiscordMessage"
            addLog("Discord message preview created")
        end)

    card(p, "CONNECTION STATUS",
        "Preview only — no webhook or Discord message is sent by this script.", 75)
end

-- SETTINGS
do
    local p = newPage("Settings")

    card(p, "PERSONALIZATION",
        "Adjust notification preferences for this GUI session.", 72)

    switch(p, "Desktop notifications preference", true, function(value)
        notifications = value
        addLog("Notification preference " .. (value and "enabled" or "disabled"))
    end)

    switch(p, "Show mini reopen button", false, function(value)
        mini.Visible = value
    end)

    local theme = card(p, "THEME", "Current theme: Arctic Ice Blue", 78)
    button(theme, "ICE BLUE ✓", UDim2.new(0, 115, 0, 30),
        UDim2.new(1, -127, 1, -39), function()
            addLog("Arctic Ice Blue theme selected")
        end)

    card(p, "WINDOW CONTROLS",
        "Drag the top bar to move the panel. Use the X button to close it.", 72)
end

-- ACTIVITY LOG
do
    local p = newPage("Activity Log")

    local frame = make("Frame", {
        Size = UDim2.new(1, -4, 0, 380),
        BackgroundColor3 = C.PANEL,
        BorderSizePixel = 0,
    }, p)
    round(frame, 10)
    stroke(frame, C.BLUE, 1, 0.8)

    label(frame, "ACTIVITY LOG", UDim2.new(1, -24, 0, 30),
        UDim2.new(0, 12, 0, 7), 14, C.ICE)

    local list = make("ScrollingFrame", {
        Name = "LogList",
        Size = UDim2.new(1, -20, 1, -85),
        Position = UDim2.new(0, 10, 0, 43),
        BackgroundColor3 = C.BG,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = C.BLUE,
        CanvasSize = UDim2.new(0, 0, 0, 0),
    }, frame)
    round(list, 8)

    button(frame, "CLEAR LOG", UDim2.new(0, 100, 0, 30),
        UDim2.new(1, -112, 1, -37), function()
            table.clear(logs)
            for _, child in ipairs(list:GetChildren()) do
                if child:IsA("TextLabel") then child:Destroy() end
            end
            list.CanvasSize = UDim2.new(0, 0, 0, 0)
        end)
end

-- Snowflake ambience
task.spawn(function()
    while gui.Parent do
        if not main.Parent then break end

        local snow = make("TextLabel", {
            BackgroundTransparency = 1,
            Text = "❄",
            TextColor3 = C.BLUE,
            TextTransparency = 0.35,
            TextSize = math.random(10, 19),
            Font = Enum.Font.Gotham,
            Size = UDim2.new(0, 24, 0, 24),
            Position = UDim2.new(math.random(5, 95) / 100, 0, -0.06, 0),
            ZIndex = 20,
        }, main)

        local duration = math.random(5, 9)
        local tween = TweenService:Create(snow, TweenInfo.new(duration, Enum.EasingStyle.Linear), {
            Position = UDim2.new(
                snow.Position.X.Scale + math.random(-8, 8) / 100,
                0, 1.1, 0
            ),
            TextTransparency = 1,
        })
        tween:Play()
        tween.Completed:Connect(function()
            snow:Destroy()
        end)

        task.wait(0.25)
    end
end)

addLog("Arctic Sniper V2 interface loaded")
goTo("Home")