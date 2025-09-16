--// Sistema de Aimlock - by vamp

----------------------------------------------------------------------
-- SERVIÇOS
----------------------------------------------------------------------
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunS = game:GetService("RunService")
local TweenS = game:GetService("TweenService")
local Camera = workspace.CurrentCamera

local player = Players.LocalPlayer

----------------------------------------------------------------------
-- SISTEMA DE LOGIN
----------------------------------------------------------------------
local CORRECT_KEY = "bloodhub"
local isAuthenticated = false

-- Função para criar GUI segura
local function createGui()
    local success, gui = pcall(function()
        return player:WaitForChild("PlayerGui")
    end)
    
    if not success then
        -- Fallback para CoreGui
        return game:GetService("CoreGui")
    end
    
    return gui
end

-- Cria GUI de Login
local loginGui = Instance.new("ScreenGui")
loginGui.Name = "BloodLogin"
loginGui.ResetOnSpawn = false
loginGui.Parent = createGui()

-- Fundo escuro
local backdrop = Instance.new("Frame")
backdrop.Size = UDim2.new(1, 0, 1, 0)
backdrop.Position = UDim2.new(0, 0, 0, 0)
backdrop.BackgroundColor3 = Color3.new(0, 0, 0)
backdrop.BackgroundTransparency = 0.3
backdrop.BorderSizePixel = 0
backdrop.Parent = loginGui

-- Frame principal do login
local loginFrame = Instance.new("Frame")
loginFrame.Size = UDim2.new(0, 400, 0, 300)
loginFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
loginFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
loginFrame.BorderSizePixel = 0
loginFrame.Parent = loginGui

local loginCorner = Instance.new("UICorner", loginFrame)
loginCorner.CornerRadius = UDim.new(0, 12)

-- Título
local titleLabel = Instance.new("TextLabel")
titleLabel.Text = "BLOOD AIMLOCK"
titleLabel.Size = UDim2.new(1, 0, 0, 60)
titleLabel.Position = UDim2.new(0, 0, 0, 20)
titleLabel.BackgroundTransparency = 1
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 24
titleLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
titleLabel.Parent = loginFrame

-- Subtitle
local subtitleLabel = Instance.new("TextLabel")
subtitleLabel.Text = "Enter access key to continue"
subtitleLabel.Size = UDim2.new(1, 0, 0, 30)
subtitleLabel.Position = UDim2.new(0, 0, 0, 80)
subtitleLabel.BackgroundTransparency = 1
subtitleLabel.Font = Enum.Font.Gotham
subtitleLabel.TextSize = 14
subtitleLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
subtitleLabel.Parent = loginFrame

-- Input Box
local keyInput = Instance.new("TextBox")
keyInput.Size = UDim2.new(0, 300, 0, 40)
keyInput.Position = UDim2.new(0.5, -150, 0, 130)
keyInput.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
keyInput.BorderSizePixel = 0
keyInput.Font = Enum.Font.Gotham
keyInput.TextSize = 16
keyInput.TextColor3 = Color3.new(1, 1, 1)
keyInput.PlaceholderText = "Enter key here..."
keyInput.PlaceholderColor3 = Color3.fromRGB(100, 100, 100)
keyInput.Text = ""
keyInput.Parent = loginFrame

local inputCorner = Instance.new("UICorner", keyInput)
inputCorner.CornerRadius = UDim.new(0, 8)

-- Login Button
local loginButton = Instance.new("TextButton")
loginButton.Size = UDim2.new(0, 300, 0, 40)
loginButton.Position = UDim2.new(0.5, -150, 0, 190)
loginButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
loginButton.BorderSizePixel = 0
loginButton.Font = Enum.Font.GothamBold
loginButton.TextSize = 16
loginButton.TextColor3 = Color3.new(1, 1, 1)
loginButton.Text = "LOGIN"
loginButton.Parent = loginFrame

local buttonCorner = Instance.new("UICorner", loginButton)
buttonCorner.CornerRadius = UDim.new(0, 8)

-- Status Label
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 0, 30)
statusLabel.Position = UDim2.new(0, 0, 0, 250)
statusLabel.BackgroundTransparency = 1
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 12
statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
statusLabel.Text = ""
statusLabel.Parent = loginFrame

