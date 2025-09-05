-- Serviços
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

-- Configurações
local SpeedValue = 45 -- Velocidade segura e indetectável
local FlySpeed = 45
local TPSpeed = 70
local BrainrotName = "Brainrot"
local BaseName = "Base"

-- Estado
local ESPEnabled, TPBrainrotEnabled, FlyEnabled, SpeedEnabled = false, false, false, false
local ESPDrawings, RenderConn, TPConn, FlyConn, SpeedConn = {}, nil, nil, nil, nil

---------------------------------------------------
-- Função para limpar ESP
---------------------------------------------------
local function ClearESP()
    for _, v in ipairs(ESPDrawings) do
        if v.box then pcall(function() v.box:Remove() end) end
    end
    ESPDrawings = {}
end

---------------------------------------------------
-- Atualiza lista de ESP
---------------------------------------------------
local function UpdateESP()
    ClearESP()
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local box = Drawing.new("Square")
            box.Color = Color3.fromRGB(255, 255, 255)
            box.Thickness = 2
            box.Filled = false
            box.Transparency = 1
            table.insert(ESPDrawings, {player = plr, box = box})
        end
    end
end

---------------------------------------------------
-- Renderiza o ESP
---------------------------------------------------
local function RenderESP()
    local cam = Workspace.CurrentCamera
    for _, bundle in ipairs(ESPDrawings) do
        local char = bundle.player.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local root = char.HumanoidRootPart
            local pos, onscreen = cam:WorldToViewportPoint(root.Position)
            if onscreen then
                local size = Vector2.new(50, 100)
                bundle.box.Size = size
                bundle.box.Position = Vector2.new(pos.X - size.X / 2, pos.Y - size.Y / 2)
                bundle.box.Visible = true
            else
                bundle.box.Visible = false
            end
        end
    end
end

---------------------------------------------------
-- Ativa ESP
---------------------------------------------------
local function EnableESP()
    UpdateESP()
    if RenderConn then RenderConn:Disconnect() end
    RenderConn = RunService.RenderStepped:Connect(function()
        if ESPEnabled then
            RenderESP()
        else
            ClearESP()
        end
    end)
end

