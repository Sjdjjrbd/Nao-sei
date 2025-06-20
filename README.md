-- ========================================
-- SISTEMA DE CHAVE DE ATIVAÇÃO
-- ========================================
local CHAVE_SECRETA = "Comente!!"
local chaveValida = false

-- Função para verificar a chave
local function verificarChave()
    local Players = game:GetService("Players")
    local player = Players.LocalPlayer
    local playerGui = player:WaitForChild("PlayerGui")
    
    -- Criar GUI de verificação de chave
    local keyGui = Instance.new("ScreenGui")
    keyGui.Name = "KeyVerificationGUI"
    keyGui.ResetOnSpawn = false
    keyGui.Parent = playerGui
    
    local keyFrame = Instance.new("Frame")
    keyFrame.Size = UDim2.new(0, 400, 0, 200)
    keyFrame.Position = UDim2.new(0.5, -200, 0.5, -100)
    keyFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    keyFrame.BorderSizePixel = 0
    keyFrame.Parent = keyGui
    
    local keyCorner = Instance.new("UICorner")
    keyCorner.CornerRadius = UDim.new(0, 15)
    keyCorner.Parent = keyFrame
    
    local keyTitle = Instance.new("TextLabel")
    keyTitle.Size = UDim2.new(1, 0, 0, 50)
    keyTitle.Position = UDim2.new(0, 0, 0, 10)
    keyTitle.BackgroundTransparency = 1
    keyTitle.Text = "🔐 VERIFICAÇÃO DE CHAVE"
    keyTitle.TextColor3 = Color3.fromRGB(255, 100, 100)
    keyTitle.TextSize = 18
    keyTitle.Font = Enum.Font.GothamBold
    keyTitle.Parent = keyFrame
    
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -20, 0, 30)
    keyLabel.Position = UDim2.new(0, 10, 0, 60)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "Digite a chave de acesso para usar o Hitbox Expander:"
    keyLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyLabel.TextSize = 12
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.TextWrapped = true
    keyLabel.Parent = keyFrame
    
    local keyInput = Instance.new("TextBox")
    keyInput.Size = UDim2.new(1, -40, 0, 35)
    keyInput.Position = UDim2.new(0, 20, 0, 100)
    keyInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    keyInput.Text = ""
    keyInput.PlaceholderText = "Insira sua chave aqui..."
    keyInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    keyInput.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
    keyInput.TextSize = 14
    keyInput.Font = Enum.Font.Gotham
    keyInput.ClearTextOnFocus = false
    keyInput.Parent = keyFrame
    
    local inputCorner = Instance.new("UICorner")
    inputCorner.CornerRadius = UDim.new(0, 8)
    inputCorner.Parent = keyInput
    
    local verifyButton = Instance.new("TextButton")
    verifyButton.Size = UDim2.new(0, 120, 0, 30)
    verifyButton.Position = UDim2.new(0.5, -60, 0, 150)
    verifyButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)
    verifyButton.Text = "✅ VERIFICAR"
    verifyButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    verifyButton.TextSize = 12
    verifyButton.Font = Enum.Font.GothamBold
    verifyButton.Parent = keyFrame
    
    local buttonCorner = Instance.new("UICorner")
    buttonCorner.CornerRadius = UDim.new(0, 8)
    buttonCorner.Parent = verifyButton
    
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, -20, 0, 20)
    statusLabel.Position = UDim2.new(0, 10, 1, -25)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = ""
    statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
    statusLabel.TextSize = 11
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.Parent = keyFrame
    
    -- Função de verificação
    local function verificar()
        local chaveDigitada = keyInput.Text
        
        if chaveDigitada == CHAVE_SECRETA then
            chaveValida = true
            statusLabel.Text = "✅ Chave válida! Carregando script..."
            statusLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
            
            wait(1)
            keyGui:Destroy()
            carregarScript()
        else
            statusLabel.Text = "❌ Chave inválida! Tente novamente."
            statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            keyInput.Text = ""
        end
    end
    
    -- Eventos
    verifyButton.MouseButton1Click:Connect(verificar)
    keyInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            verificar()
        end
    end)
