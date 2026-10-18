--[[
    AUTO CRAFT - MULTI GEAR (AUTO UID GearAutoCrafter - INSTANT)
    TimeJumper | DiamondCookie | NetRetractor
]]

-- ============================================================
-- SERVICES
-- ============================================================
local Players           = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService  = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

-- ============================================================
-- CONFIG
-- ============================================================
local CONFIG = {
    -- HANYA INI DELAY YANG TERSISA
    LoopDelay = 0.05, 
    
    ToggleKey = Enum.KeyCode.RightShift,
    GuiName   = "MultiGearAutoCraft",
    MachineType = "GearAutoCrafter",
}

-- ============================================================
-- GEARS
-- ============================================================
local GEARS = {
    {
        name  = "TimeJumper",
        items = {
            { id = "TeleportWand",      count = 10 },
            { id = "MagnifyingGlass",   count = 25 },
            { id = "SupremeAutoFeeder", count = 1  },
        },
    },
    {
        name  = "DiamondCookie",
        items = {
            { id = "XpCookie",     count = 1 },
            { id = "GoldenCookie", count = 1 },
            { id = "PetToy",       count = 1 },
        },
    },
    {
        name  = "NetRetractor",
        items = {
            { id = "NetMover",          count = 5 },
            { id = "ExtremeAutoFeeder", count = 1 },
        },
    },
}

-- ============================================================
-- HELPERS
-- ============================================================
local function expandItems(spec)
    local list = {}
    for _, e in ipairs(spec) do
        for _ = 1, e.count do
            table.insert(list, e.id)
        end
    end
    return list
end

local function describeItems(spec)
    local parts, total = {}, 0
    for _, e in ipairs(spec) do
        total = total + e.count
        table.insert(parts, e.count .. "x " .. e.id)
    end
    return table.concat(parts, ", "), total
end

-- ============================================================
-- PLAYER DATA (untuk cari UID GearAutoCrafter)
-- ============================================================
local getPlayerData
do
    local ok, pdm = pcall(function()
        local TS = ReplicatedStorage:WaitForChild("TS", 10)
        if not TS then return nil end
        return require(TS:WaitForChild("state"):WaitForChild("player-data"))
    end)
    if ok and pdm and pdm.getPlayerData then
        getPlayerData = pdm.getPlayerData
    end
end

local function findAllGearAutoCrafter()
    local found = {}
    if not getPlayerData then return found end
    local ok, data = pcall(getPlayerData, LocalPlayer)
    if not ok or not data or not data.ponds then return found end

    for pondKey, pondData in pairs(data.ponds) do
        if type(pondData) == "table" and type(pondData.machines) == "table" then
            for uid, mData in pairs(pondData.machines) do
                if type(mData) == "table" and mData.machineType == CONFIG.MachineType then
                    table.insert(found, {
                        pondKey    = pondKey,
                        uid        = uid,
                        position   = mData.position,
                        customName = pondData.customName,
                    })
                end
            end
        end
    end
    return found
end

local function pickGearAutoCrafter()
    local list = findAllGearAutoCrafter()
    if #list == 0 then return nil, list end
    for _, e in ipairs(list) do
        if tostring(e.pondKey):lower() == "pond2" then return e, list end
    end
    return list[1], list
end

-- ============================================================
-- REMOTE
-- ============================================================
local Remote = {}
local CurrentUid = nil

do
    local ok, container = pcall(function()
        return ReplicatedStorage
            :WaitForChild("rbxts_include", 10)
            :WaitForChild("node_modules", 10)
            :WaitForChild("@rbxts", 10)
            :WaitForChild("remo", 10)
            :WaitForChild("src", 10)
            :WaitForChild("container", 10)
    end)

    if not ok or not container then
        warn("[AutoCraft] Remote container tidak ditemukan!")
        return
    end

    local function fire(name, ...)
        local r = container:FindFirstChild(name)
        if r and r:IsA("RemoteEvent") then
            r:FireServer(...)
            return true
        end
        return false
    end

    function Remote.selectItem(name, amount)
        if not CurrentUid then return false end
        return fire("autoCrafting.selectCraftingItem", CurrentUid, name, amount or 1)
    end
    
    function Remote.submitItems(list)
        if not CurrentUid then return false end
        return fire("autoCrafting.submitItems", CurrentUid, list)
    end
    
    function Remote.startCraft()
        if not CurrentUid then return false end
        return fire("autoCrafting.startCraft", CurrentUid)
    end
    
    function Remote.collectCraft()
        if not CurrentUid then return false end
        return fire("autoCrafting.collectCraft", CurrentUid)
    end
    
    function Remote.cancelCraft()
        if not CurrentUid then return false end
        return fire("autoCrafting.cancelCraft", CurrentUid)
    end
