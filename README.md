-- Carrega a UI Venyx
local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/zxciaz/VenyxUI/main/Reuploaded"))()
local venyx = library.new("Xenz Hub", 5013109572)

-- Abre com a tecla P
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.P then
        venyx:toggle()
    end
end)

-- Variáveis
local ESPEnabled = false
local ShowHealth = false
local ShowName = false
local ShowDistance = false

local AimbotEnabled = false
local ShowFOV = false
local AimFOV = 100
local Smoothness = 0.05
local AimPart = "Head"
local WallCheck = false
local TeamCheck = false

-- FOV Circle
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.NumSides = 100
fovCircle.Radius = AimFOV
fovCircle.Color = Color3.fromRGB(255,255,255)
fovCircle.Filled = false
fovCircle.Visible = false

-- Atualiza FOV
game:GetService("RunService").RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X/2, workspace.CurrentCamera.ViewportSize.Y/2)
    fovCircle.Radius = AimFOV
    fovCircle.Visible = ShowFOV
end)

-- Função para ESP
local drawings = {}

local function createESP(player)
    if player == game.Players.LocalPlayer then return end
    if drawings[player] then return end

    local text = Drawing.new("Text")
    text.Center = true
    text.Outline = true
    text.Size = 14
    text.Color = Color3.fromRGB(255, 255, 255)
    text.Visible = false

    drawings[player] = text
end

-- Atualiza ESP
game:GetService("RunService").RenderStepped:Connect(function()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            createESP(player)
            local char = player.Character
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChild("Humanoid")
            local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(hrp.Position)
            local text = ""

            if onScreen and ESPEnabled then
                if ShowName then text = text .. player.Name .. " " end
                if ShowHealth and hum then text = text .. "[" .. math.floor(hum.Health) .. "] " end
                if ShowDistance then
                    local dist = (hrp.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                    text = text .. math.floor(dist) .. "m"
                end

                drawings[player].Position = Vector2.new(screenPos.X, screenPos.Y - 30)
                drawings[player].Text = text
                drawings[player].Visible = true
            else
                drawings[player].Visible = false
            end
        end
    end
end)

game.Players.PlayerAdded:Connect(function(p) createESP(p) end)
for _, p in pairs(game.Players:GetPlayers()) do createESP(p) end

-- Função para Aimbot
function getClosestPlayer()
    local closest, dist = nil, AimFOV
    local localPlayer = game.Players.LocalPlayer
    local cam = workspace.CurrentCamera

    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild(AimPart) then
            if TeamCheck and player.Team == localPlayer.Team then continue end
            local part = player.Character[AimPart]
            local pos, onScreen = cam:WorldToViewportPoint(part.Position)
            if not onScreen then continue end
            if WallCheck then
                local ray = workspace:Raycast(cam.CFrame.Position, (part.Position - cam.CFrame.Position).Unit * 1000, RaycastParams.new())
                if ray and ray.Instance and not ray.Instance:IsDescendantOf(player.Character) then continue end
            end
            local magnitude = (Vector2.new(pos.X, pos.Y) - Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)).Magnitude
            if magnitude < dist then
                dist = magnitude
                closest = part
            end
        end
    end

    return closest
end

-- Função para mover mira
game:GetService("RunService").RenderStepped:Connect(function()
    if AimbotEnabled then
        local target = getClosestPlayer()
        if target then
            local cam = workspace.CurrentCamera
            local pos = cam:WorldToViewportPoint(target.Position)
            local mouse = game.Players.LocalPlayer:GetMouse()
            mousemoverel((pos.X - cam.ViewportSize.X / 2) * Smoothness, (pos.Y - cam.ViewportSize.Y / 2) * Smoothness)
        end
    end
end)

-- UI: ESP
local espPage = venyx:addPage("ESP", 5012544693)
local espSec = espPage:addSection("Opções de ESP")
espSec:addToggle("Ativar ESP", nil, function(v) ESPEnabled = v end)
espSec:addToggle("Mostrar Vida", nil, function(v) ShowHealth = v end)
espSec:addToggle("Mostrar Nome", nil, function(v) ShowName = v end)
espSec:addToggle("Mostrar Distância", nil, function(v) ShowDistance = v end)

-- UI: Aimbot
local aimPage = venyx:addPage("Aimbot", 5012544693)
local aimSec = aimPage:addSection("Opções de Aimbot")
aimSec:addToggle("Ativar Aimbot", nil, function(v) AimbotEnabled = v end)
aimSec:addToggle("Mostrar FOV", nil, function(v) ShowFOV = v end)
aimSec:addSlider("Tamanho FOV", 10, 500, AimFOV, function(v) AimFOV = v end)
aimSec:addSlider("Suavidade", 0.01, 1, Smoothness, function(v) Smoothness = v end)
aimSec:addDropdown("Parte Alvo", {"Head", "HumanoidRootPart"}, function(v) AimPart = v end)
aimSec:addToggle("Wall Check", nil, function(v) WallCheck = v end)
aimSec:addToggle("Team Check", nil, function(v) TeamCheck = v end)

venyx:SelectPage(venyx.pages[1], true)
