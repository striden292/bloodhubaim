-- LOCAL SCRIPT (StarterPlayerScripts)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

local aiming = false
local target = nil
local FOV_RADIUS = 100
local aimbotLoaded = false
local characterRemovingConnection = nil

-- GUI Setup
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
screenGui.Name = "AimbotGUI"
screenGui.ResetOnSpawn = false

-- Tela de carregamento
local loadingFrame = Instance.new("Frame", screenGui)
loadingFrame.Size = UDim2.new(0, 350, 0, 100)
loadingFrame.Position = UDim2.new(0.5, -175, 0.5, -50)
loadingFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
loadingFrame.BorderSizePixel = 0
loadingFrame.BackgroundTransparency = 0

-- Barra de progresso
local progressBar = Instance.new("Frame", loadingFrame)
progressBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
progressBar.BorderSizePixel = 0
progressBar.Size = UDim2.new(0, 0, 1, 0)
progressBar.Position = UDim2.new(0, 0, 0, 0)

-- Texto
local loadingText = Instance.new("TextLabel", loadingFrame)
loadingText.Size = UDim2.new(1, 0, 1, 0)
loadingText.BackgroundTransparency = 1
loadingText.Text = "Carregando: 0%"
loadingText.Font = Enum.Font.Gotham
loadingText.TextColor3 = Color3.new(1, 1, 1)
loadingText.TextSize = 22

-- Som de carregamento
local loadSound = Instance.new("Sound", loadingFrame)
loadSound.SoundId = "rbxassetid://86343926092450" -- Você pode trocar por outro
loadSound.Volume = 1
loadSound.Looped = true
loadSound:Play()

-- FOV Circle (Drawing API)
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = FOV_RADIUS
fovCircle.Thickness = 1
fovCircle.Color = Color3.fromRGB(0, 255, 0)
fovCircle.Filled = false
fovCircle.Visible = false

-- Desativa o aimbot
local function disableAimbot()
	aiming = false
	target = nil

	if characterRemovingConnection then
		characterRemovingConnection:Disconnect()
		characterRemovingConnection = nil
	end
end

-- Verifica se ponto está dentro do FOV
local function isInFOV(screenPos)
	local mousePos = Vector2.new(mouse.X, mouse.Y)
	return (screenPos - mousePos).Magnitude <= FOV_RADIUS
end

-- Busca o jogador mais próximo dentro do FOV
local function getClosestPlayerInFOV()
	local closestPlayer = nil
	local shortestDistance = FOV_RADIUS
	local mousePos = Vector2.new(mouse.X, mouse.Y)

	for _, otherPlayer in ipairs(Players:GetPlayers()) do
		if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("Head") then
			local head = otherPlayer.Character.Head
			local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
				if dist <= shortestDistance then
					shortestDistance = dist
					closestPlayer = otherPlayer
				end
			end
		end
	end

	return closestPlayer
end

-- Mira automática
RunService.RenderStepped:Connect(function()
	fovCircle.Position = Vector2.new(mouse.X, mouse.Y)

	if aimbotLoaded and aiming and target and target.Character and target.Character:FindFirstChild("Head") then
		local headPos = target.Character.Head.Position
		camera.CFrame = CFrame.new(camera.CFrame.Position, headPos)
	end
end)

-- Tecla X ativa/desativa
UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end
	if input.KeyCode == Enum.KeyCode.X and aimbotLoaded then
		if aiming then
			disableAimbot()
		else
			local possibleTarget = getClosestPlayerInFOV()
			if possibleTarget then
				target = possibleTarget
				aiming = true

				characterRemovingConnection = target.Character.AncestryChanged:Connect(function(_, parent)
					if not parent then
						disableAimbot()
					end
				end)
			end
		end
	end
end)

-- Carregamento com animação
task.spawn(function()
	for i = 1, 100 do
		local tween = TweenService:Create(progressBar, TweenInfo.new(0.03, Enum.EasingStyle.Linear), {
			Size = UDim2.new(i / 100, 0, 1, 0)
		})
		tween:Play()

		loadingText.Text = "Carregando: " .. i .. "%"
		wait(0.03)
	end

	-- Finalizar
	loadSound:Stop()
	fovCircle.Visible = true
	aimbotLoaded = true

	-- Animação de fade out do frame
	local fadeOut = TweenService:Create(loadingFrame, TweenInfo.new(0.5), {BackgroundTransparency = 1})
	local textFade = TweenService:Create(loadingText, TweenInfo.new(0.5), {TextTransparency = 1})
	local barFade = TweenService:Create(progressBar, TweenInfo.new(0.5), {BackgroundTransparency = 1})

	fadeOut:Play()
	textFade:Play()
	barFade:Play()
	wait(0.6)
	loadingFrame:Destroy()
end)