end

-- ============================================================
-- RUN CRAFT (TANPA DELAY INTERNAL)
-- ============================================================
local function runCraft(gear)
    if not CurrentUid then return end

    -- Ditembak instan tanpa task.wait
    Remote.selectItem(gear.name, 1)
    Remote.submitItems(expandItems(gear.items))
    Remote.startCraft()
    Remote.collectCraft()
    Remote.cancelCraft()
end

-- ============================================================
-- STATE
-- ============================================================
local State = {
    running      = false,
    loopCount    = 0,
    thread       = nil,
    selectedGear = GEARS[1],
    uid          = nil,
    uidPond      = nil,
}

-- ============================================================
-- UI HELPERS
-- ============================================================
local function corner(parent, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 8)
    c.Parent = parent
    return c
end

local function stroke(parent)
    local s = Instance.new("UIStroke")
    s.Color = Color3.fromRGB(80, 80, 110)
    s.Thickness = 1.5
    s.Parent = parent
    return s
end

local function makeFrame(props)
    local f = Instance.new("Frame")
    f.BorderSizePixel = 0
    for k, v in pairs(props) do f[k] = v end
    return f
end

local function makeLabel(props)
    local l = Instance.new("TextLabel")
    l.BackgroundTransparency = 1
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.Font = Enum.Font.Gotham
    for k, v in pairs(props) do l[k] = v end
    return l
end

local function makeButton(props)
    local b = Instance.new("TextButton")
    b.BorderSizePixel = 0
    b.AutoButtonColor = true
    b.Font = Enum.Font.GothamBold
    b.TextColor3 = Color3.fromRGB(255, 255, 255)
    for k, v in pairs(props) do b[k] = v end
    return b
end

-- ============================================================
-- BUILD GUI
-- ============================================================
local oldGui = PlayerGui:FindFirstChild(CONFIG.GuiName)
if oldGui then oldGui:Destroy() end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = CONFIG.GuiName
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = PlayerGui

local Main = makeFrame({
    Size = UDim2.new(0, 300, 0, 380),
    Position = UDim2.new(0.5, -150, 0.5, -190),
    BackgroundColor3 = Color3.fromRGB(28, 28, 38),
    Active = true,
    Draggable = true,
    Parent = ScreenGui,
})
corner(Main, 12)
stroke(Main)

local TitleBar = makeFrame({
    Size = UDim2.new(1, 0, 0, 38),
    BackgroundColor3 = Color3.fromRGB(45, 45, 65),
    Parent = Main,
})
corner(TitleBar, 12)

makeFrame({
    Size = UDim2.new(1, 0, 0, 15),
    Position = UDim2.new(0, 0, 1, -15),
    BackgroundColor3 = Color3.fromRGB(45, 45, 65),
    Parent = TitleBar,
})

makeLabel({
    Size = UDim2.new(1, -90, 1, 0),
    Position = UDim2.new(0, 14, 0, 0),
    Text = "⚡ Auto Crafter",
    TextColor3 = Color3.fromRGB(220, 220, 255),
    TextSize = 15,
    Font = Enum.Font.GothamBold,
    Parent = TitleBar,
})

local MinBtn = makeButton({
    Size = UDim2.new(0, 26, 0, 26),
    Position = UDim2.new(1, -62, 0, 6),
    BackgroundColor3 = Color3.fromRGB(80, 80, 130),
    Text = "—",
    TextSize = 16,
    Parent = TitleBar,
})
corner(MinBtn, 6)

local CloseBtn = makeButton({
    Size = UDim2.new(0, 26, 0, 26),
    Position = UDim2.new(1, -32, 0, 6),
    BackgroundColor3 = Color3.fromRGB(200, 60, 60),
    Text = "✕",
    TextSize = 14,
    Parent = TitleBar,
})
corner(CloseBtn, 6)

