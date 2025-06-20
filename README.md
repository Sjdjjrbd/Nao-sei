-- Expander Hitbox NPC Local (Funcional Corrigido)
-- Script para expandir hitboxes funcionais de diferentes tipos de entidades
-- Criado por: [Seu Nome Aqui]
-- Versão: 2.1 - Melhorada com detecção automática e sem colisão

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Configurações
local config = {
    hitboxSize = 20, -- Tamanho da hitbox expandida
    transparency = 0.5,
    expanderEnabled = false,
    currentMode = 1, -- 1 = Bots, 2 = NPCs, 3 = Players
    modes = {"Bots", "NPCs", "Players"},
    showVisual = true, -- Mostrar hitbox visual
    isMinimized = false, -- Estado do minimizador
    autoDetectRange = 50, -- Distância para detecção automática de novos players
    autoDetectEnabled = true -- Detecção automática ativada
}

-- Variáveis para controle
local expandedParts = {}
local originalSizes = {}
local connections = {}
local gui = nil

-- Função para criar a interface
local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "HitboxExpanderGUI"
    screenGui.ResetOnSpawn = false -- Persiste após morte
    screenGui.Parent = playerGui
    
    -- Frame principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 320, 0, 240)
    mainFrame.Position = UDim2.new(0, 20, 0, 20)
    mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true -- Permite arrastar
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 12)
    corner.Parent = mainFrame
    
    -- Barra de título
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 35)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 12)
    titleCorner.Parent = titleBar
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0.8, 0, 1, 0)
    title.Position = UDim2.new(0, 10, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "🎯 Hitbox Expander v2.1"
    title.TextColor3 = Color3.fromRGB(255, 255, 255)
    title.TextSize = 14
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = titleBar
    
    -- Botão minimizar
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
    
    -- Frame de conteúdo
    local contentFrame = Instance.new("Frame")
    contentFrame.Name = "ContentFrame"
    contentFrame.Size = UDim2.new(1, 0, 1, -35)
    contentFrame.Position = UDim2.new(0, 0, 0, 35)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = mainFrame
    
    -- Créditos
    local creditsLabel = Instance.new("TextLabel")
    creditsLabel.Size = UDim2.new(1, 0, 0, 20)
    creditsLabel.Position = UDim2.new(0, 0, 0, 5)
    creditsLabel.BackgroundTransparency = 1
    creditsLabel.Text = "👨‍💻 Criado por: [Seu Nome Aqui]"
    creditsLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
    creditsLabel.TextSize = 10
    creditsLabel.Font = Enum.Font.Gotham
    creditsLabel.Parent = contentFrame
    
    -- Status
    local statusLabel = Instance.new("TextLabel")
    statusLabel.Size = UDim2.new(1, 0, 0, 20)
    statusLabel.Position = UDim2.new(0, 0, 0, 25)
    statusLabel.BackgroundTransparency = 1
    statusLabel.Text = "Status: Desativado | Alvos encontrados: 0"
    statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    statusLabel.TextSize = 11
    statusLabel.Font = Enum.Font.Gotham
    statusLabel.Parent = contentFrame
    
    -- Botão de modo
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
    
    -- Input para tamanho
    local sizeFrame = Instance.new("Frame")
    sizeFrame.Size = UDim2.new(0.9, 0, 0, 25)
    sizeFrame.Position = UDim2.new(0.05, 0, 0, 80)
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
    
    -- Botões em linha (Visual e Auto-Detect)
    local buttonFrame = Instance.new("Frame")
    buttonFrame.Size = UDim2.new(0.9, 0, 0, 25)
    buttonFrame.Position = UDim2.new(0.05, 0, 0, 110)
    buttonFrame.BackgroundTransparency = 1
    buttonFrame.Parent = contentFrame
    
    -- Botão visual
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
    
    -- Botão auto-detect
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
    
    -- Range de detecção
    local rangeFrame = Instance.new("Frame")
    rangeFrame.Size = UDim2.new(0.9, 0, 0, 25)
    rangeFrame.Position = UDim2.new(0.05, 0, 0, 140)
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
    
    -- Botão principal
    local toggleButton = Instance.new("TextButton")
    toggleButton.Size = UDim2.new(0.9, 0, 0, 30)
    toggleButton.Position = UDim2.new(0.05, 0, 0, 170)
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
            -- Minimizar
            contentFrame.Visible = false
            mainFrame:TweenSize(UDim2.new(0, 320, 0, 35), "Out", "Quad", 0.3, true)
            minimizeButton.Text = "+"
        else
            -- Maximizar
            mainFrame:TweenSize(UDim2.new(0, 320, 0, 240), "Out", "Quad", 0.3, true)
            wait(0.3)
            contentFrame.Visible = true
            minimizeButton.Text = "−"
        end
    end
    
    -- Eventos
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
                local autoStatus = config.autoDetectEnabled and " | Auto-Detect: ON" or " | Auto-Detect: OFF"
                statusLabel.Text = "Status: Ativo | Alvos expandidos: " .. count .. autoStatus
            else
                statusLabel.Text = "Status: Desativado | Alvos encontrados: 0"
            end
            wait(1)
        end
    end)
    
    gui = screenGui
