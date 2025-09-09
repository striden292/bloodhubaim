-- Green Aim 1.4 - Versão Simplificada e Funcional
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local lp = Players.LocalPlayer
local mouse = lp:GetMouse()
local cam = workspace.CurrentCamera

-- CONFIG
local Config = {
    MenuKey = Enum.KeyCode.M,
    DefaultLockKey = Enum.KeyCode.Q,
    AimRadius = 120,
    Smooth = 0.4,
    LockPart = "HumanoidRootPart"
}

-- VARS GLOBAIS
local ScreenGui, MainFrame, statusLabel, keyLabel, changeBtn
local fovCircle
local lockedTarget = nil
local lockKey = Config.DefaultLockKey
local menuOpen = true
local locked = false
local waitingForKey = false

-- CRIAR GUI
local function CreateGUI()
    -- Remove GUI anterior
    if ScreenGui then ScreenGui:Destroy() end
    
    ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "GreenAimGUI"
    ScreenGui.ResetOnSpawn = false
    ScreenGui.Parent = game.CoreGui

    -- Frame principal
    MainFrame = Instance.new("Frame")
    MainFrame.Size = UDim2.new(0, 300, 0, 180)
    MainFrame.Position = UDim2.new(0.5, -150, 0.5, -90)
    MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    MainFrame.BorderSizePixel = 0
    MainFrame.Active = true
    MainFrame.Draggable = true
    MainFrame.Visible = menuOpen
    MainFrame.Parent = ScreenGui
    
    -- Bordas redondas
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = MainFrame
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = Color3.fromRGB(0, 255, 0)
    stroke.Thickness = 2
    stroke.Parent = MainFrame

    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 40)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.fromRGB(0, 120, 0)
    title.Text = "🎯 Green Aim 1.4"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.Font = Enum.Font.GothamBold
    title.TextSize = 18
    title.Parent = MainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = title

    -- Status
    statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, -20, 0, 30)
    statusLabel.Position = UDim2.new(0, 10, 0, 50)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Status: 🔓 DESBLOQUEADO"
    statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.TextSize = 16
    statusLabel.TextXAlignment = Enum.TextXAlignment.Left
    statusLabel.Parent = MainFrame

    -- Tecla atual
    keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -20, 0, 25)
    keyLabel.Position = UDim2.new(0, 10, 0, 90)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "Tecla atual: " .. lockKey.Name
    keyLabel.TextColor3 = Color3.new(1, 1, 1)
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.TextSize = 14
    keyLabel.TextXAlignment = Enum.TextXAlignment.Left
    keyLabel.Parent = MainFrame

    -- Botão mudar tecla
    changeBtn = Instance.new("TextButton")
    changeBtn.Size = UDim2.new(1, -20, 0, 35)
    changeBtn.Position = UDim2.new(0, 10, 0, 125)
    changeBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
    changeBtn.Text = "MUDAR TECLA"
    changeBtn.TextColor3 = Color3.new(1, 1, 1)
    changeBtn.Font = Enum.Font.GothamBold
    changeBtn.TextSize = 14
    changeBtn.Parent = MainFrame
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 8)
    btnCorner.Parent = changeBtn
end

-- CRIAR CÍRCULO FOV
local function CreateFOV()
    if fovCircle then fovCircle:Remove() end
    
    fovCircle = Drawing.new("Circle")
    fovCircle.Visible = true
    fovCircle.Radius = Config.AimRadius
    fovCircle.Position = Vector2.new(mouse.X, mouse.Y + 36)
    fovCircle.Color = Color3.fromRGB(0, 255, 0)
    fovCircle.Thickness = 2
    fovCircle.NumSides = 50
    fovCircle.Transparency = 0.7
end

