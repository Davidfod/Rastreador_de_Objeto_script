local Player = game:GetService("Players").LocalPlayer
local Mouse = Player:GetMouse()
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local function Create(cls, props)
    local inst = Instance.new(cls)
    for i, v in pairs(props) do
        inst[i] = v
    end
    return inst
end

local ScreenGui = Create("ScreenGui", {
    Name = "DexFinder_CyberNoDry_V2",
    Parent = (game:GetService("CoreGui") or Player:FindFirstChild("PlayerGui")),
    ResetOnSpawn = false
})

-- Frame Principal com fundo Dark
local InfoFrame = Create("Frame", {
    Name = "InfoFrame",
    Parent = ScreenGui,
    BackgroundColor3 = Color3.fromRGB(15, 15, 15),
    BackgroundTransparency = 0.1,
    BorderSizePixel = 0,
    Position = UDim2.new(0.5, -200, 0.5, -70),
    Size = UDim2.new(0, 420, 0, 160),
    Visible = false,
    Active = true
})

Create("UICorner", { Parent = InfoFrame, CornerRadius = UDim.new(0, 12) })

-- BORDA LED RGB
local UIStroke = Create("UIStroke", {
    Parent = InfoFrame,
    Thickness = 2.5,
    ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
    Color = Color3.new(1, 1, 1) -- Inicial
})

local UIGradient = Create("UIGradient", {
    Parent = UIStroke,
    Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
        ColorSequenceKeypoint.new(0.2, Color3.fromRGB(255, 255, 0)),
        ColorSequenceKeypoint.new(0.4, Color3.fromRGB(0, 255, 0)),
        ColorSequenceKeypoint.new(0.6, Color3.fromRGB(0, 255, 255)),
        ColorSequenceKeypoint.new(0.8, Color3.fromRGB(0, 0, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 255))
    }
})

-- Barra de Título (Arraste)
local TitleBar = Create("Frame", {
    Parent = InfoFrame,
    Size = UDim2.new(1, 0, 0, 35),
    BackgroundColor3 = Color3.fromRGB(25, 25, 25),
    BackgroundTransparency = 0.5,
    BorderSizePixel = 0
})
Create("UICorner", { Parent = TitleBar, CornerRadius = UDim.new(0, 12) })

local Title = Create("TextLabel", {
    Parent = TitleBar,
    Text = "🔍 RASTREADOR DE OBJETO | CyberNoDry",
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundTransparency = 1,
    TextColor3 = Color3.new(1, 1, 1),
    Font = Enum.Font.GothamBold,
    TextSize = 14
})

local Details = Create("TextLabel", {
    Name = "Details",
    Parent = InfoFrame,
    Position = UDim2.new(0, 15, 0, 45),
    Size = UDim2.new(1, -30, 1, -60),
    BackgroundTransparency = 1,
    TextColor3 = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.Code,
    TextSize = 14,
    TextXAlignment = Enum.TextXAlignment.Left,
    TextYAlignment = Enum.TextYAlignment.Top,
    TextWrapped = true,
    RichText = true
})

local Highlight = Create("Highlight", {
    Parent = ScreenGui,
    FillColor = Color3.fromRGB(255, 255, 255),
    FillTransparency = 0.5,
    OutlineColor = Color3.new(1, 1, 1)
})

-- ANIMAÇÃO LED (Girar Cores)
RunService.RenderStepped:Connect(function()
    UIGradient.Rotation = UIGradient.Rotation + 2
    Highlight.FillColor = Color3.fromHSV(tick() % 5 / 5, 1, 1) -- Highlight também muda de cor!
end)

-- Sistema de Arraste
local dragging, dragInput, dragStart, startPos
local function update(input)
    local delta = input.Position - dragStart
    InfoFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = InfoFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then update(input) end
end)

-- Lógica de Clique
Mouse.Button1Down:Connect(function()
    task.wait(0.05)
    if dragging then return end 

    local target = Mouse.Target
    if target then
        Highlight.Adornee = target
        InfoFrame.Visible = true
        
        local fullPath = target:GetFullName()
        local parentObj = target.Parent
        
        Details.Text = string.format(
            "<b>OBJETO:</b> <font color='#00FF00'>%s</font>\n" ..
            "<b>CLASSE:</b> %s\n" ..
            "<b>DONO:</b> %s\n" ..
            "<b>LOCAL:</b> <font color='#FFFF00' size='11'>%s</font>",
            target.Name, 
            target.ClassName,
            parentObj and parentObj.Name or "N/A", 
            fullPath
        )
    else
        InfoFrame.Visible = false
        Highlight.Adornee = nil
    end
end)