end

-- Função principal do script (só executa se a chave for válida)
function carregarScript()
    if not chaveValida then
        return
    end
    
    print("🔓 Chave verificada com sucesso! Carregando Hitbox Expander...")

-- ========================================
-- SCRIPT PRINCIPAL (SEU CÓDIGO ORIGINAL)
-- ========================================

-- Expander Hitbox NPC - VERSÃO CORRIGIDA PARA DANO
-- Resolve o problema de não dar dano após expandir hitbox
-- Versão: 2.3 - Dano funcionando + Sem bugs de física

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Configurações
local config = {
    hitboxSize = 20,
    transparency = 0.5,
    expanderEnabled = false,
    currentMode = 1,
    modes = {"Bots", "NPCs", "Players"},
    showVisual = true,
    isMinimized = false,
    autoDetectRange = 50,
    autoDetectEnabled = true,
    -- NOVA CONFIGURAÇÃO PARA DANO
    damageMode = 1, -- 1 = Hitbox Separada, 2 = Hitbox Mesclada
    damageModes = {"Separada", "Mesclada"}
}

-- Variáveis para controle
local expandedParts = {}
local originalSizes = {}
local connections = {}
local gui = nil

-- Função para criar a interface (ATUALIZADA)
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "HitboxExpanderGUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 320, 0, 290) -- Aumentado para mostrar status da chave
    mainFrame.Position = UDim2.new(0, 20, 0, 20)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame
    
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 35)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0.8, 0, 1, 0)
    title.Position = UDim2.new(0, 10, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎯 Hitbox Expander v2.3 🔐 ATIVADO"
    title.TextColor3 = Color3.fromRGB(100, 255, 100)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar
    
    local minimizeButton = Instance.new("TextButton")
    minimizeButton.Size = UDim2.new(0, 30, 0, 25)
    minimizeButton.Position = UDim2.new(1, -35, 0, 5)
    minimizeButton.BackgroundColor3 = Color3.fromRGB(255, 193, 7)
    minimizeButton.Text = "−"
    minimizeButton.TextColor3 = Color3.fromRGB(0, 0, 0)
    minimizeButton.TextSize = 16
    minimizeButton.Font = Enum.Font.GothamBold
    minimizeButton.Parent = titleBar
    
    local minimizeCorner = Instance.new("UICorner")
    minimizeCorner.CornerRadius = UDim.new(0, 6)
    minimizeCorner.Parent = minimizeButton
    
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "ContentFrame"
    contentFrame.Size = UDim2.new(1, 0, 1, -35)
    contentFrame.Position = UDim2.new(0, 0, 0, 35)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    local creditsLabel = Instance.new("TextLabel")
    creditsLabel.Size = UDim2.new(1, 0, 0, 20)
    creditsLabel.Position = UDim2.new(0, 0, 0, 5)
    creditsLabel.BackgroundTransparency = 1
    creditsLabel.Text = "🔐 VERSÃO LICENCIADA - Chave Verificada"
    creditsLabel.TextColor3 = Color3.fromRGB(100, 255, 100)
    creditsLabel.TextSize = 10
    creditsLabel.Font = Enum.Font.Gotham
    creditsLabel.Parent = contentFrame
    
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 20)
    statusLabel.Position = UDim2.new(0, 0, 0, 25)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Status: Desativado | Alvos encontrados: 0"
    statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusLabel.TextSize = 11
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.Parent = contentFrame
    
    local modeButton = Instance.new("TextButton")
    modeButton.Size = UDim2.new(0.9, 0, 0, 25)
    modeButton.Position = UDim2.new(0.05, 0, 0, 50)
    modeButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
    modeButton.Text = "🎮 Modo: " .. config.modes[config.currentMode]
    modeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    modeButton.TextSize = 12
    modeButton.Font = Enum.Font.GothamSemibold
    modeButton.Parent = contentFrame
    
    local modeCorner = Instance.new("UICorner")
    modeCorner.CornerRadius = UDim.new(0, 6)
    modeCorner.Parent = modeButton
    
    -- NOVO: Botão de Modo de Dano
    local damageModeButton = Instance.new("TextButton")
    damageModeButton.Size = UDim2.new(0.9, 0, 0, 25)
    damageModeButton.Position = UDim2.new(0.05, 0, 0, 80)
    damageModeButton.BackgroundColor3 = Color3.fromRGB(255, 140, 0)
    damageModeButton.Text = "⚔️ Dano: " .. config.damageModes[config.damageMode]
    damageModeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    damageModeButton.TextSize = 12
    damageModeButton.Font = Enum.Font.GothamSemibold
    damageModeButton.Parent = contentFrame
    
    local damageModeCorner = Instance.new("UICorner")
    damageModeCorner.CornerRadius = UDim.new(0, 6)
    damageModeCorner.Parent = damageModeButton
    
    local sizeFrame = Instance.new("Frame")
    sizeFrame.Size = UDim2.new(0.9, 0, 0, 25)
    sizeFrame.Position = UDim2.new(0.05, 0, 0, 110)
    sizeFrame.BackgroundTransparency = 1
    sizeFrame.Parent = contentFrame
    
    local sizeLabel = Instance.new("TextLabel")
    sizeLabel.Size = UDim2.new(0.5, 0, 1, 0)
    sizeLabel.Position = UDim2.new(0, 0, 0, 0)
    sizeLabel.BackgroundTransparency = 1
    sizeLabel.Text = "📏 Tamanho:"
    sizeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    sizeLabel.TextSize = 11
    sizeLabel.Font = Enum.Font.Gotham
    sizeLabel.TextXAlignment = Enum.TextXAlignment.Left
    sizeLabel.Parent = sizeFrame
    
    local sizeInput = Instance.new("TextBox")
    sizeInput.Size = UDim2.new(0.45, 0, 1, 0)
    sizeInput.Position = UDim2.new(0.55, 0, 0, 0)
    sizeInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    sizeInput.Text = tostring(config.hitboxSize)
    sizeInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    sizeInput.TextSize = 11
    sizeInput.Font = Enum.Font.Gotham
    sizeInput.Parent = sizeFrame
    
    local sizeCorner = Instance.new("UICorner")
    sizeCorner.CornerRadius = UDim.new(0, 4)
    sizeCorner.Parent = sizeInput
    
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.9, 0, 0, 25)
    buttonFrame.Position = UDim2.new(0.05, 0, 0, 140)
    buttonFrame.BackgroundTransparency = 1
    buttonFrame.Parent = contentFrame
    
    local visualButton = Instance.new("TextButton")
    visualButton.Size = UDim2.new(0.48, 0, 1, 0)
    visualButton.Position = UDim2.new(0, 0, 0, 0)
    visualButton.BackgroundColor3 = config.showVisual and Color3.fromRGB(34, 139, 34) or Color3.fromRGB(120, 120, 120)
    visualButton.Text = "👁️ " .. (config.showVisual and "ON" or "OFF")
    visualButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    visualButton.TextSize = 10
    visualButton.Font = Enum.Font.Gotham
    visualButton.Parent = buttonFrame
    
    local visualCorner = Instance.new("UICorner")
    visualCorner.CornerRadius = UDim.new(0, 4)
    visualCorner.Parent = visualButton
    
    local autoButton = Instance.new("TextButton")
    autoButton.Size = UDim2.new(0.48, 0, 1, 0)
    autoButton.Position = UDim2.new(0.52, 0, 0, 0)
    autoButton.BackgroundColor3 = config.autoDetectEnabled and Color3.fromRGB(34, 139, 34) or Color3.fromRGB(120, 120, 120)
    autoButton.Text = "🔄 " .. (config.autoDetectEnabled and "AUTO" or "MAN")
    autoButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    autoButton.TextSize = 10
    autoButton.Font = Enum.Font.Gotham
    autoButton.Parent = buttonFrame
    
    local autoCorner = Instance.new("UICorner")
    autoCorner.CornerRadius = UDim.new(0, 4)
    autoCorner.Parent = autoButton
    
    local rangeFrame = Instance.new("Frame")
    rangeFrame.Size = UDim2.new(0.9, 0, 0, 25)
    rangeFrame.Position = UDim2.new(0.05, 0, 0, 170)
    rangeFrame.BackgroundTransparency = 1
    rangeFrame.Parent = contentFrame
    
    local rangeLabel = Instance.new("TextLabel")
    rangeLabel.Size = UDim2.new(0.5, 0, 1, 0)
    rangeLabel.Position = UDim2.new(0, 0, 0, 0)
    rangeLabel.BackgroundTransparency = 1
    rangeLabel.Text = "🎯 Range:"
    rangeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    rangeLabel.TextSize = 11
    rangeLabel.Font = Enum.Font.Gotham
    rangeLabel.TextXAlignment = Enum.TextXAlignment.Left
    rangeLabel.Parent = rangeFrame
    
    local rangeInput = Instance.new("TextBox")
    rangeInput.Size = UDim2.new(0.45, 0, 1, 0)
    rangeInput.Position = UDim2.new(0.55, 0, 0, 0)
    rangeInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    rangeInput.Text = tostring(config.autoDetectRange)
    rangeInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    rangeInput.TextSize = 11
    rangeInput.Font = Enum.Font.Gotham
    rangeInput.Parent = rangeFrame
    
    local rangeCorner = Instance.new("UICorner")
    rangeCorner.CornerRadius = UDim.new(0, 4)
    rangeCorner.Parent = rangeInput
    
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.9, 0, 0, 30)
    toggleButton.Position = UDim2.new(0.05, 0, 0, 200)
    toggleButton.BackgroundColor3 = Color3.fromRGB(220, 20, 60)
    toggleButton.Text = "🚀 ATIVAR EXPANDER"
    toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleButton.TextSize = 13
    toggleButton.Font = Enum.Font.GothamBold
    toggleButton.Parent = contentFrame
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 8)
    toggleCorner.Parent = toggleButton
    
    -- Função para minimizar/maximizar
    local function toggleMinimize()
        config.isMinimized = not config.isMinimized
        
        if config.isMinimized then
            contentFrame.Visible = false
            mainFrame:TweenSize(UDim2.new(0, 320, 0, 35), "Out", "Quad", 0.3, true)
            minimizeButton.Text = "+"
        else
            mainFrame:TweenSize(UDim2.new(0, 320, 0, 290), "Out", "Quad", 0.3, true)
            wait(0.3)
            contentFrame.Visible = true
            minimizeButton.Text = "−"
        end
    end
    
    -- Eventos da GUI
    minimizeButton.MouseButton1Click:Connect(toggleMinimize)
    
    modeButton.MouseButton1Click:Connect(function()
        config.currentMode = config.currentMode + 1
        if config.currentMode > #config.modes then
            config.currentMode = 1
        end
        modeButton.Text = "🎮 Modo: " .. config.modes[config.currentMode]
        
        if config.expanderEnabled then
            stopExpander()
            startExpander()
        end
    end)
    
    -- NOVO: Evento do botão de modo de dano
    damageModeButton.MouseButton1Click:Connect(function()
        config.damageMode = config.damageMode + 1
        if config.damageMode > #config.damageModes then
            config.damageMode = 1
        end
        damageModeButton.Text = "⚔️ Dano: " .. config.damageModes[config.damageMode]
        
        if config.expanderEnabled then
            stopExpander()
            startExpander()
        end
    end)
    
    sizeInput.FocusLost:Connect(function()
        local newSize = tonumber(sizeInput.Text)
        if newSize and newSize >= 5 and newSize <= 100 then
            config.hitboxSize = newSize
            if config.expanderEnabled then
                updateAllHitboxes()
            end
        else
            sizeInput.Text = tostring(config.hitboxSize)
        end
    end)
    
    rangeInput.FocusLost:Connect(function()
        local newRange = tonumber(rangeInput.Text)
        if newRange and newRange >= 10 and newRange <= 200 then
            config.autoDetectRange = newRange
        else
            rangeInput.Text = tostring(config.autoDetectRange)
        end
    end)
    
    visualButton.MouseButton1Click:Connect(function()
        config.showVisual = not config.showVisual
        visualButton.Text = "👁️ " .. (config.showVisual and "ON" or "OFF")
        visualButton.BackgroundColor3 = config.showVisual and Color3.fromRGB(34, 139, 34) or Color3.fromRGB(120, 120, 120)
        updateVisualHitboxes()
    end)
    
    autoButton.MouseButton1Click:Connect(function()
        config.autoDetectEnabled = not config.autoDetectEnabled
        autoButton.Text = "🔄 " .. (config.autoDetectEnabled and "AUTO" or "MAN")
        autoButton.BackgroundColor3 = config.autoDetectEnabled and Color3.fromRGB(34, 139, 34) or Color3.fromRGB(120, 120, 120)
    end)
    
    toggleButton.MouseButton1Click:Connect(function()
        config.expanderEnabled = not config.expanderEnabled
        
        if config.expanderEnabled then
            toggleButton.Text = "🛑 DESATIVAR EXPANDER"
            toggleButton.BackgroundColor3 = Color3.fromRGB(34, 139, 34)
            startExpander()
        else
            toggleButton.Text = "🚀 ATIVAR EXPANDER"
            toggleButton.BackgroundColor3 = Color3.fromRGB(220, 20, 60)
            stopExpander()
        end
    end)
    
    -- Atualizar status
    spawn(function()
        while gui and gui.Parent do
            if config.expanderEnabled then
                local count = 0
                for _ in pairs(expandedParts) do
                    count = count + 1
                end
                local autoStatus = config.autoDetectEnabled and " | Auto: ON" or " | Auto: OFF"
                local damageStatus = " | Dano: " .. config.damageModes[config.damageMode]
                statusLabel.Text = "Status: Ativo | Alvos: " .. count .. autoStatus .. damageStatus
            else
                statusLabel.Text = "Status: Desativado | Alvos encontrados: 0"
            end
            wait(1)
        end
    end)
    
    gui = screenGui
