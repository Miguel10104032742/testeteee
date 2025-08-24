local player = game.Players.LocalPlayer
local autoStealEnabled = false
local button

local function createButton()
    button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 120, 0, 40)
    button.Position = UDim2.new(0, 0, 0, 0)
    button.Text = "Auto Steal: OFF"
    button.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    button.TextColor3 = Color3.new(1, 1, 1)

    local screenGui = Instance.new("ScreenGui", player.PlayerGui)
    screenGui.Name = "AutoStealGui"
    button.Parent = screenGui

    button.MouseButton1Click:Connect(function()
        autoStealEnabled = not autoStealEnabled
        button.Text = autoStealEnabled and "Auto Steal: ON" or "Auto Steal: OFF"
        button.BackgroundColor3 = autoStealEnabled and Color3.new(0.2, 0.6, 0.2) or Color3.new(0.2, 0.2, 0.2)
    end)
end

local function autoStealProximity()
    while true do
        if autoStealEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local charPos = player.Character.HumanoidRootPart.Position

            -- Procura todos os ProximityPrompts no workspace
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("ProximityPrompt") and obj.ObjectText and string.lower(obj.ObjectText):find("roubar") then
                    local parent = obj.Parent
                    local promptPos = nil
                    if parent and parent:IsA("Attachment") then
                        promptPos = parent.WorldPosition
                    elseif parent and parent:IsA("Part") then
                        promptPos = parent.Position
                    end
                    if promptPos then
                        local distance = (promptPos - charPos).Magnitude
                        if distance <= obj.MaxActivationDistance then
                            obj:InputHoldBegin()
                            wait(obj.HoldDuration or 1)
                            obj:InputHoldEnd()
                            print("Roubo automático realizado!")
                            wait(1)
                        end
                    end
                end
            end
        end
        wait(0.2)
    end
end

createButton()
spawn(autoStealProximity)