end

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

-- Funções de detecção melhoradas
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
    
    -- Verifica se está em pasta de NPCs ou tem características de NPC
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

-- Função para expandir hitbox (método direto SEM COLISÃO)
local function expandHitbox(character)
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Salva tamanho original
    if not originalSizes[character] then
        originalSizes[character] = rootPart.Size
    end
    
    -- Remove hitbox antiga se existir
    local oldVisual = character:FindFirstChild("ExpandedHitboxVisual")
    if oldVisual then oldVisual:Destroy() end
    
    -- Expande a hitbox real (HumanoidRootPart)
    rootPart.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
    rootPart.Transparency = 0.9 -- Quase invisível mas funcional
    
    -- ===== REMOVE COLISÃO COMPLETAMENTE =====
    rootPart.CanCollide = false
    
    -- Cria visual se habilitado
    if config.showVisual then
        local visualHitbox = Instance.new("Part")
        visualHitbox.Name = "ExpandedHitboxVisual"
        visualHitbox.Size = Vector3.new(config.hitboxSize, config.hitboxSize, config.hitboxSize)
        visualHitbox.CFrame = rootPart.CFrame
        visualHitbox.Transparency = config.transparency
        visualHitbox.BrickColor = BrickColor.new("Really red")
        visualHitbox.Material = Enum.Material.ForceField
        visualHitbox.CanCollide = false -- SEM COLISÃO NO VISUAL TAMBÉM
        visualHitbox.Anchored = false
        visualHitbox.Shape = Enum.PartType.Ball
        visualHitbox.Parent = character
        
        -- Weld visual à rootPart
        local weld = Instance.new("WeldConstraint")
        weld.Part0 = rootPart
        weld.Part1 = visualHitbox
        weld.Parent = visualHitbox
        
        expandedParts[character] = {visual = visualHitbox, rootPart = rootPart}
    else
        expandedParts[character] = {visual = nil, rootPart = rootPart}
    end
    
    print("✅ Expandido: " .. character.Name .. " | Tamanho: " .. config.hitboxSize .. " | SEM COLISÃO")
end

-- Função para remover expansão
local function removeHitbox(character)
    if expandedParts[character] then
        -- Remove visual se existir
        if expandedParts[character].visual then
            expandedParts[character].visual:Destroy()
        end
        
        -- Restaura tamanho original E COLISÃO
        if originalSizes[character] and expandedParts[character].rootPart then
            expandedParts[character].rootPart.Size = originalSizes[character]
            expandedParts[character].rootPart.Transparency = 0
            expandedParts[character].rootPart.CanCollide = true -- Restaura colisão
        end
        
        expandedParts[character] = nil
        originalSizes[character] = nil
        print("❌ Removido: " .. character.Name)
    end
end

-- Função para atualizar todas as hitboxes
function updateAllHitboxes()
    for character, data in pairs(expandedParts) do
        if character.Parent and shouldExpand(character) then
            expandHitbox(character)
        end
    end
end

-- Função para atualizar apenas visuais
function updateVisualHitboxes()
    for character, data in pairs(expandedParts) do
        if data.visual then
            data.visual.Transparency = config.showVisual and config.transparency or 1
        end
    end
end

-- Função para processar novos players próximos
local function processNearbyTargets()
    if not config.expanderEnabled or not config.autoDetectEnabled then return end
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and shouldExpand(obj) and not expandedParts[obj] then
            expandHitbox(obj)
        end
    end
end