end

-- [RESTO DO CÓDIGO ORIGINAL CONTINUA IGUAL...]
-- (Todas as outras funções permanecem exatamente iguais)

-- Função para calcular distância
local function getDistance(pos1, pos2)
    return (pos1 - pos2).Magnitude
end

-- Função para verificar se está próximo do player
local function isNearPlayer(character)
    if not config.autoDetectEnabled then return true end
    
    local playerCharacter = player.Character
    if not playerCharacter or not playerCharacter:FindFirstChild("HumanoidRootPart") then
        return true
    end
    
    local targetRoot = character:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return false end
    
    local distance = getDistance(playerCharacter.HumanoidRootPart.Position, targetRoot.Position)
    return distance <= config.autoDetectRange
end

-- Funções de detecção
local function isValidTarget(character)
    if not character or not character:IsA("Model") then return false end
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    return humanoid and rootPart and humanoid.Health > 0 and isNearPlayer(character)
end

local function isBot(character)
    if not isValidTarget(character) then return false end
    local player = Players:GetPlayerFromCharacter(character)
    return player == nil
end

local function isNPC(character)
    if not isValidTarget(character) then return false end
    local player = Players:GetPlayerFromCharacter(character)
    if player then return false end
    
    return character.Parent.Name:lower():find("npc") or 
           character.Name:lower():find("npc") or
           character.Name:lower():find("bot") or
           character.Parent ~= workspace
