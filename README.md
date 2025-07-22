-- CIATIMATO-LX - INTERFACE TOTALMENTE REDESENHADA + FUNÃ‡Ã•ES ORIGINAIS + LX FLUTUANTE PB

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

-- CONFIGURAÃ‡ÃƒO
local Config = {
    Aimbot = {
        Enabled = false,
        FOV = 80,
        Smoothness = 0.3,
        IgnoreTeam = true,
    },
    ESP = {
        Enabled = false,
        Color = Color3.fromRGB(255, 255, 255),
        ShowDistance = true,
    }
}

local myTeam = LocalPlayer.Team
local ESPCache = {}

-- NOVA INTERFACE
local screenGui = Instance.new("ScreenGui", CoreGui)
screenGui.Name = "CIATIMATO_LX_GUI"
screenGui.ResetOnSpawn = false

-- Main Window Moderno
local mainUI = Instance.new("Frame", screenGui)
mainUI.Size = UDim2.new(0, 370, 0, 290)
mainUI.Position = UDim2.new(0.5, -185, 0.5, -145)
mainUI.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainUI.BackgroundTransparency = 0.15
mainUI.BorderSizePixel = 0
mainUI.Visible = false
local mainCorner = Instance.new("UICorner", mainUI)
mainCorner.CornerRadius = UDim.new(0, 18)

-- TÃ­tulo estilizado
local titleBar = Instance.new("TextLabel", mainUI)
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
titleBar.BackgroundTransparency = 0.6
titleBar.Text = "CIATIMATO-LX"
titleBar.TextColor3 = Color3.fromRGB(0, 0, 0)
titleBar.Font = Enum.Font.GothamBold
titleBar.TextSize = 22
local titleCorner = Instance.new("UICorner", titleBar)
titleCorner.CornerRadius = UDim.new(0, 12)

-- Painel de estatÃ­sticas moderno
local statsPanel = Instance.new("Frame", mainUI)
statsPanel.Size = UDim2.new(1, -32, 0, 56)
statsPanel.Position = UDim2.new(0, 16, 1, -66)
statsPanel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
statsPanel.BackgroundTransparency = 0.8
Instance.new("UICorner", statsPanel).CornerRadius = UDim.new(0, 12)

local statsText = Instance.new("TextLabel", statsPanel)
statsText.Size = UDim2.new(1, -18, 1, 0)
statsText.Position = UDim2.new(0, 9, 0, 0)
statsText.BackgroundTransparency = 1
statsText.TextColor3 = Color3.fromRGB(30, 30, 30)
statsText.Font = Enum.Font.Gotham
statsText.TextSize = 15
statsText.TextXAlignment = Enum.TextXAlignment.Left
statsText.Text = "Status: Interface carregada."

-- BotÃ£o LX flutuante (canto superior direito)
local iconBtn = Instance.new("TextButton", screenGui)
iconBtn.Size = UDim2.new(0, 54, 0, 54)
iconBtn.Position = UDim2.new(1, -72, 0, 16)
iconBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- branco
iconBtn.BackgroundTransparency = 0
iconBtn.BorderSizePixel = 0
Instance.new("UICorner", iconBtn).CornerRadius = UDim.new(1, 0)
iconBtn.Text = "LX"
iconBtn.TextColor3 = Color3.fromRGB(0,0,0) -- preto
iconBtn.Font = Enum.Font.GothamBold
iconBtn.TextSize = 26
iconBtn.MouseEnter:Connect(function() iconBtn.BackgroundColor3 = Color3.fromRGB(220, 220, 220) end)
iconBtn.MouseLeave:Connect(function() iconBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 255) end)
iconBtn.MouseButton1Click:Connect(function()
    mainUI.Visible = not mainUI.Visible
end)

-- BotÃµes minimalistas
local function createToggle(label, yPos, onToggle, default)
    local btn = Instance.new("TextButton", mainUI)
    btn.Size = UDim2.new(1, -32, 0, 34)
    btn.Position = UDim2.new(0, 16, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(255,255,255)
    btn.BackgroundTransparency = 0.6
    btn.TextColor3 = Color3.fromRGB(30,30,30)
    btn.Text = label .. ": " .. ((default or false) and "ON" or "OFF")
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 10)
    local state = default or false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = label .. ": " .. (state and "ON" or "OFF")
        onToggle(state)
    end)
end

