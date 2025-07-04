-- FERVISKI HUB SCRIPT COMPLETO COM SISTEMA DE KEY E NOCLIP FUNCIONAL
repeat wait() until game:IsLoaded()
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- GUI de verificação da key
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "KeyGUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 180)
frame.Position = UDim2.new(0.5, -150, 0.5, -90)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Active = true

local keyBox = Instance.new("TextBox", frame)
keyBox.Size = UDim2.new(0.9, 0, 0, 40)
keyBox.Position = UDim2.new(0.05, 0, 0.15, 0)
keyBox.PlaceholderText = "Digite a key..."
keyBox.TextScaled = true
keyBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
keyBox.TextColor3 = Color3.new(1, 1, 1)

local getKey = Instance.new("TextButton", frame)
getKey.Size = UDim2.new(0.4, 0, 0, 40)
getKey.Position = UDim2.new(0.05, 0, 0.5, 0)
getKey.Text = "Get Key"
getKey.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
getKey.TextColor3 = Color3.new(1, 1, 1)
getKey.MouseButton1Click:Connect(function()
    setclipboard("ferviski") -- Sua key exclusiva
end)

local verifyKey = Instance.new("TextButton", frame)
verifyKey.Size = UDim2.new(0.4, 0, 0, 40)
verifyKey.Position = UDim2.new(0.55, 0, 0.5, 0)
verifyKey.Text = "Verify Key"
verifyKey.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
verifyKey.TextColor3 = Color3.new(1, 1, 1)

-- Função drag compatível com mouse e touch (emulador)
local function makeDraggable(frame, handle)
    local dragging, dragStart, startPos

    local function update(input)
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            UserInputService.InputChanged:Connect(function(moveInput)
                if dragging and moveInput == input then
                    update(moveInput)
                end
            end)
        end
    end)
end

makeDraggable(frame, frame)