end

local function isPlayer(character)
    if not isValidTarget(character) then return false end
    local targetPlayer = Players:GetPlayerFromCharacter(character)
    return targetPlayer and targetPlayer ~= player
end

local function shouldExpand(character)
    local mode = config.modes[config.currentMode]
    
    if mode == "Bots" then
        return isBot(character)
    elseif mode == "NPCs" then
        return isNPC(character)
    elseif mode == "Players" then
        return isPlayer(character)
    end
    
    return false
end

-- FUNÇÃO PRINCIPAL - CORRIGIDA PARA DANO
local function expandHitbox(character)
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Remove hitbox antiga se existir
    local oldHitbox = character:FindFirstChild("ExpandedHitbox")
    if oldHitbox then oldHitbox:Destroy() end
    
    local oldVisual = character:FindFirstChild("ExpandedHitboxVisual")
    if oldVisual then oldVisual:Destroy() end
    
    -- Salva o tamanho original se ainda não foi salvo
    if not originalSizes[character] then
        originalSizes[character] = rootPart.Size
    end
    
    if config.damageMode == 1 then
        -- MODO 1: Hitbox Separada (Sem afetar física)
        local hitboxPart = Instance.new("Part")
        hitboxPart.Name = "ExpandedHitbox"
        hitboxPart.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
        hitboxPart.CFrame = rootPart.CFrame
        hitboxPart.Transparency = 1
        hitboxPart.CanCollide = false
        hitboxPart.Anchored = false
        hitboxPart.BrickColor = BrickColor.new("Really red")
        hitboxPart.Material = Enum.Material.ForceField
        hitboxPart.Shape = Enum.PartType.Ball
        hitboxPart.Parent = character
        
        -- CRUCIAL: Adicionar tags/propriedades para detecção de dano
        hitboxPart:SetAttribute("IsExpandedHitbox", true)
        hitboxPart:SetAttribute("OriginalCharacter", character.Name)