----------------------------------------------------------------------
-- SISTEMA PRINCIPAL DE AIMLOCK
----------------------------------------------------------------------
local function initializeAimlock()
    
    print("🎯 Inicializando Blood Aimlock...")
    
    ----------------------------------------------------------------------
    -- CONFIGURAÇÕES
    ----------------------------------------------------------------------
    local Config = {
        FOV_RADIUS = 80,
        TARGET_BONE = "Head"
    }

    ----------------------------------------------------------------------
    -- VARIÁVEIS GLOBAIS
    ----------------------------------------------------------------------
    local aimlockActive = false
    local currentTarget = nil
    local connections = {}

    ----------------------------------------------------------------------
    -- INTRO ANIMADA
    ----------------------------------------------------------------------
    local function showIntro()
        local success, introGui = pcall(function()
            local gui = Instance.new("ScreenGui")
            gui.Name = "BloodAimIntro"
            gui.ResetOnSpawn = false
            gui.Parent = createGui()
            return gui
        end)
        
        if not success then return end
        
        -- Texto principal
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Size = UDim2.new(1, 0, 1, 0)
        titleLabel.Position = UDim2.new(0, 0, 0, 0)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = "Blood Aim"
        titleLabel.TextColor3 = Color3.fromRGB(220, 20, 60)
        titleLabel.TextScaled = true
        titleLabel.TextTransparency = 1
        titleLabel.Font = Enum.Font.GothamBold
        titleLabel.Parent = introGui
        
        local titleStroke = Instance.new("UIStroke")
        titleStroke.Color = Color3.fromRGB(139, 0, 0)
        titleStroke.Thickness = 3
        titleStroke.Parent = titleLabel
        
        -- Texto "by vamp"
        local creditLabel = Instance.new("TextLabel")
        creditLabel.Size = UDim2.new(1, 0, 0.3, 0)
        creditLabel.Position = UDim2.new(0, 0, 0.6, 0)
        creditLabel.BackgroundTransparency = 1
        creditLabel.Text = "by vamp"
        creditLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        creditLabel.TextScaled = true
        creditLabel.TextTransparency = 1
        creditLabel.Font = Enum.Font.Gotham
        creditLabel.Parent = introGui
        
        local creditStroke = Instance.new("UIStroke")
        creditStroke.Color = Color3.fromRGB(0, 0, 0)
        creditStroke.Thickness = 2
        creditStroke.Parent = creditLabel
        
        -- Animações
        spawn(function()
            TweenS:Create(titleLabel, TweenInfo.new(1), {TextTransparency = 0}):Play()
            wait(0.3)
            TweenS:Create(creditLabel, TweenInfo.new(1), {TextTransparency = 0}):Play()
            wait(1.5)
            TweenS:Create(titleLabel, TweenInfo.new(0.8), {TextTransparency = 1}):Play()
            TweenS:Create(creditLabel, TweenInfo.new(0.8), {TextTransparency = 1}):Play()
            wait(0.8)
            if introGui and introGui.Parent then
                introGui:Destroy()
            end
        end)
    end

    ----------------------------------------------------------------------
    -- FUNÇÕES SEGURAS
    ----------------------------------------------------------------------
    local function safeGetCharacter(targetPlayer)
        if not targetPlayer then return nil end
        
        local success, character = pcall(function()
            return targetPlayer.Character
        end)
        
        if not success or not character then return nil end
        return character
    end

    local function safeGetHumanoid(character)
        if not character then return nil end
        
        local success, humanoid = pcall(function()
            return character:FindFirstChildOfClass("Humanoid")
        end)
        
        if not success or not humanoid then return nil end
        return humanoid
    end

    local function isAlive(character)
        if not character then return false end
        
        local humanoid = safeGetHumanoid(character)
        if not humanoid then return false end
        
        local success, health = pcall(function()
            return humanoid.Health
        end)
        
        if not success or health <= 0 then return false end
        
        return character:FindFirstChild("Head") ~= nil
    end

    local function getTargetBone(character)
        if not character then return nil end
        
        local bone = character:FindFirstChild(Config.TARGET_BONE)
        if bone then return bone end
        
        local fallbacks = {"Head", "Torso", "UpperTorso", "HumanoidRootPart"}
        for _, boneName in ipairs(fallbacks) do
            bone = character:FindFirstChild(boneName)
            if bone then return bone end
        end
        
        return nil
    end

    local function getClosestTarget()
        local success, mousePos = pcall(function()
            return UIS:GetMouseLocation()
        end)
        
        if not success then return nil end
        
        local closest = nil
        local closestDistance = math.huge
        
        for _, targetPlayer in ipairs(Players:GetPlayers()) do
            if targetPlayer == player then continue end
            
            local character = safeGetCharacter(targetPlayer)
            if not isAlive(character) then continue end
            
            local targetBone = getTargetBone(character)
            if not targetBone then continue end
            
            local success2, screenPos, onScreen = pcall(function()
                return Camera:WorldToViewportPoint(targetBone.Position)
            end)
            
            if not success2 or not onScreen then continue end
            
            local distance = (mousePos - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
            if distance > Config.FOV_RADIUS then continue end
            
            if distance < closestDistance then
                closestDistance = distance
                closest = character
            end
        end
        
        return closest
    end

    ----------------------------------------------------------------------
    -- SISTEMA PRINCIPAL DE AIMLOCK
    ----------------------------------------------------------------------
    local function updateAimlock()
        if not aimlockActive then return end
        
        -- Verifica se o alvo atual ainda está vivo
        if currentTarget and not isAlive(currentTarget) then
            currentTarget = nil
        end
        
        -- Se não tem alvo, procura um
        if not currentTarget then
            currentTarget = getClosestTarget()
        end
        
        -- Se ainda não tem alvo, sai
        if not currentTarget then return end
        
        local targetBone = getTargetBone(currentTarget)
        if not targetBone then
            currentTarget = nil
            return
        end
        
        -- AIMLOCK - MIRA 100% GRUDADA
        local success = pcall(function()
            local targetPosition = targetBone.Position
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
        end)
        
        if not success then
            currentTarget = nil
        end
    end

    ----------------------------------------------------------------------
    -- CONTROLES
    ----------------------------------------------------------------------
    connections.input = UIS.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        
        if input.KeyCode == Enum.KeyCode.X then
            aimlockActive = not aimlockActive
            
            print("🎯 Aimlock:", aimlockActive and "ATIVADO" or "DESATIVADO")
            
            if aimlockActive then
                currentTarget = getClosestTarget()
                if currentTarget then
                    local targetName = "Unknown"
                    pcall(function()
                        for _, p in ipairs(Players:GetPlayers()) do
                            if p.Character == currentTarget then
                                targetName = p.Name
                                break
                            end
                        end
                    end)
                    print("🎯 Alvo:", targetName)
                else
                    print("🎯 Nenhum alvo encontrado")
                end
            else
                currentTarget = nil
            end
        end
    end)

    ----------------------------------------------------------------------
    -- LOOP PRINCIPAL
    ----------------------------------------------------------------------
    connections.update = RunS.RenderStepped:Connect(function()
        updateAimlock()
    end)

    -- Mostrar intro
    showIntro()
    
    ----------------------------------------------------------------------
    -- NOTIFICAÇÃO DE SUCESSO
    ----------------------------------------------------------------------
    spawn(function()
        local successGui = Instance.new("ScreenGui")
        successGui.Name = "BloodSuccess"
        successGui.ResetOnSpawn = false
        successGui.Parent = createGui()

        local successLabel = Instance.new("TextLabel")
        successLabel.Text = "🎯 BLOOD AIMLOCK LOADED - Press X to Toggle"
        successLabel.Size = UDim2.new(0, 450, 0, 50)
        successLabel.Position = UDim2.new(0.5, -225, 0, 100)
        successLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
        successLabel.BackgroundTransparency = 0.2
        successLabel.BorderSizePixel = 0
        successLabel.Font = Enum.Font.GothamBold
        successLabel.TextSize = 16
        successLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        successLabel.Parent = successGui

        local successCorner = Instance.new("UICorner", successLabel)
        successCorner.CornerRadius = UDim.new(0, 8)

        -- Remove notificação após 4 segundos
        wait(4)
        TweenS:Create(successLabel, TweenInfo.new(0.5), {BackgroundTransparency = 1, TextTransparency = 1}):Play()
        wait(0.5)
        if successGui and successGui.Parent then
            successGui:Destroy()
        end
    end)
    
    print("✅ Blood Aimlock carregado com sucesso!")
