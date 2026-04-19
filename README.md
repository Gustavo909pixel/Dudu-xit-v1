local player = game.Players.LocalPlayer
local players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local camera = workspace.CurrentCamera

-- ============ CONFIGURAÇÕES INICIAIS ============
local Settings = {
    TextRGB = true,
    ESPRGB = true,
    HitboxRGB = true,
    MenuRGB = true,
    MenuColor = Color3.fromRGB(0, 0, 0),
    TextColor = Color3.fromRGB(255, 255, 255),
    ESPPlayerColor = Color3.fromRGB(255, 255, 255),
    ESPHealthColor = Color3.fromRGB(0, 255, 0),
    HitboxSizeValue = 10, -- Valor numérico da Hitbox
    TPDistance = 15
}

local espAtivo, mostrarVida, hitboxAtiva, puloInfinito, tpFrAtivo = false, false, false, false, false

-- ============ FUNÇÕES AUXILIARES ============
local function isEnemy(plr)
    if not plr or plr == player then return false end
    if player.Team ~= nil and plr.Team == player.Team then return false end
    return true
end

local function criarObjetosESP(plr)
    if plr == player then return end
    local function aplicar(char)
        local root = char:WaitForChild("HumanoidRootPart", 5)
        if not root then return end
        if not root:FindFirstChild("ESP_BOX") then
            local box = Instance.new("BillboardGui", root)
            box.Name = "ESP_BOX"; box.Size = UDim2.new(4, 0, 5.5, 0); box.AlwaysOnTop = true
            local f = Instance.new("Frame", box); f.Size = UDim2.new(1, 0, 1, 0); f.BackgroundTransparency = 1
            local s = Instance.new("UIStroke", f); s.Thickness = 1.5; s.Color = Settings.ESPPlayerColor
        end
        if not root:FindFirstChild("HP_BAR") then
            local hpGui = Instance.new("BillboardGui", root)
            hpGui.Name = "HP_BAR"; hpGui.Size = UDim2.new(3, 0, 0.3, 0); hpGui.StudsOffset = Vector3.new(0, -3.5, 0); hpGui.AlwaysOnTop = true
            local bg = Instance.new("Frame", hpGui); bg.Size = UDim2.new(1, 0, 1, 0); bg.BackgroundColor3 = Color3.new(0, 0, 0)
            local bar = Instance.new("Frame", bg); bar.Name = "BAR"; bar.Size = UDim2.new(1, 0, 1, 0); bar.BackgroundColor3 = Settings.ESPHealthColor; bar.BorderSizePixel = 0
        end
    end
    if plr.Character then aplicar(plr.Character) end
    plr.CharacterAdded:Connect(aplicar)
end

-- ============ INTERFACE ============
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.ResetOnSpawn = false
gui.Name = "DUDUXIT_V1"

local open = Instance.new("TextButton", gui)
open.Name = "XIT"; open.Size = UDim2.new(0,60,0,60); open.Position = UDim2.new(0,15,0,15); open.Text = "XIT"
open.BackgroundColor3 = Color3.fromRGB(0, 0, 0); open.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", open).CornerRadius = UDim.new(1,0)

local tpBtnFloating = Instance.new("TextButton", gui)
tpBtnFloating.Name = "TP_FR_FLOAT"; tpBtnFloating.Size = UDim2.new(0, 80, 0, 45); tpBtnFloating.Position = UDim2.new(1, -100, 0, 20)
tpBtnFloating.Text = "TP FR"; tpBtnFloating.BackgroundColor3 = Color3.fromRGB(0,0,0); tpBtnFloating.TextColor3 = Color3.new(1,1,1)
tpBtnFloating.Visible = false; Instance.new("UICorner", tpBtnFloating)
local tpStroke = Instance.new("UIStroke", tpBtnFloating); tpStroke.Color = Color3.new(1,1,1); tpStroke.Thickness = 2

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0,320,0,400); frame.Position = UDim2.new(0.5,-160,0.3,0)
frame.BackgroundColor3 = Settings.MenuColor; frame.Visible = false; frame.Active = true
Instance.new("UICorner", frame)
local mainStroke = Instance.new("UIStroke", frame); mainStroke.Thickness = 2.5; mainStroke.Color = Color3.fromRGB(255, 255, 255)

local title = Instance.new("TextLabel", frame)
title.Text = "DUDU XIT V1"; title.Size = UDim2.new(1, 0, 0, 30); title.Position = UDim2.new(0, 0, 0, 5)
title.BackgroundTransparency = 1; title.Font = Enum.Font.GothamBold; title.TextSize = 18