-- Função para iniciar o expander
function startExpander()
    stopExpander()
    print("🚀 Iniciando Hitbox Expander - Modo: " .. config.modes[config.currentMode])
    
    -- Processa todos os modelos existentes
    local function processWorkspace()
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("Model") and shouldExpand(obj) then
                expandHitbox(obj)
            end
        end
    end
    
    processWorkspace()
    
    -- Monitor para novos personagens
    connections.descendantAdded = workspace.DescendantAdded:Connect(function(obj)
        if obj:IsA("Model") and shouldExpand(obj) then
            wait(0.5) -- Aguarda carregar
            if obj.Parent and shouldExpand(obj) then
                expandHitbox(obj)
            end
        end
    end)
    
    -- Monitor para personagens removidos
    connections.descendantRemoving = workspace.DescendantRemoving:Connect(function(obj)
        if expandedParts[obj] then
            removeHitbox(obj)
        end
    end)
    
    -- Loop de manutenção e detecção automática
    connections.heartbeat = RunService.Heartbeat:Connect(function()
        if not config.expanderEnabled then return end
        
        -- Remove alvos que não devem mais estar expandidos
        for character, data in pairs(expandedParts) do
            if not character.Parent or not shouldExpand(character) then
                removeHitbox(character)
            end
        end
        
        -- Processa novos alvos próximos (a cada 60 frames para performance)
        if tick() % 1 < 0.016 then -- Aproximadamente 1 segundo
            processNearbyTargets()
        end
    end)
    
    -- Monitor específico para novos players
    connections.playerAdded = Players.PlayerAdded:Connect(function(newPlayer)
        newPlayer.CharacterAdded:Connect(function(character)
            wait(1) -- Aguarda o personagem carregar completamente
            if config.expanderEnabled and shouldExpand(character) then
                expandHitbox(character)
                print("🎯 Novo player detectado e expandido: " .. newPlayer.Name)
            end
        end)
    end)
    
    -- Processa players já existentes
    for _, existingPlayer in pairs(Players:GetPlayers()) do
        if existingPlayer ~= player and existingPlayer.Character then
            existingPlayer.CharacterAdded:Connect(function(character)
                wait(1)
                if config.expanderEnabled and shouldExpand(character) then
                    expandHitbox(character)
                    print("🎯 Player existente detectado: " .. existingPlayer.Name)
                end
            end)
        end
    end
end

-- Função para parar o expander
function stopExpander()
    print("🛑 Parando Hitbox Expander...")
    
    -- Remove todas as expansões
    for character, data in pairs(expandedParts) do
        removeHitbox(character)
    end
    
    expandedParts = {}
    originalSizes = {}
    
    -- Desconecta eventos
    for _, connection in pairs(connections) do
        if connection then
            connection:Disconnect()
        end
    end
    connections = {}
end

-- Limpeza
local function cleanup()
    stopExpander()
    if gui then
        gui:Destroy()
    end
end

-- Inicialização
createGUI()

-- Sistema de persistência após morte
local function onCharacterAdded(character)
    local humanoid = character:WaitForChild("Humanoid")
    
    -- Reconecta o player quando renascer
    humanoid.Died:Connect(function()
        print("🔄 Player morreu, mantendo script ativo...")
        -- O script continua funcionando pois a GUI tem ResetOnSpawn = false
    end)
    
    -- Atualiza referência do player
    player = Players.LocalPlayer
    
    print("🎯 Player renasceu, Hitbox Expander ainda ativo!")
    
    -- Reprocessa alvos próximos após renascer
    if config.expanderEnabled then
        wait(2)
        processNearbyTargets()
    end
end

-- Conecta para o personagem atual e futuros
if player.Character then
    onCharacterAdded(player.Character)
end
player.CharacterAdded:Connect(onCharacterAdded)

-- Limpeza apenas quando o jogador realmente sair do jogo
Players.PlayerRemoving:Connect(function(plr)
    if plr == player then
        cleanup()
    end
end)

print("🎯 Hitbox Expander v2.1 carregado!")
print("✅ Criado por: [Seu Nome Aqui]")
print("✅ Persistente após morte")
print("✅ Interface minimizável e arrastável")
print("✅ SEM COLISÃO nas hitboxes expandidas")
print("✅ Detecção automática de novos players próximos")
print("✅ Range de detecção configurável")
print("💡 Dica: Use 'Auto-Detect' para detectar automaticamente novos alvos próximos!")
print("💡 Configure o 'Range' para definir a distância de detecção!")
