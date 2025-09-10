local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Configurações iniciais
local aimlockEnabled = false
local aimlockKey = Enum.KeyCode.E
local fovRadius = 100
local fovColor = Color3.fromRGB(255, 0, 0)
local lockedTargetHumanoid = nil

-- Configuração Speed Hack
local speedEnabled = false
local speedValue = 16 -- Valor padrão Roblox
local speedMax = 100
local speedMin = 16

-- Criar GUI do menu
local bloodHubGui = Instance.new("ScreenGui")
bloodHubGui.Name = "BloodHub"
bloodHubGui.ResetOnSpawn = false
bloodHubGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 230)
frame.Position = UDim2.new(0.5, -125, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(120, 0, 0) -- vermelho escuro
frame.BorderSizePixel = 0
frame.Active = true -- Necessário para drag
frame.Selectable = true
frame.Parent = bloodHubGui

-- Bordas arredondadas e borda preta
local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 18)
frameCorner.Parent = frame

local frameStroke = Instance.new("UIStroke")
frameStroke.Color = Color3.fromRGB(0, 0, 0)
frameStroke.Thickness = 3
frameStroke.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Blood Hub"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBlack
title.TextSize = 32
title.Parent = frame

local aimlockBtn = Instance.new("TextButton")
aimlockBtn.Size = UDim2.new(0.8, 0, 0, 40)
aimlockBtn.Position = UDim2.new(0.1, 0, 0, 50)
aimlockBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
aimlockBtn.Text = "Aimlock: OFF"
aimlockBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
aimlockBtn.Font = Enum.Font.GothamBold
aimlockBtn.TextSize = 22
aimlockBtn.Parent = frame

local aimlockBtnCorner = Instance.new("UICorner")
aimlockBtnCorner.CornerRadius = UDim.new(0, 10)
aimlockBtnCorner.Parent = aimlockBtn

local keyLabel = Instance.new("TextLabel")
keyLabel.Size = UDim2.new(0.8, 0, 0, 30)
keyLabel.Position = UDim2.new(0.1, 0, 0, 100)
keyLabel.BackgroundTransparency = 1
keyLabel.Text = "Tecla Aimlock: E"
keyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
keyLabel.Font = Enum.Font.Gotham
keyLabel.TextSize = 18
keyLabel.Parent = frame

local changeKeyBtn = Instance.new("TextButton")
changeKeyBtn.Size = UDim2.new(0.8, 0, 0, 30)
changeKeyBtn.Position = UDim2.new(0.1, 0, 0, 140)
changeKeyBtn.BackgroundColor3 = Color3.fromRGB(180, 0, 0)
changeKeyBtn.Text = "Mudar Tecla"
changeKeyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
changeKeyBtn.Font = Enum.Font.GothamBold
changeKeyBtn.TextSize = 18
changeKeyBtn.Parent = frame

local changeKeyBtnCorner = Instance.new("UICorner")
changeKeyBtnCorner.CornerRadius = UDim.new(0, 10)
changeKeyBtnCorner.Parent = changeKeyBtn

-- Speed Hack GUI
local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0.8, 0, 0, 24)
speedLabel.Position = UDim2.new(0.1, 0, 0, 180)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Speed: " .. speedValue
speedLabel.TextColor3 = Color3.fromRGB(255,255,255)
speedLabel.Font = Enum.Font.Gotham
speedLabel.TextSize = 18
speedLabel.Parent = frame

local speedMinusBtn = Instance.new("TextButton")
speedMinusBtn.Size = UDim2.new(0.2, 0, 0, 24)
speedMinusBtn.Position = UDim2.new(0.1, 0, 0, 205)
speedMinusBtn.BackgroundColor3 = Color3.fromRGB(80,0,0)
speedMinusBtn.Text = "-"
speedMinusBtn.TextColor3 = Color3.fromRGB(255,255,255)
speedMinusBtn.Font = Enum.Font.GothamBold
speedMinusBtn.TextSize = 20
speedMinusBtn.Parent = frame