local close = Instance.new("TextButton", frame)
close.Size = UDim2.new(0,30,0,30); close.Position = UDim2.new(1,-35,0,5); close.Text = "X"
close.BackgroundColor3 = Color3.fromRGB(30, 30, 30); close.TextColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", close).CornerRadius = UDim.new(0.5,0)

local content = Instance.new("Frame", frame)
content.Size = UDim2.new(1,0,1,-80); content.Position = UDim2.new(0,0,0,80); content.BackgroundTransparency = 1

local function createTab(name, pos)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0, 95, 0, 30); btn.Position = UDim2.new(pos, 0, 0, 40)
    btn.Text = name; btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25); btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 12; Instance.new("UICorner", btn)
    local page = Instance.new("ScrollingFrame", content)
    page.Size = UDim2.new(1,0,1,0); page.BackgroundTransparency = 1; page.Visible = false; page.ScrollBarThickness = 0
    local layout = Instance.new("UIListLayout", page); layout.Padding = UDim.new(0,8); layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    return btn, page
end

local espTab, espPage = createTab("ESP", 0.03)
local mainTab, mainPage = createTab("HACKS", 0.35)
local configTab, configPage = createTab("CONFIG", 0.67)

local function createBtn(parent, text)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(0.9,0,0,35); b.Text = text; b.BackgroundColor3 = Color3.fromRGB(20, 20, 20); b.TextColor3 = Color3.fromRGB(255, 255, 255)
    Instance.new("UICorner", b)
    return b
end

-- Função para criar caixa de entrada de texto
local function createInput(parent, placeholder, default)
    local box = Instance.new("TextBox", parent)
    box.Size = UDim2.new(0.9, 0, 0, 35)
    box.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    box.TextColor3 = Color3.new(1, 1, 1)
    box.PlaceholderText = placeholder
    box.Text = default
    box.Font = Enum.Font.Gotham
    box.TextSize = 14
    Instance.new("UICorner", box)
    return box
end

-- BOTÕES E INPUTS
local espBtn = createBtn(espPage, "ESP: OFF")
local hpBtn = createBtn(espPage, "ESP VIDA: OFF")

local hitboxBtn = createBtn(mainPage, "HITBOX: OFF")
local hitboxInput = createInput(mainPage, "TAMANHO HITBOX (NÚMERO)", "10")

local jumpBtn = createBtn(mainPage, "PULO INFINITO: OFF")

local tpFrBtn = createBtn(mainPage, "TP FR: OFF")
local tpInput = createInput(mainPage, "DISTÂNCIA TP (NÚMERO)", "15")

local rgbMenuBtn = createBtn(configPage, "MENU RGB: ON")
local rgbHitboxBtn = createBtn(configPage, "HITBOX RGB: ON")
local rgbTextBtn = createBtn(configPage, "TEXTO RGB: ON")
local rgbESPBtn = createBtn(configPage, "ESP RGB: ON")

-- ============ LÓGICA ============

-- Atualizar TPDistance via Input
tpInput.FocusLost:Connect(function(enter)
    local val = tonumber(tpInput.Text)
    if val then Settings.TPDistance = val else tpInput.Text = Settings.TPDistance end
end)

-- Atualizar HitboxSize via Input
hitboxInput.FocusLost:Connect(function(enter)
    local val = tonumber(hitboxInput.Text)
    if val then Settings.HitboxSizeValue = val else hitboxInput.Text = Settings.HitboxSizeValue end
end)

tpBtnFloating.MouseButton1Click:Connect(function()
    local char = player.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame * CFrame.new(0, 0, -Settings.TPDistance)
    end
end)

