--[[ 
    Sistema de Testes Roblox - Mobile
    ESP + Aimbot + Config Menu (Botão AHG)
    Por ChatGPT
]]

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Configurações
local ESP_ENABLED = true
local AIMBOT_ENABLED = false
local MENU_OPEN = false
local TEAM_CHECK = true

-- Config ESP
local BOX_COLOR = Color3.fromRGB(255, 0, 0)

-- Tabelas
local Drawings = {}

-- Funções ESP
local function CreateESP(player)
    Drawings[player] = {
        Box = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        HealthBar = Drawing.new("Line")
    }
    
    local d = Drawings[player]

    d.Box.Visible = false
    d.Box.Color = BOX_COLOR
    d.Box.Thickness = 2
    d.Box.Transparency = 1
    d.Box.Filled = false

    d.Name.Visible = false
    d.Name.Color = Color3.fromRGB(255, 255, 255)
    d.Name.Size = 16
    d.Name.Center = true
    d.Name.Outline = true

    d.HealthBar.Visible = false
    d.HealthBar.Thickness = 2
end

local function RemoveESP(player)
    if Drawings[player] then
        for _, obj in pairs(Drawings[player]) do
            obj:Remove()
        end
        Drawings[player] = nil
    end
end

local function IsEnemy(player)
    if not TEAM_CHECK then
        return true
    end
    return player.Team ~= LocalPlayer.Team
end

RunService.RenderStepped:Connect(function()
    if not ESP_ENABLED then
        for _, v in pairs(Drawings) do
            for _, obj in pairs(v) do
                obj.Visible = false
            end
        end
        return
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local hrp = player.Character.HumanoidRootPart
            local humanoid = player.Character.Humanoid

            if humanoid.Health > 0 and IsEnemy(player) then
                local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

                if onScreen then
                    local distance = (LocalPlayer.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                    local scale = math.clamp(1 / (distance / 50), 0.5, 2)
                    local boxSize = Vector2.new(40 * scale, 80 * scale)

                    local d = Drawings[player]

                    if not d then
                        CreateESP(player)
                        d = Drawings[player]
                    end

                    d.Box.Size = boxSize
                    d.Box.Position = Vector2.new(pos.X - boxSize.X/2, pos.Y - boxSize.Y/2)
                    d.Box.Visible = true

                    d.Name.Text = player.Name
                    d.Name.Position = Vector2.new(pos.X, pos.Y - boxSize.Y/2 - 16)
                    d.Name.Visible = true

                    local healthPercent = humanoid.Health / humanoid.MaxHealth
                    local barHeight = boxSize.Y * healthPercent

                    d.HealthBar.From = Vector2.new(d.Box.Position.X - 6, d.Box.Position.Y + boxSize.Y)
                    d.HealthBar.To = Vector2.new(d.Box.Position.X - 6, d.Box.Position.Y + boxSize.Y - barHeight)
                    d.HealthBar.Color = Color3.fromRGB(
                        255 * (1 - healthPercent),
                        255 * healthPercent,
                        0
                    )
                    d.HealthBar.Visible = true
                else
                    for _, obj in pairs(Drawings[player]) do
                        obj.Visible = false
                    end
                end
            else
                for _, obj in pairs(Drawings[player]) do
                    obj.Visible = false
                end
            end
        end
    end
end)

Players.PlayerRemoving:Connect(RemoveESP)
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        CreateESP(player)
    end)
end)

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LocalPlayer then
        CreateESP(player)
    end
end

-- Funções Aimbot
local function GetClosestEnemy()
    local closest = nil
    local shortestDistance = math.huge

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            if player.Character.Humanoid.Health > 0 and IsEnemy(player) then
                local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)

                if onScreen then
                    local mousePos = UIS:GetMouseLocation()
                    local distance = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude

                    if distance < shortestDistance then
                        shortestDistance = distance
                        closest = player
                    end
                end
            end
        end
    end

    return closest
end

local AimbotActive = false

RunService.RenderStepped:Connect(function()
    if AIMBOT_ENABLED and AimbotActive then
        local target = GetClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = target.Character.HumanoidRootPart.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
        end
    end
end)

-- Interface - Botão AHG e Menu
local screenSize = Camera.ViewportSize

local AHGButton = Drawing.new("Square")
AHGButton.Size = Vector2.new(50, 30)
AHGButton.Position = Vector2.new(screenSize.X - 60, 10)
AHGButton.Color = Color3.fromRGB(30, 30, 30)
AHGButton.Filled = true
AHGButton.Transparency = 0.8

local AHGText = Drawing.new("Text")
AHGText.Text = "AHG"
AHGText.Size = 18
AHGText.Position = Vector2.new(screenSize.X - 35, 15)
AHGText.Center = true
AHGText.Color = Color3.fromRGB(255, 255, 255)
AHGText.Outline = true

local MenuButtons = {}

local function CreateMenu()
    local options = {"Toggle ESP", "Toggle Aimbot", "Press Aim"}
    for i, option in ipairs(options) do
        local btn = Drawing.new("Square")
        btn.Size = Vector2.new(200, 40)
        btn.Position = Vector2.new(screenSize.X/2 - 100, 100 + (i-1)*50)
        btn.Color = Color3.fromRGB(50, 50, 50)
        btn.Filled = true
        btn.Transparency = 0.8
        btn.Visible = false

        local txt = Drawing.new("Text")
        txt.Text = option
        txt.Size = 16
        txt.Position = Vector2.new(screenSize.X/2, 110 + (i-1)*50)
        txt.Center = true
        txt.Color = Color3.fromRGB(255, 255, 255)
        txt.Outline = true
        txt.Visible = false

        MenuButtons[i] = {Button = btn, Text = txt}
    end
end

CreateMenu()

-- Função de Clique (Mobile)
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end

    if input.UserInputType == Enum.UserInputType.Touch then
        local touchPos = input.Position

        if touchPos.X >= screenSize.X - 60 and touchPos.X <= screenSize.X - 10 and touchPos.Y >= 10 and touchPos.Y <= 40 then
            MENU_OPEN = not MENU_OPEN
            for _, elements in ipairs(MenuButtons) do
                elements.Button.Visible = MENU_OPEN
                elements.Text.Visible = MENU_OPEN
            end
        end

        if MENU_OPEN then
            for idx, elements in ipairs(MenuButtons) do
                local btn = elements.Button
                if touchPos.X >= btn.Position.X and touchPos.X <= btn.Position.X + btn.Size.X and touchPos.Y >= btn.Position.Y and touchPos.Y <= btn.Position.Y + btn.Size.Y then
                    if idx == 1 then
                        ESP_ENABLED = not ESP_ENABLED
                    elseif idx == 2 then
                        AIMBOT_ENABLED = not AIMBOT_ENABLED
                    elseif idx == 3 then
                        AimbotActive = not AimbotActive
                    end
                end
            end
        end
    end
end)

print("[Sistema Mobile] Carregado! Toque em 'AHG' no canto superior direito para abrir o menu.")