local Content = makeFrame({
    Size = UDim2.new(1, -24, 1, -52),
    Position = UDim2.new(0, 12, 0, 46),
    BackgroundTransparency = 1,
    Parent = Main,
})

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 8)
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Content

makeLabel({
    Size = UDim2.new(1, 0, 0, 16),
    Text = "SELECT GEAR",
    TextColor3 = Color3.fromRGB(160, 160, 200),
    TextSize = 11,
    Font = Enum.Font.GothamBold,
    LayoutOrder = 1,
    Parent = Content,
})

local GearBtn = makeButton({
    Size = UDim2.new(1, 0, 0, 34),
    BackgroundColor3 = Color3.fromRGB(50, 50, 70),
    Text = State.selectedGear.name .. "  ▼",
    TextSize = 14,
    LayoutOrder = 2,
    Parent = Content,
})
corner(GearBtn, 6)

local Dropdown = makeFrame({
    Size = UDim2.new(1, 0, 0, 0),
    BackgroundColor3 = Color3.fromRGB(38, 38, 52),
    Visible = false,
    ZIndex = 5,
    LayoutOrder = 3,
    Parent = Content,
    AutomaticSize = Enum.AutomaticSize.Y,
})
corner(Dropdown, 6)

local DropdownPad = Instance.new("UIPadding")
DropdownPad.PaddingLeft   = UDim.new(0, 4)
DropdownPad.PaddingRight  = UDim.new(0, 4)
DropdownPad.PaddingTop    = UDim.new(0, 4)
DropdownPad.PaddingBottom = UDim.new(0, 4)
DropdownPad.Parent = Dropdown

local DropdownLayout = Instance.new("UIListLayout")
DropdownLayout.Padding = UDim.new(0, 2)
DropdownLayout.SortOrder = Enum.SortOrder.LayoutOrder
DropdownLayout.Parent = Dropdown

local InfoBox = makeFrame({
    Size = UDim2.new(1, 0, 0, 62),
    BackgroundColor3 = Color3.fromRGB(40, 40, 55),
    LayoutOrder = 4,
    Parent = Content,
})
corner(InfoBox, 6)

local InfoLabel = makeLabel({
    Size = UDim2.new(1, -16, 1, 0),
    Position = UDim2.new(0, 8, 0, 0),
    Text = "",
    TextColor3 = Color3.fromRGB(180, 180, 210),
    TextSize = 11,
    TextWrapped = true,
    Parent = InfoBox,
})

local function refreshInfo()
    local desc, total = describeItems(State.selectedGear.items)
    InfoLabel.Text = "Bahan: " .. desc .. "\nTotal: " .. total .. " item per craft"
end

local UidBox = makeFrame({
    Size = UDim2.new(1, 0, 0, 50),
    BackgroundColor3 = Color3.fromRGB(40, 40, 55),
    LayoutOrder = 5,
    Parent = Content,
})
corner(UidBox, 6)

local UidLabel = makeLabel({
    Size = UDim2.new(1, -16, 1, 0),
    Position = UDim2.new(0, 8, 0, 0),
    Text = "UID: (belum dicari)",
    TextColor3 = Color3.fromRGB(200, 200, 255),
    TextSize = 10,
    TextWrapped = true,
    Font = Enum.Font.Code,
    Parent = UidBox,
})

local UidRefreshBtn = makeButton({
    Size = UDim2.new(0, 70, 0, 20),
    Position = UDim2.new(1, -78, 0, 6),
    BackgroundColor3 = Color3.fromRGB(70, 110, 180),
    Text = "🔄 Cari",
    TextSize = 11,
    ZIndex = 6,
    Parent = UidBox,
})
corner(UidRefreshBtn, 4)

local function refreshUid(showPrint)
    local chosen, list = pickGearAutoCrafter()
    if chosen then
        State.uid     = chosen.uid
        State.uidPond = chosen.pondKey
        UidLabel.Text = string.format("UID: %s\npond: %s", tostring(chosen.uid), tostring(chosen.pondKey))
        UidLabel.TextColor3 = Color3.fromRGB(160, 255, 160)
        if showPrint then
            print(string.format("[AutoCraft] GearAutoCrafter ditemukan di %s\n  UID = %s",
                tostring(chosen.pondKey), tostring(chosen.uid)))
        end
    else
        State.uid     = nil
        State.uidPond = nil
        UidLabel.Text = "UID: GearAutoCrafter TIDAK ditemukan"
        UidLabel.TextColor3 = Color3.fromRGB(255, 150, 150)
        if showPrint then
            warn("[AutoCraft] GearAutoCrafter tidak ditemukan di pond manapun.")
        end
    end
    return chosen
