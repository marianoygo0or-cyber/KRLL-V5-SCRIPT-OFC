print("KRLL Iniciando...")

-- // SERVIÇOS DO ROBLOX
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting")
local MarketplaceService = game:GetService("MarketplaceService")
local HttpService = game:GetService("HttpService")
local Debris = game:GetService("Debris")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- // CRIAÇÃO DA TELA PRINCIPAL
if PlayerGui:FindFirstChild("KRLL_UI") then PlayerGui.KRLL_UI:Destroy() end
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "KRLL_UI"
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 999

local successGethui, gethuiParent = pcall(function() return gethui() end)
if successGethui and gethuiParent then
    ScreenGui.Parent = gethuiParent
else
    ScreenGui.Parent = PlayerGui
end

-- // FUNÇÃO PARA CRIAR UI
local function Create(className, properties, parent)
    local obj = Instance.new(className)
    for k, v in pairs(properties) do
        obj[k] = v
    end
    obj.Parent = parent
    return obj
end

-- ==========================================
-- // VARIÁVEIS E LÓGICA
-- ==========================================
local Toggles = {
    Noclip = false, ESP = false, AntiSit = false, GodMode = false, 
    AntiCollision = false, ReduzirLag = false, FPS = false, AutoRevive = false, 
    NoKnockback = false, InstaKill = false, AntiDetect = false, AntiRubberband = false, 
    SuperPunch = false, NoFall = false
}
local Speed, Jump = 16, 50
local UI_Open = false
local LastRevive = 0
local IsRunning = true
local ESP_Highlights = {}

local ColorTargets = {}
local StrokeTargets = {}

local function SetColor(color)
    for _, obj in ipairs(ColorTargets) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") then obj.TextColor3 = color
        elseif obj:IsA("Frame") then obj.BackgroundColor3 = color end
    end
    for _, stroke in ipairs(StrokeTargets) do stroke.Color = color end
end

local function AddGlow(textLabel)
    local stroke = Instance.new("UIStroke", textLabel)
    stroke.Thickness = 2
    stroke.Transparency = 0.5
    stroke.Color = textLabel.TextColor3 
    stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Contextual
    table.insert(StrokeTargets, stroke)
    return stroke
end

if isfile and isfile("KRLL_Config.json") then
    local success, data = pcall(function() return HttpService:JSONDecode(readfile("KRLL_Config.json")) end)
    if success and data then Speed = data.Sp or 16 Jump = data.Ju or 50 end
end

local MapName = "Mapa"
local successMap, info = pcall(function() return MarketplaceService:GetProductInfo(game.PlaceId).Name end)
if successMap and info then MapName = info end
local AvatarImg = "rbxthumb://type=Avatar&id="..LocalPlayer.UserId.."&w=420&h=420"

-- ==========================================
-- // INTERFACE DO MENU
-- ==========================================
local MinimizedButton = Create("TextButton", {Size=UDim2.fromOffset(380,30), Position=UDim2.new(.5,-190,-.1,0), BackgroundColor3=Color3.fromRGB(40,40,40), BackgroundTransparency=.4, Text="", AutoButtonColor=false, Visible=false, ZIndex=999}, ScreenGui)
Create("UICorner", {CornerRadius=UDim.new(0,8)}, MinimizedButton)
Create("UIStroke", {Color=Color3.fromRGB(80,80,80), Thickness=2, ApplyStrokeMode=Enum.ApplyStrokeMode.Border}, MinimizedButton)

local MinV5Text = Create("TextLabel", {Size=UDim2.fromOffset(50,30), Position=UDim2.fromOffset(15,0), BackgroundTransparency=1, Text="V5", TextColor3=Color3.fromRGB(255,50,50), Font=Enum.Font.GothamBold, TextSize=14, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=1000}, MinimizedButton)
table.insert(ColorTargets, MinV5Text)
AddGlow(MinV5Text)

Create("TextLabel", {Size=UDim2.fromScale(1,1), BackgroundTransparency=1, Text="KRLL", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=14, ZIndex=1000}, MinimizedButton)
Create("TextLabel", {Size=UDim2.fromOffset(40,30), Position=UDim2.new(1,-50,0,0), BackgroundTransparency=1, Text="+", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=20, TextXAlignment=Enum.TextXAlignment.Right, ZIndex=1000}, MinimizedButton)

