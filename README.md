-- Green Aim 1.4 - Ignora o próprio jogador + Grudado nos outros
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local lp = Players.LocalPlayer
local mouse = lp:GetMouse()
local cam = workspace.CurrentCamera

-- CONFIG
local Config = {
    MenuKey = Enum.KeyCode.M,        -- M fecha/abre menu
    DefaultLockKey = Enum.KeyCode.Q, -- tecla de lock (mudável)
    AimRadius = 120,
    Smooth = 0.4,
    LockPart = "HumanoidRootPart",
    WallCheck = false
}

-- VARS
local ScreenGui, MainFrame
local fovCircle
local lockedTarget = nil
local lockKey = Config.DefaultLockKey
local menuOpen = true
local locked = false

-- GUI
local function CreateGUI()
    ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "GreenAim"
    ScreenGui.Parent = game:GetService("CoreGui")

    MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 240, 0, 120)
    MainFrame.Position = UDim2.new(0.5, -120, 0.5, -60)
    MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    MainFrame.BorderColor3 = Color3.fromRGB(0, 255, 0)
    MainFrame.BorderSizePixel = 2
    MainFrame.Visible = menuOpen
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Parent = ScreenGui

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 30)
    title.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
    title.Text = "Green Aim 1.4"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.Font = Enum.Font.SourceSansBold
    title.TextSize = 16
    title.Parent = MainFrame

    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -20, 0, 20)
    keyLabel.Position = UDim2.new(0, 10, 0, 40)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "Lock-Key: " .. lockKey.Name
    keyLabel.TextColor3 = Color3.new(1, 1, 1)
    keyLabel.Font = Enum.Font.SourceSans
    keyLabel.TextSize = 14
    keyLabel.Parent = MainFrame

    local changeBtn = Instance.new("TextButton")
    changeBtn.Size = UDim2.new(1, -20, 0, 25)
    changeBtn.Position = UDim2.new(0, 10, 0, 70)
    changeBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 0)
    changeBtn.Text = "Mudar tecla"
    changeBtn.TextColor3 = Color3.new(1, 1, 1)
    changeBtn.Font = Enum.Font.SourceSans
    changeBtn.TextSize = 14
    changeBtn.Parent = MainFrame

    local listening = false
    changeBtn.MouseButton1Click:Connect(function()
        listening = true
        changeBtn.Text = "Aperte uma tecla..."
    end)

    UIS.InputBegan:Connect(function(input, gp)
        if listening and input.UserInputType == Enum.UserInputType.Keyboard then
            lockKey = input.KeyCode
            keyLabel.Text = "Lock-Key: " .. lockKey.Name
            changeBtn.Text = "Mudar tecla"
            listening = false
        end
    end)
end

-- FOV
local function CreateFOV()
    fovCircle = Drawing.new("Circle")
    fovCircle.Visible = true
    fovCircle.Radius = Config.AimRadius
    fovCircle.Position = Vector2.new(mouse.X, mouse.Y + 36)
    fovCircle.Color = Color3.fromRGB(0, 255, 0)
    fovCircle.Thickness = 2
    fovCircle.NumSides = 30
end

-- Alvo sob o cursor (IGNORA O PRÓPRIO JOGADOR)
local function GetTarget()
    local closest, minDist = nil, math.huge
    for _, obj in pairs(workspace:GetDescendants()) do
        local hrp = obj:FindFirstChild(Config.LockPart)
        local hum = obj:FindFirstChildOfClass("Humanoid")
        if hrp and hum and hum.Health > 0 then
            -- IGNORA SE FOR O PRÓPRIO CHARACTER
            if obj ~= lp.Character then
                local screen, onScreen = cam:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local dist = (Vector2.new(screen.X, screen.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                    if dist < Config.AimRadius and dist < minDist then
                        closest = hrp
                        minDist = dist
                    end
                end
            end
        end
    end
    return closest
end

-- Lock com a tecla escolhida
local locked = false
UIS.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == lockKey then
        if locked then -- já travado? desbloqueia
            lockedTarget = nil
            locked = false
        else -- livre? tenta travar
            local tgt = GetTarget()
            if tgt then
                lockedTarget = tgt
                locked = true
            end
        end
    elseif input.KeyCode == Config.MenuKey then -- M minimiza/restaura
        menuOpen = not menuOpen
        MainFrame.Visible = menuOpen
    end
end)

-- Loop GRUDADO
RS.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(mouse.X, mouse.Y + 36)
    fovCircle.Radius = Config.AimRadius

    if lockedTarget and lockedTarget.Parent then
        -- SEMPRE olha para o alvo (grudado)
        cam.CFrame = cam.CFrame:Lerp(CFrame.lookAt(cam.CFrame.Position, lockedTarget.Position), Config.Smooth)
    else
        -- Alvo sumiu / morreu
        lockedTarget = nil
        locked = false
    end
end)

-- Init
CreateGUI()
CreateFOV()
print("✅ Green Aim 1.4 - Tecla: " .. lockKey.Name .. " | M menu | IGNORA VOCÊ | Travamento GRUDADO")