UIS.JumpRequest:Connect(function()
    if puloInfinito and player.Character then
        player.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

RunService.RenderStepped:Connect(function()
    local rainbow = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
    
    if Settings.MenuRGB then mainStroke.Color = rainbow end
    if tpFrAtivo then tpStroke.Color = rainbow end

    for _, plr in pairs(players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local root = plr.Character:FindFirstChild("HumanoidRootPart")
            local hum = plr.Character:FindFirstChild("Humanoid")
            if root then
                local isEnemyTarget = isEnemy(plr)
                
                if hitboxAtiva and isEnemyTarget then
                    local s = Settings.HitboxSizeValue
                    root.Size = Vector3.new(s, s, s)
                    root.Transparency = 0.5
                    root.Color = (Settings.HitboxRGB and rainbow or Color3.fromRGB(255,255,255))
                    root.CanCollide = false
                else
                    root.Size = Vector3.new(2, 2, 1); root.Transparency = 1; root.CanCollide = true
                end
                
                local box = root:FindFirstChild("ESP_BOX")
                if box then 
                    box.Enabled = espAtivo and isEnemyTarget
                    box.Frame.UIStroke.Color = Settings.ESPRGB and rainbow or Settings.ESPPlayerColor
                end

                local hp = root:FindFirstChild("HP_BAR")
                if hp then
                    hp.Enabled = espAtivo and mostrarVida and isEnemyTarget
                    if hp.Enabled and hum then
                        hp.Frame.BAR.Size = UDim2.new(math.clamp(hum.Health/hum.MaxHealth, 0, 1), 0, 1, 0)
                        hp.Frame.BAR.BackgroundColor3 = Settings.ESPRGB and rainbow or Settings.ESPHealthColor
                    end
                end
            end
        end
    end
end)

task.spawn(function()
    while true do
        local rainbow = Color3.fromHSV(tick() % 5 / 5, 0.8, 1)
        for _, v in pairs(gui:GetDescendants()) do
            if (v:IsA("TextButton") or v:IsA("TextLabel")) and v.Name ~= "XIT" and v.Name ~= "TP_FR_FLOAT" then
                v.TextColor3 = Settings.TextRGB and rainbow or Settings.TextColor
            end
        end
        task.wait()
    end
end)

-- ============ EVENTOS BOTÕES ============
espTab.MouseButton1Click:Connect(function() espPage.Visible = true; mainPage.Visible = false; configPage.Visible = false end)
mainTab.MouseButton1Click:Connect(function() espPage.Visible = false; mainPage.Visible = true; configPage.Visible = false end)
configTab.MouseButton1Click:Connect(function() espPage.Visible = false; mainPage.Visible = false; configPage.Visible = true end)

espBtn.MouseButton1Click:Connect(function() 
    espAtivo = not espAtivo; espBtn.Text = "ESP: "..(espAtivo and "ON" or "OFF")
    if espAtivo then for _,p in pairs(players:GetPlayers()) do criarObjetosESP(p) end end 
end)

hpBtn.MouseButton1Click:Connect(function() 
    mostrarVida = not mostrarVida; hpBtn.Text = "ESP VIDA: "..(mostrarVida and "ON" or "OFF") 
end)

hitboxBtn.MouseButton1Click:Connect(function() 
    hitboxAtiva = not hitboxAtiva; hitboxBtn.Text = "HITBOX: "..(hitboxAtiva and "ON" or "OFF") 
end)

jumpBtn.MouseButton1Click:Connect(function() 
    puloInfinito = not puloInfinito; jumpBtn.Text = "PULO INFINITO: "..(puloInfinito and "ON" or "OFF") 
end)

tpFrBtn.MouseButton1Click:Connect(function()
    tpFrAtivo = not tpFrAtivo
    tpFrBtn.Text = "TP FR: "..(tpFrAtivo and "ON" or "OFF")
    tpBtnFloating.Visible = tpFrAtivo
end)

rgbMenuBtn.MouseButton1Click:Connect(function() Settings.MenuRGB = not Settings.MenuRGB; rgbMenuBtn.Text = "MENU RGB: "..(Settings.MenuRGB and "ON" or "OFF") end)
rgbHitboxBtn.MouseButton1Click:Connect(function() Settings.HitboxRGB = not Settings.HitboxRGB; rgbHitboxBtn.Text = "HITBOX RGB: "..(Settings.HitboxRGB and "ON" or "OFF") end)
rgbTextBtn.MouseButton1Click:Connect(function() Settings.TextRGB = not Settings.TextRGB; rgbTextBtn.Text = "TEXTO RGB: "..(Settings.TextRGB and "ON" or "OFF") end)
rgbESPBtn.MouseButton1Click:Connect(function() Settings.ESPRGB = not Settings.ESPRGB; rgbESPBtn.Text = "ESP RGB: "..(Settings.ESPRGB and "ON" or "OFF") end)

open.MouseButton1Click:Connect(function() frame.Visible = true; open.Visible = false end)
close.MouseButton1Click:Connect(function() frame.Visible = false; open.Visible = true end)

-- ARRASTAR MENU
local drag, dStart, sPos; frame.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = true; dStart = i.Position; sPos = frame.Position end end)
UIS.InputChanged:Connect(function(i) if drag and (i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch) then local delta = i.Position - dStart; frame.Position = UDim2.new(sPos.X.Scale, sPos.X.Offset + delta.X, sPos.Y.Scale, sPos.Y.Offset + delta.Y) end end)
UIS.InputEnded:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then drag = false end end)

espPage.Visible = true