local MainFrame = Create("Frame", {Size=UDim2.fromOffset(460,280), Position=UDim2.new(.5,-230,.5,-140), BackgroundColor3=Color3.fromRGB(35,35,35), BackgroundTransparency=.15, Visible=false, ClipsDescendants=true, ZIndex=10}, ScreenGui)
Create("UICorner", {CornerRadius=UDim.new(0,12)}, MainFrame)
Create("UIStroke", {Color=Color3.fromRGB(80,80,80)}, MainFrame)

local TopBar = Create("Frame", {Size=UDim2.new(1,0,0,35), BackgroundTransparency=1, ZIndex=11}, MainFrame)
local TopV5Text = Create("TextLabel", {Size=UDim2.fromOffset(50,35), Position=UDim2.fromOffset(15,0), BackgroundTransparency=1, Text="V5", TextColor3=Color3.fromRGB(255,50,50), Font=Enum.Font.GothamBold, TextSize=18, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, TopBar)
table.insert(ColorTargets, TopV5Text)
AddGlow(TopV5Text)
Create("TextLabel", {Size=UDim2.fromScale(1,1), BackgroundTransparency=1, Text="KRLL", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=20, ZIndex=12}, TopBar)

local MinButton = Create("TextButton", {Size=UDim2.fromOffset(35,35), Position=UDim2.new(1,-75,0,0), BackgroundTransparency=1, Text="-", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=24, ZIndex=12}, TopBar)
local CloseButton = Create("TextButton", {Size=UDim2.fromOffset(35,35), Position=UDim2.new(1,-40,0,0), BackgroundTransparency=1, Text="X", TextColor3=Color3.fromRGB(255,50,50), Font=Enum.Font.GothamBold, TextSize=22, ZIndex=12}, TopBar)
table.insert(ColorTargets, CloseButton)

local Sidebar = Create("Frame", {Size=UDim2.new(0,115,1,-50), Position=UDim2.fromOffset(15,40), BackgroundColor3=Color3.fromRGB(25,25,25), BackgroundTransparency=.2, ZIndex=11}, MainFrame)
Create("UICorner", {CornerRadius=UDim.new(0,8)}, Sidebar)
local TabScroll = Create("ScrollingFrame", {Size=UDim2.new(1,0,1,-105), BackgroundTransparency=1, ScrollBarThickness=0, CanvasSize=UDim2.fromOffset(0,250), ZIndex=12}, Sidebar)

local function CreateTab(text, y)
    local btn = Create("TextButton", {Size=UDim2.new(1,-20,0,30), Position=UDim2.fromOffset(10,y), BackgroundColor3=Color3.fromRGB(60,60,60), Text=text, TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=11, ZIndex=13}, TabScroll)
    Create("UICorner", {CornerRadius=UDim.new(0,6)}, btn)
    return btn
end

local TabMain = CreateTab("PRINCIPAL", 10)
local TabCombat = CreateTab("COMBATE", 45)
local TabESP = CreateTab("ESP", 80)
local TabAnti = CreateTab("ANTI XIT", 115)
local TabSound = CreateTab("SOM ALL", 150)
local TabConfig = CreateTab("CONFIG", 185)

local ProfileFrame = Create("Frame", {Size=UDim2.new(1,-20,0,100), Position=UDim2.new(0,10,1,-105), BackgroundTransparency=1, ZIndex=12}, Sidebar)
Create("TextLabel", {Size=UDim2.new(1,0,0,15), BackgroundTransparency=1, Text="HI "..LocalPlayer.Name, TextColor3=Color3.fromRGB(0,255,0), Font=Enum.Font.GothamBold, TextSize=11, ZIndex=13}, ProfileFrame)
Create("TextLabel", {Size=UDim2.new(1,0,0,15), Position=UDim2.fromOffset(0,16), BackgroundTransparency=1, Text="Mapa: "..MapName, TextColor3=Color3.fromRGB(0,255,0), Font=Enum.Font.GothamBold, TextSize=11, ZIndex=13, TextTruncate=Enum.TextTruncate.AtEnd}, ProfileFrame)
Create("TextLabel", {Size=UDim2.new(1,0,0,15), Position=UDim2.fromOffset(0,32), BackgroundTransparency=1, Text="Online", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=11, ZIndex=13}, ProfileFrame)

local AvatarFrame = Create("Frame", {Size=UDim2.fromOffset(95,45), Position=UDim2.fromOffset(0,50), BackgroundColor3=Color3.new(1,1,1), ZIndex=13, ClipsDescendants=true}, ProfileFrame)
Create("UICorner", {CornerRadius=UDim.new(0,4)}, AvatarFrame)
Create("ImageLabel", {Size=UDim2.fromScale(1,1), BackgroundTransparency=1, ScaleType=Enum.ScaleType.Fit, ZIndex=14, Image=AvatarImg}, AvatarFrame)

local ContentArea = Create("ScrollingFrame", {Size=UDim2.new(1,-160,1,-50), Position=UDim2.fromOffset(145,40), BackgroundColor3=Color3.fromRGB(25,25,25), BackgroundTransparency=.2, ClipsDescendants=true, ZIndex=11, ScrollBarThickness=3, ScrollBarImageColor3=Color3.fromRGB(150,150,150), CanvasSize=UDim2.fromOffset(0,600)}, MainFrame)
Create("UICorner", {CornerRadius=UDim.new(0,8)}, ContentArea)

local FPSLabel = Create("TextLabel", {Size=UDim2.fromOffset(80,25), Position=UDim2.new(1,-120,0,150), BackgroundTransparency=.4, BackgroundColor3=Color3.new(0,0,0), TextColor3=Color3.fromRGB(0,255,0), Font=Enum.Font.GothamBold, TextSize=12, Text="FPS: 60", Visible=false, ZIndex=999}, ScreenGui)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, FPSLabel)
local FPSCount, FPSTime = 0, tick()
RunService.RenderStepped:Connect(function()
    if not IsRunning then return end
    FPSCount = FPSCount + 1
    if tick() - FPSTime >= 1 then FPSLabel.Text = "FPS: "..FPSCount FPSCount = 0 FPSTime = tick() end
end)

