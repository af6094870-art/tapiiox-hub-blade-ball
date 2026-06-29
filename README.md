local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "tapiiox hub",
    SubTitle = "by mr",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = false, -- Desativa o efeito blur do Windows 11
    Theme = "Darker", -- CORRIGIDO: "Sakura" não existe na biblioteca original. Use "Darker", "Dark", "Aqua" ou "Light"
    MinimizeKey = Enum.KeyCode.RightControl
})

-- Força o Fluent original a tirar a transparência do fundo
Fluent:ToggleTransparency(false)

-- Notificação apenas para testar se carregou
Fluent:Notify({
    Title = "Tapiiox hub",
    Content = "welcome tapiiox hub!",
    Duration = 3
})
local ScreenGui = Instance.new("ScreenGui")
local ToggleBtn = Instance.new("ImageButton")
local UICorner = Instance.new("UICorner")

ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.ResetOnSpawn = true

ToggleBtn.Parent = ScreenGui
ToggleBtn.Size = UDim2.new(0, 60, 0, 60)
ToggleBtn.Position = UDim2.new(0, 20, 0.5, 0)
ToggleBtn.BackgroundTransparency = 1
ToggleBtn.Image = "rbxassetid://132313789987260"

UICorner.CornerRadius = UDim.new(1, 0)
UICorner.Parent = ToggleBtn

-- Abre/fecha igual RightControl
ToggleBtn.MouseButton1Click:Connect(function()
    game:GetService("VirtualInputManager"):SendKeyEvent(true, Enum.KeyCode.RightControl, false, game)
    task.wait(0.1)
    game:GetService("VirtualInputManager"):SendKeyEvent(false, Enum.KeyCode.RightControl, false, game)
end)

-- Arrastar a bolinha
local dragging, dragStart, startPos

ToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = ToggleBtn.Position
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        ToggleBtn.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        ToggleBtn.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X, 
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
-- ========================================================
-- 1. PRIMEIRO: CRIA TODAS AS ABAS DO MENU
-- ========================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" })
}

-- [[ GERENCIADORES INTERNOS ]]
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:SetIgnoreIndexes({})

local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local Workspace = game:GetService("Workspace")

local Player = Players.LocalPlayer or Players.PlayerAdded:Wait()
local Balls = Workspace:WaitForChild("Balls", 9e9)

_G.AutoParryActive = false

-- Funções internas do seu script
local function VerifyBall(Ball)
    if typeof(Ball) == "Instance" and Ball:IsA("BasePart") and Ball:IsDescendantOf(Balls) and Ball:GetAttribute("realBall") == true then
        return true
    end
end

local function IsTarget()
    return (Player.Character and Player.Character:FindFirstChild("Highlight"))
end

local function Parry()
    -- Só clica se o Toggle ainda estiver ativo
    if _G.AutoParryActive then
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
        VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
    end
end

-- Evento principal (fica escutando as bolas novas)
Balls.ChildAdded:Connect(function(Ball)
    if not VerifyBall(Ball) then
        return
    end

    local OldPosition = Ball.Position
    local OldTick = tick()

    Ball:GetPropertyChangedSignal("Position"):Connect(function()
        -- Só faz o cálculo e o parry se o Toggle estiver ligado no menu
        if _G.AutoParryActive and IsTarget() then
            local Distance = (Ball.Position - Workspace.CurrentCamera.Focus.Position).Magnitude
            local Velocity = (OldPosition - Ball.Position).Magnitude

            if (Distance / Velocity) <= 10 then
                Parry()
            end
        end

        if (tick() - OldTick >= 1/60) then
            OldTick = tick()
            OldPosition = Ball.Position
        end
    end)
end)

-- TOGGLE: Auto Parry / Auto Rebatida
Tabs.Main:AddToggle("AutoParryToggle", {
    Title = "Auto Parry",
    Description = "Rebate a bola automaticamente quando ela estiver vindo na sua direção",
    Default = false,
    Callback = function(Value)
        _G.AutoParryActive = Value
    end
})
