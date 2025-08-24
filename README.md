-- Coloque este script em StarterPlayerScripts ou StarterGui como LocalScript

local player = game.Players.LocalPlayer
local autoStealEnabled = false

-- Função para criar botão na tela
local function createButton()
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 120, 0, 40)
    button.Position = UDim2.new(0, 10, 0, 10)
    button.Text = "Auto Steal: OFF"
    button.Parent = game:GetService("Players").LocalPlayer.PlayerGui:WaitForChild("ScreenGui", 3) or Instance.new("ScreenGui", player.PlayerGui)
    button.MouseButton1Click:Connect(function()
        autoStealEnabled = not autoStealEnabled
        button.Text = autoStealEnabled and "Auto Steal: ON" or "Auto Steal: OFF"
    end)
    return button
end

-- Função para detectar proximidade ao Brainrot
local function checkProximity()
    while true do
        if autoStealEnabled then
            -- Substitua "Brainrot" pelo nome do objeto/NPC do jogo
            local brainrot = workspace:FindFirstChild("Brainrot")
            if brainrot and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local distance = (brainrot.Position - player.Character.HumanoidRootPart.Position).Magnitude
                if distance < 10 then -- distância para roubar (ajuste conforme necessário)
                    -- Evento de roubo (substitua pelo evento real do jogo)
                    -- Por exemplo, se for um RemoteEvent:
                    -- game.ReplicatedStorage:FindFirstChild("StealEvent"):FireServer(brainrot)
                    
                    print("Roubando Brainrot!") -- Troque pelo que faz o roubo no jogo
                end
            end
        end
        wait(0.5) -- Checa a cada meio segundo
    end
end

-- Cria botão na tela
createButton()

-- Inicia verificação de proximidade
spawn(checkProximity)