local ActiveSlider = nil
UserInputService.InputChanged:Connect(function(input)
    if ActiveSlider and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
        local sizeX = math.clamp((input.Position.X-ActiveSlider.Bar.AbsolutePosition.X)/ActiveSlider.Bar.AbsoluteSize.X, 0, 1)
        ActiveSlider.Fill.Size = UDim2.new(sizeX, 0, 1, 0)
        local val = math.floor(ActiveSlider.Min + (ActiveSlider.Max-ActiveSlider.Min)*sizeX)
        ActiveSlider.Label.Text = ActiveSlider.Text..": "..val
        ActiveSlider.Callback(val)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then ActiveSlider = nil end
end)

local function CreateToggle(parent, title, desc, y, callback)
    local frame = Create("Frame", {Size=UDim2.new(1,-20,0,40), Position=UDim2.fromOffset(10,y), BackgroundTransparency=1, ZIndex=12}, parent)
    Create("TextLabel", {Size=UDim2.new(1,-40,0,18), BackgroundTransparency=1, Text=title, TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, frame)
    Create("TextLabel", {Size=UDim2.new(1,-40,0,15), Position=UDim2.fromOffset(0,18), BackgroundTransparency=1, Text=desc, TextColor3=Color3.fromRGB(180,180,180), Font=Enum.Font.Gotham, TextSize=10, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, frame)
    local switch = Create("TextButton", {Size=UDim2.fromOffset(36,18), Position=UDim2.new(1,-36,.5,-9), BackgroundColor3=Color3.fromRGB(80,80,80), Text="", ZIndex=13}, frame)
    Create("UICorner", {CornerRadius=UDim.new(1,0)}, switch)
    local circle = Create("Frame", {Size=UDim2.fromOffset(14,14), Position=UDim2.new(0,2,.5,-7), BackgroundColor3=Color3.new(1,1,1), ZIndex=14}, switch)
    Create("UICorner", {CornerRadius=UDim.new(1,0)}, circle)
    local state = false
    switch.MouseButton1Click:Connect(function()
        state = not state
        TweenService:Create(switch, TweenInfo.new(.2), {BackgroundColor3=state and Color3.fromRGB(0,200,0) or Color3.fromRGB(80,80,80)}):Play()
        TweenService:Create(circle, TweenInfo.new(.2), {Position=state and UDim2.new(1,-16,.5,-7) or UDim2.new(0,2,.5,-7)}):Play()
        callback(state)
    end)
end

local function CreateSlider(parent, title, y, min, max, default, callback)
    local frame = Create("Frame", {Size=UDim2.new(1,-20,0,45), Position=UDim2.fromOffset(10,y), BackgroundTransparency=1, ZIndex=12}, parent)
    local label = Create("TextLabel", {Size=UDim2.new(1,0,0,15), BackgroundTransparency=1, Text=title..": "..default, TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, frame)
    local bar = Create("Frame", {Size=UDim2.new(1,0,0,8), Position=UDim2.fromOffset(0,20), BackgroundColor3=Color3.fromRGB(80,80,80), ZIndex=13}, frame)
    Create("UICorner", {CornerRadius=UDim.new(1,0)}, bar)
    local fill = Create("Frame", {Size=UDim2.new((default-min)/(max-min),0,1,0), BackgroundColor3=Color3.fromRGB(220,20,20), ZIndex=14}, bar)
    Create("UICorner", {CornerRadius=UDim.new(1,0)}, fill)
    table.insert(ColorTargets, fill)
    bar.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            ActiveSlider = {Bar=bar, Fill=fill, Label=label, Min=min, Max=max, Text=title, Callback=callback}
            local sizeX = math.clamp((input.Position.X-bar.AbsolutePosition.X)/bar.AbsoluteSize.X, 0, 1)
            fill.Size = UDim2.new(sizeX, 0, 1, 0)
            local val = math.floor(min + (max-min)*sizeX)
            label.Text = title..": "..val
            callback(val)
        end
    end)
end

local PageMain = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, ZIndex=12}, ContentArea)
CreateSlider(PageMain, "Velocidade", 0, 16, 200, Speed, function(v) Speed=v end)
CreateSlider(PageMain, "Pulo", 50, 50, 200, Jump, function(v) Jump=v end)
CreateToggle(PageMain, "SEM DANO DE QUEDA", "Não se machuca ao cair", 100, function(s) Toggles.NoFall=s end)
CreateToggle(PageMain, "NOCLIP", "Atravessa paredes", 140, function(s) Toggles.Noclip=s end)
CreateToggle(PageMain, "REDUZIR LAG", "Desliga efeitos gráficos", 180, function(s)
    Toggles.ReduzirLag=s
    if s then
        Lighting.GlobalShadows=false settings().Rendering.QualityLevel=1
        for _,v in pairs(workspace:GetDescendants())do if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") then v.Enabled=false end end
    else
        Lighting.GlobalShadows=true settings().Rendering.QualityLevel=21
    end
end)
CreateToggle(PageMain, "FPS CONTADOR", "Mostra os quadros por segundo", 220, function(s) Toggles.FPS=s FPSLabel.Visible=s end)
local FlyButton = Create("TextButton", {Size=UDim2.new(1,0,0,30), Position=UDim2.fromOffset(0,260), BackgroundColor3=Color3.fromRGB(30,30,30), Text="EXECUTAR FLY", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=11, ZIndex=13}, PageMain)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, FlyButton)
FlyButton.MouseButton1Click:Connect(function() pcall(function() loadstring(game:HttpGet("https://scriptblox.com/raw/Universal-Script-Fly-gui-v3-78856"))() end) end)