-- Weld para manter alinhado com o RootPart
       local weld = Instance.new("WeldConstraint")
       weld.Part0 = rootPart
       weld.Part1 = hitboxPart
       weld.Parent = hitboxPart
       
       expandedParts[character] = hitboxPart
       
   else
       -- MODO 2: Hitbox Mesclada (Modifica o RootPart diretamente)
       rootPart.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
       expandedParts[character] = rootPart
   end
   
   -- Visual da hitbox (se ativado)
   if config.showVisual then
       local visualPart = Instance.new("Part")
       visualPart.Name = "ExpandedHitboxVisual"
       visualPart.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
       visualPart.CFrame = rootPart.CFrame
       visualPart.Transparency = config.transparency
       visualPart.CanCollide = false
       visualPart.Anchored = false
       visualPart.BrickColor = BrickColor.new("Really red")
       visualPart.Material = Enum.Material.Neon
       visualPart.Shape = Enum.PartType.Ball
       visualPart.Parent = character
       
       local visualWeld = Instance.new("WeldConstraint")
       visualWeld.Part0 = rootPart
       visualWeld.Part1 = visualPart
       visualWeld.Parent = visualPart
   end
end

-- Função para restaurar hitbox
local function restoreHitbox(character)
   local expandedPart = expandedParts[character]
   if expandedPart then
       if config.damageMode == 1 then
           -- Remove a parte separada
           expandedPart:Destroy()
       else
           -- Restaura o tamanho original do RootPart
           local originalSize = originalSizes[character]
           if originalSize then
               expandedPart.Size = originalSize
           end
       end
       expandedParts[character] = nil
   end
   
   -- Remove visual
   local visualPart = character:FindFirstChild("ExpandedHitboxVisual")
   if visualPart then visualPart:Destroy() end