end

local ToggleBtn = makeButton({
    Size = UDim2.new(1, 0, 0, 52),
    BackgroundColor3 = Color3.fromRGB(60, 180, 100),
    Text = "▶  START",
    TextSize = 18,
    LayoutOrder = 6,
    Parent = Content,
})
corner(ToggleBtn, 8)

local StatusLabel = makeLabel({
    Size = UDim2.new(1, 0, 0, 26),
    BackgroundColor3 = Color3.fromRGB(40, 40, 55),
    BackgroundTransparency = 0,
    Text = "   Status: OFF",
    TextColor3 = Color3.fromRGB(180, 180, 200),
    TextSize = 12,
    LayoutOrder = 7,
    Parent = Content,
})
corner(StatusLabel, 6)

for i, gear in ipairs(GEARS) do
    local item = makeButton({
        Size = UDim2.new(1, 0, 0, 30),
        BackgroundColor3 = Color3.fromRGB(55, 55, 75),
        Text = gear.name,
        TextSize = 13,
        LayoutOrder = i,
        Parent = Dropdown,
    })
    corner(item, 4)

    item.MouseButton1Click:Connect(function()
        State.selectedGear = gear
        GearBtn.Text = gear.name .. "  ▼"
        Dropdown.Visible = false
        refreshInfo()
    end)
end

refreshInfo()

GearBtn.MouseButton1Click:Connect(function()
    Dropdown.Visible = not Dropdown.Visible
end)

UidRefreshBtn.MouseButton1Click:Connect(function()
    refreshUid(true)
end)

refreshUid(false)

-- ============================================================
-- CONTROLLER
-- ============================================================
local function setStatus(text, color)
    StatusLabel.Text = "   " .. text
    StatusLabel.TextColor3 = color or Color3.fromRGB(180, 180, 200)
end

local function start()
    if State.running then return end

    local chosen = refreshUid(true)
    if not chosen then
        setStatus("Gagal: GearAutoCrafter tidak ada", Color3.fromRGB(255, 120, 120))
        return
    end
    CurrentUid = State.uid

    State.running = true
    State.loopCount = 0

    ToggleBtn.BackgroundColor3 = Color3.fromRGB(200, 70, 70)
    ToggleBtn.Text = "⏸  STOP"
    setStatus("Status: RUNNING", Color3.fromRGB(120, 220, 150))

    State.thread = task.spawn(function()
        while State.running do
            local gear = State.selectedGear
            pcall(runCraft, gear)

            State.loopCount = State.loopCount + 1
            setStatus(
                "Status: RUNNING (" .. State.loopCount .. ") - " .. gear.name,
                Color3.fromRGB(120, 220, 150)
            )

            if State.loopCount % 5 == 0 then
                pcall(function()
                    local c = refreshUid(false)
                    if c then CurrentUid = c.uid end
                end)
            end

            -- HANYA DELAY LOOP YANG TERSISA
            task.wait(CONFIG.LoopDelay)
        end
    end)
end

local function stop()
    State.running = false
    if State.thread then
        pcall(task.cancel, State.thread)
        State.thread = nil
    end

    ToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 100)
    ToggleBtn.Text = "▶  START"
    setStatus("Status: OFF (Total: " .. State.loopCount .. ")")
end

-- ============================================================
-- EVENTS
-- ============================================================
ToggleBtn.MouseButton1Click:Connect(function()
    if State.running then stop() else start() end
end)

CloseBtn.MouseButton1Click:Connect(function()
    stop()
    ScreenGui:Destroy()
end)

local minimized = false
MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    Content.Visible = not minimized
    Main.Size = minimized
        and UDim2.new(0, 300, 0, 38)
        or  UDim2.new(0, 300, 0, 380)
    MinBtn.Text = minimized and "+" or "—"
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == CONFIG.ToggleKey then
        Main.Visible = not Main.Visible
    end
end)

print("[AutoCraft] Ready. UID GearAutoCrafter didapat otomatis. RightShift untuk toggle.")