local function createSlider(label, yPos, getValue, setValue, min, max, step)
    local sliderLabel = Instance.new("TextLabel", mainUI)
    sliderLabel.Size = UDim2.new(1, -32, 0, 22)
    sliderLabel.Position = UDim2.new(0, 16, 0, yPos)
    sliderLabel.Text = label .. ": " .. getValue()
    sliderLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
    sliderLabel.BackgroundTransparency = 1
    sliderLabel.Font = Enum.Font.Gotham
    sliderLabel.TextSize = 15
    sliderLabel.TextXAlignment = Enum.TextXAlignment.Left

    local btn = Instance.new("TextButton", mainUI)
    btn.Size = UDim2.new(1, -32, 0, 28)
    btn.Position = UDim2.new(0, 16, 0, yPos + 22)
    btn.BackgroundColor3 = Color3.fromRGB(255,255,255)
    btn.BackgroundTransparency = 0.6
    btn.Text = "Aumentar"
    btn.TextColor3 = Color3.fromRGB(30, 30, 30)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

    btn.MouseButton1Click:Connect(function()
        local val = getValue() + step
        if val > max then val = min end
        setValue(val)
        sliderLabel.Text = label .. ": " .. val
        if label == "FOV" and fovCircle then
            fovCircle.Radius = val
        end
    end)
end

-- Interface ativa
createToggle("Aimbot", 56, function(v) Config.Aimbot.Enabled = v if fovCircle then fovCircle.Visible = v end end)
createToggle("ESP", 96, function(v)
    Config.ESP.Enabled = v
    if v then UpdateAllESP() else ClearESP() end
end)
createSlider("FOV", 136, function() return Config.Aimbot.FOV end, function(v) Config.Aimbot.FOV = v if fovCircle then fovCircle.Radius = v end end, 40, 180, 20)

-- FOV VISUAL
local fovCircle = Drawing and Drawing.new("Circle") or nil
if fovCircle then
    fovCircle.Radius = Config.Aimbot.FOV
    fovCircle.Thickness = 2
    fovCircle.Filled = false
    fovCircle.Transparency = 0.5
    fovCircle.Color = Color3.fromRGB(90,100,255)
    fovCircle.Visible = false
end