end

-- Função para atualizar todas as hitboxes
function updateAllHitboxes()
   for character, _ in pairs(expandedParts) do
       if character.Parent then
           expandHitbox(character)
       else
           expandedParts[character] = nil
       end
   end
end

-- Função para atualizar visuais
function updateVisualHitboxes()
   for character, _ in pairs(expandedParts) do
       if character.Parent then
           local visual = character:FindFirstChild("ExpandedHitboxVisual")
           if config.showVisual and not visual then
               local rootPart = character:FindFirstChild("HumanoidRootPart")
               if rootPart then
                   local visualPart = Instance.new("Part")
                   visualPart.Name = "ExpandedHitboxVisual"
                   visualPart.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
                   visualPart.CFrame = rootPart.CFrame
                   visualPart.Transparency = config.transparency
                   visualPart.CanCollide = false
                   visualPart.Anchored = false
                   visualPart.BrickColor = BrickColor.new("Really red")
                   visualPart.Material = Enum.Material.Neon
                   visualPart.Shape = Enum.PartType.Ball
                   visualPart.Parent = character
                   
                   local visualWeld = Instance.new("WeldConstraint")
                   visualWeld.Part0 = rootPart
                   visualWeld.Part1 = visualPart
                   visualWeld.Parent = visualPart
               end
           elseif not config.showVisual and visual then
               visual:Destroy()
           end
       end
   end
