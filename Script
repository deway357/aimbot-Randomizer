-- SETTINGS
getgenv().AimbotInput = "RightClick"
getgenv().AimbotEasing = 1
getgenv().TeamCheck = false
getgenv().FOVRadius = 150

-- Show Loading Message
local function showLoadingMessage()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "LoadingGui"
    screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(0, 400, 0, 100)
    textLabel.Position = UDim2.new(0.5, -200, 0.5, -50)
    textLabel.BackgroundTransparency = 0.5
    textLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    textLabel.Text = "Please Wait whilst it loads..."
    textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    textLabel.TextSize = 20
    textLabel.Parent = screenGui

    -- Optionally, you can set a timeout for the message to disappear after some time
    wait(3)  -- Adjust the time as needed
    screenGui:Destroy()  -- Remove the loading message after 3 seconds
end

-- Call the function before running the script to show the loading message
showLoadingMessage()

-- INTERNALS
if getgenv().AlreadyRanItBroDanger then return end
getgenv().AlreadyRanItBroDanger = true

local uis = game:GetService("UserInputService")
local players = game:GetService("Players")
local lp = players.LocalPlayer
local mouse = lp:GetMouse()
local cam = workspace.CurrentCamera

-- FOV CIRCLE (Green)
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = getgenv().FOVRadius
fovCircle.Color = Color3.fromRGB(0, 255, 0)  -- Green
fovCircle.Thickness = 1
fovCircle.Transparency = 0.5
fovCircle.Visible = true
fovCircle.Filled = false

-- Clear existing ESP
for _, v in pairs(game:GetService("CoreGui"):GetChildren()) do
    if v.Name == "HighlightESP" then
        v:Destroy()
    end
end

-- ESP FUNCTION (Red Outline)
local function applyESP(player)
    if player == lp then return end

    local function add()
        local char = player.Character
        if not char then return end
        if char:FindFirstChild("HighlightESP") then return end

        -- Ensure parts exist
        local head = char:FindFirstChild("Head") or char:WaitForChild("Head", 5)
        if not head then return end

        local h = Instance.new("Highlight")
        h.Name = "HighlightESP"
        h.Adornee = char
        h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        h.FillColor = Color3.fromRGB(255, 0, 0)  -- Red outline
        h.FillTransparency = 1  -- Only outline
        h.OutlineColor = Color3.new(1, 1, 1)  -- White outline color
        h.OutlineTransparency = 0
        h.Parent = char  -- Parent to the character instead of CoreGui
    end

    -- Character spawns
    player.CharacterAdded:Connect(function()
        task.wait(1)
        add()
    end)

    -- Apply immediately if possible
    if player.Character then
        task.spawn(function()
            task.wait(1)
            add()
        end)
    end
end

-- Apply ESP to all players
for _, plr in pairs(players:GetPlayers()) do
    applyESP(plr)
end
players.PlayerAdded:Connect(applyESP)

-- VISIBILITY CHECK
local function isVisible(part, model)
    local origin = cam.CFrame.Position
    local _, onScreen = cam:WorldToViewportPoint(part.Position)
    if not onScreen then return false end
    local ray = Ray.new(origin, (part.Position - origin))
    local hit = workspace:FindPartOnRayWithIgnoreList(ray, lp.Character:GetDescendants())
    return hit and hit:IsDescendantOf(model)
end

-- GET CLOSEST TARGET
local function getClosestPlayer()
    local closest = nil
    local shortest = math.huge
    for _, v in pairs(players:GetPlayers()) do
        if v ~= lp and v.Character and v.Character:FindFirstChild("Head") and v.Character:FindFirstChild("Humanoid") and v.Character.Humanoid.Health > 0 then
            if not getgenv().TeamCheck or v.Team ~= lp.Team then
                local pos, onScreen = cam:WorldToViewportPoint(v.Character.Head.Position)
                if onScreen then
                    local dist = (Vector2.new(mouse.X, mouse.Y) - Vector2.new(pos.X, pos.Y)).Magnitude
                    if dist < getgenv().FOVRadius and dist < shortest and isVisible(v.Character.Head, v.Character) then
                        closest = v
                        shortest = dist
                    end
                end
            end
        end
    end
    return closest
end

-- AIMBOT TOGGLE
local aiming = false
uis.InputBegan:Connect(function(i, g)
    if not g then
        if getgenv().AimbotInput == "RightClick" and i.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = true
        elseif getgenv().AimbotInput == "LeftClick" and i.UserInputType == Enum.UserInputType.MouseButton1 then
            aiming = true
        elseif i.KeyCode.Name == getgenv().AimbotInput then
            aiming = true
        end
    end
end)

uis.InputEnded:Connect(function(i, g)
    if not g then
        if getgenv().AimbotInput == "RightClick" and i.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = false
        elseif getgenv().AimbotInput == "LeftClick" and i.UserInputType == Enum.UserInputType.MouseButton1 then
            aiming = false
        elseif i.KeyCode.Name == getgenv().AimbotInput then
            aiming = false
        end
    end
end)

-- MAIN LOOP
game:GetService("RunService").RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(mouse.X, mouse.Y)

    if aiming then
        local target = getClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            local head = target.Character.Head.Position
            cam.CFrame = cam.CFrame:Lerp(CFrame.new(cam.CFrame.Position, head), getgenv().AimbotEasing)
        end
    end
end)