-- BUSCAR ALVO
local function GetTarget()
    local closest = nil
    local shortestDistance = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= lp and player.Character then
            local hrp = player.Character:FindFirstChild(Config.LockPart)
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            
            if hrp and humanoid and humanoid.Health > 0 then
                local vector, onScreen = cam:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local distance = (Vector2.new(vector.X, vector.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
                    if distance < Config.AimRadius and distance < shortestDistance then
                        closest = hrp
                        shortestDistance = distance
                    end
                end
            end
        end
    end
    
    return closest
end

-- ATUALIZAR STATUS VISUAL
local function UpdateStatus()
    if statusLabel then
        if locked and lockedTarget then
            statusLabel.Text = "Status: 🎯 BLOQUEADO"
            statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        else
            statusLabel.Text = "Status: 🔓 DESBLOQUEADO"
            statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        end
    end
end

-- EVENTO DO BOTÃO
local function OnButtonClick()
    if not waitingForKey then
        waitingForKey = true
        changeBtn.Text = "PRESSIONE UMA TECLA..."
        changeBtn.BackgroundColor3 = Color3.fromRGB(200, 200, 0)
        print("🔧 Clique em qualquer tecla para alterar...")
    end
end

-- SISTEMA DE INPUT
local function OnInputBegan(input, gameProcessed)
    if gameProcessed then return end
    
    -- MUDANÇA DE TECLA
    if waitingForKey and input.UserInputType == Enum.UserInputType.Keyboard then
        lockKey = input.KeyCode
        waitingForKey = false
        
        -- Atualiza interface
        keyLabel.Text = "Tecla atual: " .. lockKey.Name
        changeBtn.Text = "MUDAR TECLA"
        changeBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        
        print("✅ Nova tecla: " .. lockKey.Name)
        return
    end
    
    -- LOCK/UNLOCK
    if input.KeyCode == lockKey then
        if locked then
            -- Desbloquear
            lockedTarget = nil
            locked = false
            print("🔓 Desbloqueado!")
        else
            -- Tentar bloquear
            local target = GetTarget()
            if target then
                lockedTarget = target
                locked = true
                print("🎯 Bloqueado em alvo!")
            else
                print("❌ Nenhum alvo no alcance!")
            end
        end
        UpdateStatus()
        return
    end
    
    -- MENU
    if input.KeyCode == Config.MenuKey then
        menuOpen = not menuOpen
        MainFrame.Visible = menuOpen
        print("📱 Menu: " .. (menuOpen and "Aberto" or "Fechado"))
        return
    end
end

-- LOOP PRINCIPAL
local function MainLoop()
    -- Atualizar FOV
    if fovCircle then
        fovCircle.Position = Vector2.new(mouse.X, mouse.Y + 36)
        fovCircle.Color = locked and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(0, 255, 0)
    end
    
    -- Sistema de mira
    if locked and lockedTarget then
        if lockedTarget.Parent then
            local character = lockedTarget.Parent
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            
            if humanoid and humanoid.Health > 0 then
                -- Mirar no alvo
                local targetPosition = lockedTarget.Position
                local newCFrame = CFrame.lookAt(cam.CFrame.Position, targetPosition)
                cam.CFrame = cam.CFrame:Lerp(newCFrame, Config.Smooth)
            else
                -- Alvo morreu
                lockedTarget = nil
                locked = false
                UpdateStatus()
                print("💀 Alvo eliminado!")
            end
        else
            -- Alvo sumiu
            lockedTarget = nil
            locked = false
            UpdateStatus()
            print("👻 Alvo perdido!")
        end
    end
end

-- CONECTAR EVENTOS
UIS.InputBegan:Connect(OnInputBegan)
RS.RenderStepped:Connect(MainLoop)

-- INICIALIZAR
CreateGUI()
CreateFOV()

-- Conectar botão DEPOIS da GUI estar criada
changeBtn.MouseButton1Click:Connect(OnButtonClick)

print("✅ Green Aim 1.4 carregado!")
print("🎯 Tecla de mira: " .. lockKey.Name)
print("📱 Tecla do menu: M")
print("🔧 Clique no botão para mudar a tecla")