local speedMinusBtnCorner = Instance.new("UICorner")
speedMinusBtnCorner.CornerRadius = UDim.new(0, 8)
speedMinusBtnCorner.Parent = speedMinusBtn

local speedPlusBtn = Instance.new("TextButton")
speedPlusBtn.Size = UDim2.new(0.2, 0, 0, 24)
speedPlusBtn.Position = UDim2.new(0.7, 0, 0, 205)
speedPlusBtn.BackgroundColor3 = Color3.fromRGB(80,0,0)
speedPlusBtn.Text = "+"
speedPlusBtn.TextColor3 = Color3.fromRGB(255,255,255)
speedPlusBtn.Font = Enum.Font.GothamBold
speedPlusBtn.TextSize = 20
speedPlusBtn.Parent = frame

local speedPlusBtnCorner = Instance.new("UICorner")
speedPlusBtnCorner.CornerRadius = UDim.new(0, 8)
speedPlusBtnCorner.Parent = speedPlusBtn

local speedOnOffBtn = Instance.new("TextButton")
speedOnOffBtn.Size = UDim2.new(0.4, 0, 0, 24)
speedOnOffBtn.Position = UDim2.new(0.3, 0, 0, 205)
speedOnOffBtn.BackgroundColor3 = Color3.fromRGB(180,0,0)
speedOnOffBtn.Text = "Speed: OFF"
speedOnOffBtn.TextColor3 = Color3.fromRGB(255,255,255)
speedOnOffBtn.Font = Enum.Font.GothamBold
speedOnOffBtn.TextSize = 16
speedOnOffBtn.Parent = frame

local speedOnOffBtnCorner = Instance.new("UICorner")
speedOnOffBtnCorner.CornerRadius = UDim.new(0, 8)
speedOnOffBtnCorner.Parent = speedOnOffBtn

-- Drag menu logic
local dragging = false
local dragInput, dragStart, startPos

local function updateDrag(input)
    local delta = input.Position - dragStart
    frame.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        -- Só inicia drag se não clicar em botões internos
        local mouse = LocalPlayer:GetMouse()
        local guiObjects = frame:GetChildren()
        local mouseX, mouseY = mouse.X, mouse.Y
        local frameAbsPos = frame.AbsolutePosition
        local frameAbsSize = frame.AbsoluteSize
        local clickedOnButton = false
        for _, obj in guiObjects do
            if obj:IsA("TextButton") or obj:IsA("TextBox") then
                local objPos = obj.AbsolutePosition
                local objSize = obj.AbsoluteSize
                if mouseX >= objPos.X and mouseX <= objPos.X + objSize.X and mouseY >= objPos.Y and mouseY <= objPos.Y + objSize.Y then
                    clickedOnButton = true
                    break
                end
            end
        end
        if not clickedOnButton then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end
end)