local PageCombat = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, Visible=false, ZIndex=12}, ContentArea)
CreateToggle(PageCombat, "SUPER SOCO", "Empurra jogadores com força máxima", 10, function(s) Toggles.SuperPunch=s end)
CreateToggle(PageCombat, "INSTAKILL", "Dano máximo em armas", 50, function(s) Toggles.InstaKill=s end)

local PageESP = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, Visible=false, ZIndex=12}, ContentArea)
CreateToggle(PageESP, "ESP JOGADORES", "Destaca jogadores com Barra de Vida", 10, function(s) Toggles.ESP=s end)

local PageAnti = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, Visible=false, ZIndex=12}, ContentArea)
CreateToggle(PageAnti, "ANTI SENTAR", "Impede de ser forçado a sentar", 0, function(s) Toggles.AntiSit=s end)
CreateToggle(PageAnti, "GOD MODE", "Vida infinita e impede a morte", 40, function(s) Toggles.GodMode=s end)
CreateToggle(PageAnti, "ANTI COLISÃO", "Atravessa outros jogadores", 80, function(s) Toggles.AntiCollision=s end)
CreateToggle(PageAnti, "AUTO-REVIVER", "Respawn automático ao morrer", 120, function(s) Toggles.AutoRevive=s end)
CreateToggle(PageAnti, "SEM KNOCKBACK", "Não é empurrado ao levar dano", 160, function(s) Toggles.NoKnockback=s end)
CreateToggle(PageAnti, "ANTI-DETECT", "Esconde a UI de anti-cheat", 200, function(s) Toggles.AntiDetect=s end)
CreateToggle(PageAnti, "ANTI-RUBBERBAND", "Evita ser puxado pelo servidor", 240, function(s) Toggles.AntiRubberband=s end)