end

-- Função para iniciar o expander
function startExpander()
   -- Limpa conexões antigas
   for _, connection in pairs(connections) do
       connection:Disconnect()
   end
   connections = {}
   
   -- Função para processar todos os modelos
   local function processAllModels()
       for _, child in pairs(workspace:GetDescendants()) do
           if child:IsA("Model") and shouldExpand(child) then
               if not expandedParts[child] then
                   expandHitbox(child)
               end
           end
       end
   end
   
   -- Processa modelos existentes
   processAllModels()
   
   -- Monitor para novos modelos
   connections[#connections + 1] = workspace.DescendantAdded:Connect(function(descendant)
       if descendant:IsA("Model") and shouldExpand(descendant) then
           wait(0.1) -- Pequeno delay para garantir que o modelo está completo
           if descendant.Parent and shouldExpand(descendant) then
               expandHitbox(descendant)
           end
       end
   end)
   
   -- Monitor para modelos removidos
   connections[#connections + 1] = workspace.DescendantRemoving:Connect(function(descendant)
       if expandedParts[descendant] then
           expandedParts[descendant] = nil
           originalSizes[descendant] = nil
       end
   end)
   
   -- Loop de atualização automática
   connections[#connections + 1] = RunService.Heartbeat:Connect(function()
       -- Verifica se alvos ainda são válidos
       for character, _ in pairs(expandedParts) do
           if not character.Parent or not shouldExpand(character) then
               restoreHitbox(character)
               originalSizes[character] = nil
           end
       end
       
       -- Adiciona novos alvos se auto-detect estiver ativo
       if config.autoDetectEnabled then
           for _, child in pairs(workspace:GetChildren()) do
               if child:IsA("Model") and shouldExpand(child) and not expandedParts[child] then
                   expandHitbox(child)
               end
           end
       end
   end)
end

-- Função para parar o expander
function stopExpander()
   -- Desconecta todos os eventos
   for _, connection in pairs(connections) do
       connection:Disconnect()
   end
   connections = {}
   
   -- Restaura todas as hitboxes
   for character, _ in pairs(expandedParts) do
       restoreHitbox(character)
   end
   
   -- Limpa tabelas
   expandedParts = {}
   originalSizes = {}
end

-- Controles de teclado
UserInputService.InputBegan:Connect(function(input, gameProcessed)
   if gameProcessed then return end
   
   if input.KeyCode == Enum.KeyCode.H then
       config.expanderEnabled = not config.expanderEnabled
       
       if config.expanderEnabled then
           startExpander()
           print("🎯 Hitbox Expander: ATIVADO")
       else
           stopExpander()
           print("🎯 Hitbox Expander: DESATIVADO")
       end
   elseif input.KeyCode == Enum.KeyCode.G then
       config.currentMode = config.currentMode + 1
       if config.currentMode > #config.modes then
           config.currentMode = 1
       end
       print("🎮 Modo alterado para: " .. config.modes[config.currentMode])
       
       if config.expanderEnabled then
           stopExpander()
           startExpander()
       end
   elseif input.KeyCode == Enum.KeyCode.V then
       config.showVisual = not config.showVisual
       print("👁️ Visual: " .. (config.showVisual and "ATIVADO" or "DESATIVADO"))
       updateVisualHitboxes()
   end
end)

-- Cleanup quando o player sai
Players.PlayerRemoving:Connect(function(removingPlayer)
   if removingPlayer == player then
       stopExpander()
   end
end)

-- Criar GUI
createGUI()

print("🔓 Hitbox Expander v2.3 - VERSÃO LICENCIADA CARREGADA!")
print("🎮 Controles:")
print("   H = Ligar/Desligar")
print("   G = Trocar Modo")
print("   V = Visual ON/OFF")
print("🔐 Chave verificada com sucesso!")

end

-- Inicia a verificação de chave
verificarChave()