frame.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        updateDrag(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Função para detectar Model com Humanoid sob o mouse e retornar o Humanoid
local function getHumanoidUnderMouse()
    local mouse = LocalPlayer:GetMouse()
    local camera = workspace.CurrentCamera
    local ray = camera:ScreenPointToRay(mouse.X, mouse.Y)
    local rayOrigin = ray.Origin
    local rayDirection = ray.Direction * 1000

    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = {}
    if LocalPlayer.Character then
        table.insert(params.FilterDescendantsInstances, LocalPlayer.Character)
    end

    local result = workspace:Raycast(rayOrigin, rayDirection, params)
    if result and result.Instance then
        local part = result.Instance
        -- Verifica se é parte de um Model com Humanoid (player, NPC ou dummy)
        if part.Parent and part.Parent:IsA("Model") then
            local humanoid = part.Parent:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                -- Nunca grudar em si mesmo
                if part.Parent ~= LocalPlayer.Character then
                    return humanoid
                end
            end
        end
    end
    return nil
end

-- Aimlock lógica
RunService.RenderStepped:Connect(function()
    if aimlockEnabled and lockedTargetHumanoid then
        local camera = workspace.CurrentCamera
        local rootPart = lockedTargetHumanoid.Parent:FindFirstChild("HumanoidRootPart")
        if rootPart then
            camera.CFrame = CFrame.new(camera.CFrame.Position, rootPart.Position)
        end
    end
end)

-- FOV vermelho em volta do mouse
local fovCircle = Instance.new("Frame")
fovCircle.Size = UDim2.new(0, fovRadius*2, 0, fovRadius*2)
fovCircle.BackgroundTransparency = 1
fovCircle.BorderSizePixel = 0
fovCircle.Parent = bloodHubGui

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(1,0)
uiCorner.Parent = fovCircle

local uiStroke = Instance.new("UIStroke")
uiStroke.Color = fovColor
uiStroke.Thickness = 3
uiStroke.Parent = fovCircle

local function updateFovCircle()
    local mouse = LocalPlayer:GetMouse()
    fovCircle.Position = UDim2.new(0, mouse.X - fovRadius, 0, mouse.Y - fovRadius)
end

RunService.RenderStepped:Connect(updateFovCircle)

-- Função para alternar Aimlock
local function toggleAimlock()
    if not aimlockEnabled then
        -- Ativar: buscar Model com Humanoid sob mouse (player, NPC ou dummy)
        local targetHumanoid = getHumanoidUnderMouse()
        if targetHumanoid then
            aimlockEnabled = true
            lockedTargetHumanoid = targetHumanoid
            aimlockBtn.Text = "Aimlock: ON"
        else
            aimlockBtn.Text = "Aimlock: Nenhum alvo"
        end
    else
        -- Desativar
        aimlockEnabled = false
        lockedTargetHumanoid = nil
        aimlockBtn.Text = "Aimlock: OFF"
    end
end

-- Botão Aimlock
aimlockBtn.MouseButton1Click:Connect(toggleAimlock)

-- Mudar tecla de ativação
local changingKey = false
changeKeyBtn.MouseButton1Click:Connect(function()
    changingKey = true
    changeKeyBtn.Text = "Pressione nova tecla..."
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if changingKey and input.UserInputType == Enum.UserInputType.Keyboard then
        aimlockKey = input.KeyCode
        keyLabel.Text = "Tecla Aimlock: " .. input.KeyCode.Name
        changeKeyBtn.Text = "Mudar Tecla"
        changingKey = false
    elseif input.KeyCode == aimlockKey and not processed then
        toggleAimlock()
    elseif input.KeyCode == Enum.KeyCode.LeftControl and not processed then
        -- Ativa/desativa speed hack ao pressionar LeftControl
        speedEnabled = not speedEnabled
        if speedEnabled then
            speedOnOffBtn.Text = "Speed: ON"
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = speedValue
                end
            end
        else
            speedOnOffBtn.Text = "Speed: OFF"
            local character = LocalPlayer.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = 16 -- Valor padrão
                end
            end
        end
    end
end)

-- Speed Hack lógica
local function setSpeed(val)
    speedValue = val
    speedLabel.Text = "Speed: " .. speedValue
    if speedEnabled then
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = speedValue
            end
        end
    end
end

speedMinusBtn.MouseButton1Click:Connect(function()
    local newVal = speedValue - 4
    if newVal < speedMin then newVal = speedMin end
    setSpeed(newVal)
end)
speedPlusBtn.MouseButton1Click:Connect(function()
    local newVal = speedValue + 4
    if newVal > speedMax then newVal = speedMax end
    setSpeed(newVal)
end)

speedOnOffBtn.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    if speedEnabled then
        speedOnOffBtn.Text = "Speed: ON"
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = speedValue
            end
        end
    else
        speedOnOffBtn.Text = "Speed: OFF"
        local character = LocalPlayer.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = 16 -- Valor padrão
            end
        end
    end
end)

-- Sempre que o personagem do jogador mudar, re-aplica speed
LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.2)
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then
        if speedEnabled then
            humanoid.WalkSpeed = speedValue
        else
            humanoid.WalkSpeed = 16
        end
    end
end)

-- Se já estiver spawnado ao iniciar
if LocalPlayer.Character then
    local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speedEnabled and speedValue or 16
    end
end