-- Função para criar o Hub com as opções
local function createHub()
    gui:Destroy()

    local hub = Instance.new("ScreenGui", game.CoreGui)
    hub.Name = "FerviskiHub"

    local main = Instance.new("Frame", hub)
    main.Size = UDim2.new(0, 300, 0, 320)
    main.Position = UDim2.new(0.5, -150, 0.5, -160)
    main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    main.BorderSizePixel = 0
    main.Active = true

    local titleBar = Instance.new("Frame", main)
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    titleBar.BorderSizePixel = 0

    local title = Instance.new("TextLabel", titleBar)
    title.Size = UDim2.new(1, -30, 1, 0)
    title.Position = UDim2.new(0, 10, 0, 0)
    title.Text = "Ferviski Hub"
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.TextColor3 = Color3.new(1, 1, 1)
    title.BackgroundTransparency = 1

    local toggleBtn = Instance.new("TextButton", titleBar)
    toggleBtn.Size = UDim2.new(0, 30, 1, 0)
    toggleBtn.Position = UDim2.new(1, -30, 0, 0)
    toggleBtn.Text = "-"
    toggleBtn.TextColor3 = Color3.new(1, 1, 1)
    toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

    local content = Instance.new("Frame", main)
    content.Size = UDim2.new(1, 0, 1, -30)
    content.Position = UDim2.new(0, 0, 0, 30)
    content.BackgroundTransparency = 1
    content.Name = "Content"

    local layout = Instance.new("UIListLayout", content)
    layout.Padding = UDim.new(0, 8)
    layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
    layout.VerticalAlignment = Enum.VerticalAlignment.Top

    local function addButton(text, callback)
        local btn = Instance.new("TextButton", content)
        btn.Size = UDim2.new(0.9, 0, 0, 40)
        btn.Text = text
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.MouseButton1Click:Connect(callback)
    end

    -- Infinite Jump que só ativa quando o jogador pula
    local infJumpEnabled = false
    local jumpConn

    addButton("Toggle Infinite Jump", function()
        infJumpEnabled = not infJumpEnabled

        if infJumpEnabled then
            jumpConn = UserInputService.JumpRequest:Connect(function()
                local character = LocalPlayer.Character
                local humanoid = character and character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end)
        else
            if jumpConn then
                jumpConn:Disconnect()
                jumpConn = nil
            end
        end
    end)

    -- God Mode
    local godModeEnabled = false
    local godModeConn

    addButton("Toggle God Mode", function()
        godModeEnabled = not godModeEnabled
        local char = LocalPlayer.Character
        if not char then return end

        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end

        if godModeEnabled then
            -- Desativa colisão e faz invencível
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                    part.Anchored = false
                end
            end

            hum.MaxHealth = math.huge
            hum.Health = math.huge

            godModeConn = hum.HealthChanged:Connect(function()
                if hum.Health < math.huge then
                    hum.Health = math.huge
                end
            end)
        else
            -- Reativa colisão normal
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
            if godModeConn then
                godModeConn:Disconnect()
                godModeConn = nil
            end
        end
    end)

    -- Boost FPS
    addButton("Boost FPS", function()
        local lighting = game:GetService("Lighting")
        for _,v in pairs(workspace:GetDescendants()) do
            if v:IsA("Decal") or v:IsA("Texture") or v:IsA("ParticleEmitter") or v:IsA("Trail") then
                v:Destroy()
            elseif v:IsA("BasePart") then
                v.Material = Enum.Material.SmoothPlastic
                v.Reflectance = 0
            end
        end
        for _,v in pairs(lighting:GetChildren()) do
            if v:IsA("PostEffect") then v:Destroy() end
        end
        lighting.GlobalShadows = false
        lighting.FogEnd = 1e6
        lighting.Brightness = 1
        lighting.Ambient = Color3.fromRGB(128,128,128)
    end)

    -- Fast Speed toggle (100% mais rápido)
    local fastSpeedEnabled = false
    local normalWalkSpeed = 16

    local function applyFastSpeed()
        local char = LocalPlayer.Character
        if not char then return end

        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end

        if fastSpeedEnabled then
            hum.WalkSpeed = 32 -- dobro da velocidade normal
        else
            hum.WalkSpeed = normalWalkSpeed
        end
    end

    addButton("Toggle Fast Speed", function()
        fastSpeedEnabled = not fastSpeedEnabled
        applyFastSpeed()
    end)

    -- Mantém velocidade após respawn
    LocalPlayer.CharacterAdded:Connect(function()
        wait(1)
        applyFastSpeed()
    end)

    -- No Clip funcional (não deixa cair no vazio)
    local noClipEnabled = false
    local noClipConn

    local function enableNoClip()
        noClipEnabled = true
        local char = LocalPlayer.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        noClipConn = RunService.Stepped:Connect(function()
            if not noClipEnabled then
                noClipConn:Disconnect()
                return
            end
            if not char or not hrp then return end

            -- Desativa colisão em partes exceto HumanoidRootPart
            for _, part in pairs(char:GetChildren()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    part.CanCollide = false
                end
            end

            -- Raycast para manter o personagem "no chão"
            local rayOrigin = hrp.Position
            local rayDirection = Vector3.new(0, -50, 0) -- 50 studs para baixo
            local raycastParams = RaycastParams.new()
            raycastParams.FilterDescendantsInstances = {char}
            raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

            local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)
            if raycastResult then
                local groundPos = raycastResult.Position
                -- Mantém o personagem com um pequeno offset do chão (1.5 studs)
                hrp.CFrame = CFrame.new(hrp.Position.X, groundPos.Y + 1.5, hrp.Position.Z)
            end
        end)
    end

    local function disableNoClip()
        noClipEnabled = false
        local char = LocalPlayer.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        end
        for _, part in pairs(char:GetChildren()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
        if noClipConn then
            noClipConn:Disconnect()
            noClipConn = nil
        end
    end

    addButton("Toggle NoClip", function()
        if noClipEnabled then
            disableNoClip()
        else
            enableNoClip()
        end
    end)

    -- Minimizar/restaurar
    local minimized = false
    toggleBtn.MouseButton1Click:Connect(function()
        minimized = not minimized
        content.Visible = not minimized
        main.Size = minimized and UDim2.new(0, 300, 0, 30) or UDim2.new(0, 300, 0, 320)
        toggleBtn.Text = minimized and "□" or "-"
    end)

    -- Ativa o drag na barra de título (move janela inteira)
    makeDraggable(main, titleBar)
end

-- Verifica a key
verifyKey.MouseButton1Click:Connect(function()
    if keyBox.Text == "ferviski" then
        createHub()
    else
        keyBox.Text = "Key inválida"
    end
end)