local PageSound = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, Visible=false, ZIndex=12}, ContentArea)
Create("TextLabel", {Size=UDim2.new(1,-20,0,20), Position=UDim2.fromOffset(10,10), BackgroundTransparency=1, Text="Digite o ID de Áudio:", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, PageSound)
local IDInput = Create("TextBox", {Size=UDim2.new(1,-20,0,30), Position=UDim2.fromOffset(10,35), BackgroundColor3=Color3.fromRGB(60,60,60), Text="", PlaceholderText="Ex: 1837879082", TextColor3=Color3.new(1,1,1), Font=Enum.Font.Gotham, TextSize=14, ZIndex=13, ClearTextOnFocus=false}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, IDInput)
local PlayBtn = Create("TextButton", {Size=UDim2.new(0.5,-15,0,30), Position=UDim2.fromOffset(10,75), BackgroundColor3=Color3.fromRGB(0,200,0), Text="TOCAR ID", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, ZIndex=13}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, PlayBtn)
local StopBtn = Create("TextButton", {Size=UDim2.new(0.5,-15,0,30), Position=UDim2.new(0.5,5,0,75), BackgroundColor3=Color3.fromRGB(200,50,50), Text="PARAR", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, ZIndex=13}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, StopBtn)
Create("Frame", {Size=UDim2.new(1,-20,0,2), Position=UDim2.fromOffset(10,120), BackgroundColor3=Color3.fromRGB(80,80,80), ZIndex=12}, PageSound)
Create("TextLabel", {Size=UDim2.new(1,-20,0,20), Position=UDim2.fromOffset(10,130), BackgroundTransparency=1, Text="Salvar Áudio (ID e Bio):", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=12}, PageSound)
local SaveIDInput = Create("TextBox", {Size=UDim2.new(0.5,-15,0,30), Position=UDim2.fromOffset(10,155), BackgroundColor3=Color3.fromRGB(60,60,60), Text="", PlaceholderText="ID", TextColor3=Color3.new(1,1,1), Font=Enum.Font.Gotham, TextSize=14, ZIndex=13, ClearTextOnFocus=false}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, SaveIDInput)
local SaveBioInput = Create("TextBox", {Size=UDim2.new(0.5,-15,0,30), Position=UDim2.new(0.5,5,0,155), BackgroundColor3=Color3.fromRGB(60,60,60), Text="", PlaceholderText="Bio", TextColor3=Color3.new(1,1,1), Font=Enum.Font.Gotham, TextSize=14, ZIndex=13, ClearTextOnFocus=false}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, SaveBioInput)
local SaveBtn = Create("TextButton", {Size=UDim2.new(1,-20,0,30), Position=UDim2.fromOffset(10,195), BackgroundColor3=Color3.fromRGB(50,150,255), Text="SALVAR ÁUDIO", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, ZIndex=13}, PageSound)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, SaveBtn)
local SavedList = Create("ScrollingFrame", {Size=UDim2.new(1,-20,0,150), Position=UDim2.fromOffset(10,235), BackgroundTransparency=1, ScrollBarThickness=2, CanvasSize=UDim2.fromOffset(0,0), ZIndex=12}, PageSound)

local PageConfig = Create("Frame", {Size=UDim2.new(1,-10,1,0), BackgroundTransparency=1, Visible=false, ZIndex=12}, ContentArea)
CreateSlider(PageConfig, "FOV (Zoom)", 0, 70, 120, 70, function(v) workspace.CurrentCamera.FieldOfView=v end)
CreateSlider(PageConfig, "Transparência", 45, 0, 100, 15, function(v) MainFrame.BackgroundTransparency=v/100 end)
CreateToggle(PageConfig, "SALVAR CONFIGURAÇÕES", "Guarda suas escolhas no celular", 90, function(s)
    if writefile and s then local data = {Sp = Speed, Ju = Jump} writefile("KRLL_Config.json", HttpService:JSONEncode(data)) end
end)
local ColorButton = Create("TextButton", {Size=UDim2.new(1,-20,0,30), Position=UDim2.fromOffset(10,130), BackgroundColor3=Color3.fromRGB(60,60,60), Text="MUDAR COR", TextColor3=Color3.new(1,1,1), Font=Enum.Font.GothamBold, TextSize=12, ZIndex=12}, PageConfig)
Create("UICorner", {CornerRadius=UDim.new(0,6)}, ColorButton)
local PaletteFrame = Create("Frame", {Size=UDim2.fromOffset(260, 75), Position=UDim2.fromOffs