-- ESP / Aimbot Functions
function CreateESP(player)
    if not player or not player.Character or player == LocalPlayer then return end
    RemoveESP(player)
    local character = player.Character
    local head = character:FindFirstChild("Head")
    if not head then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_"..player.Name
    highlight.OutlineColor = Config.ESP.Color
    highlight.FillColor = Config.ESP.Color
    highlight.FillTransparency = 0.7
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = character

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Distance_"..player.Name
    billboard.Size = UDim2.new(0, 120, 0, 34)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.Adornee = head
    billboard.AlwaysOnTop = true
    billboard.Parent = head

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Name = "NameLabel"
    nameLabel.Text = player.Name
    nameLabel.Size = UDim2.new(1, 0, 0.6, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.Parent = billboard

    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Name = "DistanceLabel"
    distanceLabel.Text = "0 studs"
    distanceLabel.Size = UDim2.new(1, 0, 0.4, 0)
    distanceLabel.Position = UDim2.new(0, 0, 0.6, 0)
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.TextColor3 = Color3.fromRGB(50, 50, 50)
    distanceLabel.TextScaled = true
    distanceLabel.Font = Enum.Font.Gotham
    distanceLabel.Parent = billboard

    ESPCache[player] = {highlight, billboard}
end

function RemoveESP(player)
    if ESPCache[player] then
        for _, obj in pairs(ESPCache[player]) do
            if obj and obj.Parent then obj:Destroy() end
        end
        ESPCache[player] = nil
    end
    -- Remove marcador de alvo
    if player and player.Character and player.Character:FindFirstChild("AIMBOT_MARK") then
        player.Character.AIMBOT_MARK:Destroy()
    end
end

function ClearESP()
    for player, _ in pairs(ESPCache) do RemoveESP(player) end
    ESPCache = {}
end

function UpdateAllESP()
    ClearESP()
    if Config.ESP.Enabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                CreateESP(player)
            end
        end
    end
end

function IsVisible(targetPart)
    if not targetPart or not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("Head") then return false end
    local origin = Camera.CFrame.Position
    local direction = (targetPart.Position - origin).Unit * (targetPart.Position - origin).Magnitude
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
    rayParams.IgnoreWater = true
    local result = workspace:Raycast(origin, direction, rayParams)
    if result and result.Instance and result.Instance:IsDescendantOf(targetPart.Parent) then
        return true
    end
    return false
end

local function GetClosestEnemyToMe()
    local closestPlayer, closestDist = nil, math.huge
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("Head") then return nil end
    local myPos = myChar.Head.Position
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= myTeam and player.Character then
            local head = player.Character:FindFirstChild("Head")
            local hum = player.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 and head then
                local dist = (head.Position - myPos).Magnitude
                if dist < Config.Aimbot.FOV and dist < closestDist then
                    closestPlayer = player
                    closestDist = dist
                end
            end
        end
    end
    return closestPlayer
end

-- Bola vermelha com borda preta no alvo do aimbot
local function MarkAimbotTarget(player)
    if player and player.Character and player.Character:FindFirstChild("AIMBOT_MARK") then
        player.Character.AIMBOT_MARK:Destroy()
    end
    if player and player.Character and player.Character:FindFirstChild("Head") then
        local head = player.Character.Head
        local mark = Instance.new("BillboardGui")
        mark.Name = "AIMBOT_MARK"
        mark.Adornee = head
        mark.Size = UDim2.new(0, 40, 0, 40)
        mark.StudsOffset = Vector3.new(0, 1.5, 0)
        mark.AlwaysOnTop = true
        mark.Parent = player.Character

        local border = Instance.new("Frame", mark)
        border.Size = UDim2.new(1, 0, 1, 0)
        border.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        border.BackgroundTransparency = 0
        border.BorderSizePixel = 0
        local borderCorner = Instance.new("UICorner", border)
        borderCorner.CornerRadius = UDim.new(1, 0)

        local ball = Instance.new("Frame", mark)
        ball.Size = UDim2.new(0.8, 0, 0.8, 0)
        ball.Position = UDim2.new(0.1, 0, 0.1, 0)
        ball.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        ball.BackgroundTransparency = 0.2
        ball.BorderSizePixel = 0
        local corner = Instance.new("UICorner", ball)
        corner.CornerRadius = UDim.new(1, 0)
    end
end

local lastMarked
RunService.RenderStepped:Connect(function()
    local aimbotStatus = "Desligado"
    local targetDistance = "N/A"
    local targetName = ""
    local visStatus = "Indefinido"

    if Config.Aimbot.Enabled then
        aimbotStatus = "Ligado"
        local target = GetClosestEnemyToMe()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head
            targetName = target.Name
            targetDistance = math.floor((head.Position - Camera.CFrame.Position).Magnitude).." studs"
            local isVisible = IsVisible(head)
            visStatus = isVisible and "VisÃ­vel" or "AtrÃ¡s de parede"
            if isVisible then
                local currentCFrame = Camera.CFrame
                local targetCFrame = CFrame.new(currentCFrame.Position, head.Position)
                Camera.CFrame = currentCFrame:Lerp(targetCFrame, Config.Aimbot.Smoothness)
            end
            if lastMarked and lastMarked ~= target then
                if lastMarked.Character and lastMarked.Character:FindFirstChild("AIMBOT_MARK") then
                    lastMarked.Character.AIMBOT_MARK:Destroy()
                end
            end
            MarkAimbotTarget(target)
            lastMarked = target
        else
            if lastMarked and lastMarked.Character and lastMarked.Character:FindFirstChild("AIMBOT_MARK") then
                lastMarked.Character.AIMBOT_MARK:Destroy()
            end
            lastMarked = nil
        end
    else
        if lastMarked and lastMarked.Character and lastMarked.Character:FindFirstChild("AIMBOT_MARK") then
            lastMarked.Character.AIMBOT_MARK:Destroy()
        end
        lastMarked = nil
    end

    statsText.Text = string.format(
        "Aimbot: %s\nAlvo: %s | DistÃ¢ncia: %s | Visibilidade: %s\nFOV: %d",
        aimbotStatus,
        targetName~="" and targetName or "N/A",
        targetDistance,
        visStatus,
        Config.Aimbot.FOV
    )

    if fovCircle then
        fovCircle.Visible = Config.Aimbot.Enabled and mainUI.Visible
        local mousePos = UIS:GetMouseLocation()
        fovCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
    end

    if Config.ESP.Enabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Head") then
        for player, espData in pairs(ESPCache) do
            if player.Character and player.Character:FindFirstChild("Head") then
                local distance = math.floor((player.Character.Head.Position - LocalPlayer.Character.Head.Position).Magnitude)
                if espData[2] and espData[2]:FindFirstChild("DistanceLabel") then
                    espData[2].DistanceLabel.Text = distance.." studs"
                end
            end
        end
    end
end)

LocalPlayer:GetPropertyChangedSignal("Team"):Connect(function()
    myTeam = LocalPlayer.Team
    UpdateAllESP()
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(0.5)
        if Config.ESP.Enabled and player ~= LocalPlayer then
            CreateESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveESP(player)
end)

print("[CIATIMATO-LX] Nova interface carregada com sucesso!")
