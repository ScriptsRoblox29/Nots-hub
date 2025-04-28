local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local running = false
local attackLoop

local gui = Instance.new("ScreenGui", game.CoreGui)
local button = Instance.new("TextButton", gui)
local stroke = Instance.new("UIStroke", button)
local uicorner = Instance.new("UICorner", button)

button.Size = UDim2.new(0, 200, 0, 50)
button.Position = UDim2.new(0, 100, 0, 100)
button.Text = "Disabled"
button.TextColor3 = Color3.new(0, 0, 0)
button.BackgroundColor3 = Color3.fromRGB(173, 216, 230)
button.BackgroundTransparency = 0.25
button.Active = true
button.Draggable = true

stroke.Thickness = 2
stroke.Color = Color3.new(255, 255, 255)
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
stroke.Transparency = 0.25

uicorner.CornerRadius = UDim.new(0, 0.4)

local function equipTool()
    local backpack = LocalPlayer:FindFirstChildOfClass("Backpack")
    if backpack and not Character:FindFirstChild("SlapTool") then
        local tool = backpack:FindFirstChild("SlapTool")
        if tool then
            tool.Parent = Character
        end
    end
end

local function claimSlapTool()
    local args = { [1] = "SlapTool" }
    ReplicatedStorage:WaitForChild("ClaimReward"):FireServer(unpack(args))
end

local function startAttack()
    attackLoop = task.spawn(function()
        while running do
            claimSlapTool()
            equipTool()
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                    local args = {
                        [1] = "slash",
                        [2] = player.Character,
                        [3] = player.Character.HumanoidRootPart.Position
                    }
                    if Character:FindFirstChild("SlapTool") then
                        Character.SlapTool.Event:FireServer(unpack(args))
                    end
                end
            end
            task.wait()
        end
    end)
end

button.MouseButton1Click:Connect(function()
    running = not running
    if running then
        button.Text = "Activated"
        startAttack()
    else
        button.Text = "Disabled"
        if attackLoop then
            task.cancel(attackLoop)
        end
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
end)