end

----------------------------------------------------------------------
-- SISTEMA DE AUTENTICAÇÃO
----------------------------------------------------------------------
local function authenticateKey(inputKey)
    if inputKey == CORRECT_KEY then
        isAuthenticated = true
        
        statusLabel.Text = "✓ Access Granted!"
        statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
        
        -- Animação de fechamento
        spawn(function()
            TweenS:Create(loginFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.In), 
                {Size = UDim2.new(0, 0, 0, 0), Position = UDim2.new(0.5, 0, 0.5, 0)}):Play()
            
            TweenS:Create(backdrop, TweenInfo.new(0.5), {BackgroundTransparency = 1}):Play()
            
            wait(0.6)
            if loginGui and loginGui.Parent then
                loginGui:Destroy()
            end
            
            -- Inicia o sistema principal
            initializeAimlock()
        end)
        
    else
        statusLabel.Text = "✗ Invalid Key"
        statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
        
        -- Animação de erro
        spawn(function()
            TweenS:Create(loginFrame, TweenInfo.new(0.1), {Position = UDim2.new(0.5, -210, 0.5, -150)}):Play()
            wait(0.1)
            TweenS:Create(loginFrame, TweenInfo.new(0.1), {Position = UDim2.new(0.5, -190, 0.5, -150)}):Play()
            wait(0.1)
            TweenS:Create(loginFrame, TweenInfo.new(0.1), {Position = UDim2.new(0.5, -200, 0.5, -150)}):Play()
        end)
        
        keyInput.Text = ""
    end
end

-- Eventos de login
loginButton.MouseButton1Click:Connect(function()
    authenticateKey(keyInput.Text)
end)

keyInput.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        authenticateKey(keyInput.Text)
    end
end)
