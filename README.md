-- Piso que segue o jogador (ligar/desligar por botão e tecla F)
-- Use APENAS em jogos seus (Roblox Studio). Não funciona como exploit em jogos de outras pessoas.

-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

-- Referências do jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local function getHRP()
	local c = player.Character or player.CharacterAdded:Wait()
	return c:WaitForChild("HumanoidRootPart")
end

-- Estado
local floorActive = false
local followConn -- RBXScriptConnection para seguir o jogador
local floorPart -- a peça do piso

-- Cria (ou garante) o piso
local function ensureFloor()
	if floorPart and floorPart.Parent then return end

	floorPart = Instance.new("Part")
	floorPart.Name = "PlayerFloor_" .. player.Name
	floorPart.Size = Vector3.new(12, 1, 12)
	floorPart.Anchored = true
	floorPart.CanCollide = false -- começa desligado
	floorPart.Transparency = 1   -- invisível quando desligado
	floorPart.Material = Enum.Material.Neon
	floorPart.BrickColor = BrickColor.new("Bright blue")
	floorPart.TopSurface = Enum.SurfaceType.Smooth
	floorPart.BottomSurface = Enum.SurfaceType.Smooth
	floorPart.Parent = workspace
end

-- Atualiza posição do piso para ficar logo abaixo do jogador
local function updateFloorPosition()
	if not floorPart or not floorPart.Parent then return end
	local hrp = getHRP()
	-- 4 studs abaixo do centro do corpo evita empurrões estranhos
	floorPart.CFrame = hrp.CFrame * CFrame.new(0, -4, 0)
end

-- Liga o piso
local function activateFloor()
	ensureFloor()
	floorActive = true
	floorPart.CanCollide = true
	floorPart.Transparency = 0.2

	-- Segue o jogador todo frame no cliente (suave)
	if followConn then followConn:Disconnect() end
	followConn = RunService.RenderStepped:Connect(function()
		if floorActive then
			updateFloorPosition()
		end
	end)
	updateFloorPosition()
end

-- Desliga o piso
local function deactivateFloor()
	floorActive = false
	if followConn then
		followConn:Disconnect()
		followConn = nil
	end
	if floorPart and floorPart.Parent then
		-- Em vez de destruir, deixamos pronto para ligar de novo
		floorPart.CanCollide = false
		floorPart.Transparency = 1
	end
end

-- Alterna
local function toggleFloor()
	if floorActive then
		deactivateFloor()
	else
		activateFloor()
	end
	updateButtonUI()
end

-- Reage a respawns: mantém a funcionalidade
player.CharacterAdded:Connect(function(newChar)
	character = newChar
	if floorActive then
		-- Reposiciona o piso para o novo personagem
		task.defer(updateFloorPosition)
	end
end)

----------------------------------------------------------------
-- GUI: botão na tela
----------------------------------------------------------------
local function createGui()
	local gui = Instance.new("ScreenGui")
	gui.Name = "FloorToggleGui"
	gui.ResetOnSpawn = false
	gui.IgnoreGuiInset = true
	gui.Parent = player:WaitForChild("PlayerGui")

	local btn = Instance.new("TextButton")
	btn.Name = "ToggleButton"
	btn.Size = UDim2.fromOffset(140, 48)
	btn.Position = UDim2.new(1, -156, 1, -68) -- canto inferior direito
	btn.AnchorPoint = Vector2.new(0, 0)
	btn.TextScaled = true
	btn.Font = Enum.Font.GothamBold
	btn.BackgroundTransparency = 0.1
	btn.Text = "Piso: OFF"
	btn.Parent = gui

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 12)
	corner.Parent = btn

	local stroke = Instance.new("UIStroke")
	stroke.Thickness = 2
	stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	stroke.Parent = btn

	btn.MouseButton1Click:Connect(function()
		toggleFloor()
	end)
end

function updateButtonUI()
	local gui = player:FindFirstChild("PlayerGui")
	if not gui then return end
	local screen = gui:FindFirstChild("FloorToggleGui")
	if not screen then return end
	local btn = screen:FindFirstChild("ToggleButton")
	if not btn then return end
	btn.Text = floorActive and "Piso: ON" or "Piso: OFF"
end

createGui()
updateButtonUI()

----------------------------------------------------------------
-- Atalho no teclado: tecla F
----------------------------------------------------------------
UIS.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.F then
		toggleFloor()
	end
end)