---------------------------------------------------
-- Função para encontrar sua base
---------------------------------------------------
local function FindMyBase()
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and obj.Name == BaseName and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            if (obj.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude < 1000 then
                return obj
            end
        end
    end
    return nil
end

---------------------------------------------------
-- TP Brainrot fixado
---------------------------------------------------
local function EnableTPBrainrot()
    if TPConn then TPConn:Disconnect() end
    TPConn = RunService.Heartbeat:Connect(function()
        if not TPBrainrotEnabled then return end
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        local hrp = char.HumanoidRootPart

        -- Só ativa se tiver Brainrot
        if not LocalPlayer.Backpack:FindFirstChild(BrainrotName) and not char:FindFirstChild(BrainrotName) then return end

        local base = FindMyBase()
        if base then
            local targetPos = base.Position + Vector3.new(0, 5, 0)
            local dir = (targetPos - hrp.Position)
            if dir.Magnitude > 1 then
                -- Move de forma segura (não teleporta)
                hrp.CFrame = hrp.CFrame + dir.Unit * math.min(dir.Magnitude, TPSpeed * RunService.Heartbeat:Wait())
            else
                -- Chegou na base, desativa TP
                TPBrainrotEnabled = false
            end
        end
    end)
end

---------------------------------------------------
-- Speed Indetectável (usando AssemblyLinearVelocity)
---------------------------------------------------
local function EnableSpeed()
    if SpeedConn then SpeedConn:Disconnect() end
    SpeedConn = RunService.Heartbeat:Connect(function()
        if not SpeedEnabled then return end
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") or UIS:GetFocusedTextBox() then return end

        local hrp = char.HumanoidRootPart
        local moveDirection = Vector3.new()

        local camCF = Workspace.CurrentCamera.CFrame
        local forward = Vector3.new(camCF.LookVector.X, 0, camCF.LookVector.Z).Unit
        local right = Vector3.new(camCF.RightVector.X, 0, camCF.RightVector.Z).Unit

        if UIS:IsKeyDown(Enum.KeyCode.W) then moveDirection += forward end
        if UIS:IsKeyDown(Enum.KeyCode.S) then moveDirection -= forward end
        if UIS:IsKeyDown(Enum.KeyCode.D) then moveDirection += right end
        if UIS:IsKeyDown(Enum.KeyCode.A) then moveDirection -= right end

        if moveDirection.Magnitude > 0 then
            hrp.AssemblyLinearVelocity = Vector3.new(moveDirection.Unit.X * SpeedValue, hrp.AssemblyLinearVelocity.Y, moveDirection.Unit.Z * SpeedValue)
        else
            hrp.AssemblyLinearVelocity = Vector3.new(0, hrp.AssemblyLinearVelocity.Y, 0)
        end
    end)
end

---------------------------------------------------
-- Fly
---------------------------------------------------
local function EnableFly()
    if FlyConn then FlyConn:Disconnect() end
    FlyConn = RunService.RenderStepped:Connect(function()
        if not FlyEnabled then return end
        local char = LocalPlayer.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") then return end
        local hrp = char.HumanoidRootPart
        local move = Vector3.new()
        if UIS:IsKeyDown(Enum.KeyCode.W) then move += Workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then move -= Workspace.CurrentCamera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then move -= Workspace.CurrentCamera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then move += Workspace.CurrentCamera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then move += Vector3.new(0, 1, 0) end
        if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then move -= Vector3.new(0, 1, 0) end
        hrp.Velocity = move.Magnitude > 0 and move.Unit * FlySpeed or Vector3.new(0, 0, 0)
    end)
end

---------------------------------------------------
-- Criar ScreenGui principal
---------------------------------------------------
local screenGui = Instance.new("ScreenGui", CoreGui)
screenGui.Name = "BolinhaGui"

---------------------------------------------------
-- Criar bolinha roxa arredondada
---------------------------------------------------
local bolinha = Instance.new("ImageButton", screenGui)
bolinha.Name = "Bolinha"
bolinha.Size = UDim2.new(0, 50, 0, 50)
bolinha.Position = UDim2.new(0, 20, 0, 20)
bolinha.BackgroundColor3 = Color3.fromRGB(128, 0, 128)
bolinha.BorderSizePixel = 0
bolinha.Image = ""

local uicorner = Instance.new("UICorner", bolinha)
uicorner.CornerRadius = UDim.new(1, 0)

---------------------------------------------------
-- Criar painel do SECRET SCRIPT dentro do screenGui
---------------------------------------------------
local painel = Instance.new("Frame", screenGui)
painel.Name = "Painel"
painel.Size = UDim2.new(0, 320, 0, 380)
painel.Position = UDim2.new(0, 80, 0, 20)
painel.BackgroundColor3 = Color3.fromHex("#7d18f0")
painel.BorderSizePixel = 0
painel.Visible = false
painel.Active = true
painel.Draggable = true

local corner = Instance.new("UICorner", painel)
corner.CornerRadius = UDim.new(0, 15)

local title = Instance.new("TextLabel", painel)
title.Size = UDim2.new(1, 0, 0, 45)
title.Text = "SECRET SCRIPT V6"
title.Font = Enum.Font.GothamBlack
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.BackgroundTransparency = 1

local status = Instance.new("TextLabel", painel)
status.Size = UDim2.new(1, 0, 0, 25)
status.Position = UDim2.new(0, 0, 0, 50)
status.BackgroundTransparency = 1
status.Text = "🔴 Todos desativados"
status.Font = Enum.Font.Gotham
status.TextColor3 = Color3.fromRGB(255, 255, 255)
status.TextScaled = true

---------------------------------------------------
-- Função para atualizar o status dos cheats
---------------------------------------------------
local function updateStatus()
    local ativos = {}
    if ESPEnabled then table.insert(ativos, "ESP") end
    if TPBrainrotEnabled then table.insert(ativos, "TPBrainrot") end
    if FlyEnabled then table.insert(ativos, "Fly") end
    if SpeedEnabled then table.insert(ativos, "Speed") end
    status.Text = (#ativos > 0 and "🟢 Ativos: " .. table.concat(ativos, " | ") or "🔴 Todos desativados")
end

---------------------------------------------------
-- Função para criar botões dentro do painel
---------------------------------------------------
local function makeBtn(name, ypos, callback)
    local btn = Instance.new("TextButton", painel)
    btn.Size = UDim2.new(0, 280, 0, 40)
    btn.Position = UDim2.new(0, 20, 0, ypos)
    btn.BackgroundColor3 = Color3.fromHex("#9264f5")
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 20
    btn.Text = name .. " [OFF]"
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)

    local cornerBtn = Instance.new("UICorner", btn)
    cornerBtn.CornerRadius = UDim.new(0, 10)

    btn.MouseButton1Click:Connect(function()
        callback(btn)
        updateStatus()
    end)

    return btn
end

---------------------------------------------------
-- Criar botões principais
---------------------------------------------------
local btnESP = makeBtn("ESP", 90, function(btn)
    ESPEnabled = not ESPEnabled
    btn.Text = "ESP [" .. (ESPEnabled and "ON" or "OFF") .. "]"
    if ESPEnabled then EnableESP() else ClearESP() end
end)

local btnTP = makeBtn("TP Brainrot", 140, function(btn)
    TPBrainrotEnabled = not TPBrainrotEnabled
    btn.Text = "TP Brainrot [" .. (TPBrainrotEnabled and "ON" or "OFF") .. "]"
    if TPBrainrotEnabled then EnableTPBrainrot() end
end)

local btnFly = makeBtn("Fly", 190, function(btn)
    FlyEnabled = not FlyEnabled
    btn.Text = "Fly [" .. (FlyEnabled and "ON" or "OFF") .. "]"
    if FlyEnabled then EnableFly() else if FlyConn then FlyConn:Disconnect() end end
end)

local btnSpeed = makeBtn("Speed", 240, function(btn)
    SpeedEnabled = not SpeedEnabled
    btn.Text = "Speed [" .. (SpeedEnabled and "ON" or "OFF") .. "]"
    if SpeedEnabled then EnableSpeed() else if SpeedConn then SpeedConn:Disconnect() end end
end)

local btnClose = Instance.new("TextButton", painel)
btnClose.Size = UDim2.new(0, 40, 0, 40)
btnClose.Position = UDim2.new(1, -50, 0, 10)
btnClose.Text = "X"
btnClose.TextColor3 = Color3.fromRGB(255, 255, 255)
btnClose.Font = Enum.Font.GothamBold
btnClose.TextSize = 24
btnClose.BackgroundColor3 = Color3.fromHex("#d93e48")

local cornerClose = Instance.new("UICorner", btnClose)
cornerClose.CornerRadius = UDim.new(0, 10)

btnClose.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if RenderConn then RenderConn:Disconnect() end
    if TPConn then TPConn:Disconnect() end
    if FlyConn then FlyConn:Disconnect() end
    if SpeedConn then SpeedConn:Disconnect() end
end)

---------------------------------------------------
-- Toggle painel ao clicar na bolinha
---------------------------------------------------
bolinha.MouseButton1Click:Connect(function()
    painel.Visible = not painel.Visible
end)

---------------------------------------------------
-- Código para arrastar a bolinha
---------------------------------------------------
local dragging = false
local dragInput
local dragStart
local startPos

bolinha.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = bolinha.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

bolinha.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        bolinha.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

---------------------------------------------------
-- Início
---------------------------------------------------
updateStatus()
