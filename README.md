-- DeX Explorer Complete Edition
-- Versão 2.0 - Interface Moderna e Funcionalidades Completas

local player = game:GetService("Players").LocalPlayer
local mouse = player:GetMouse()
local userInput = game:GetService("UserInputService")
local tweenService = game:GetService("TweenService")
local runService = game:GetService("RunService")
local textService = game:GetService("TextService")

-- Cache e variáveis globais
local nodes = {}
local selectedNodes = {}
local expandedNodes = {}
local propertyCache = {}

-- Cores modernas (tema escuro refinado)
local colors = {
    background = Color3.fromRGB(30, 30, 30),
    backgroundLight = Color3.fromRGB(45, 45, 45),
    backgroundDark = Color3.fromRGB(25, 25, 25),
    accent = Color3.fromRGB(0, 160, 255),
    accentLight = Color3.fromRGB(100, 200, 255),
    text = Color3.fromRGB(240, 240, 240),
    textDim = Color3.fromRGB(160, 160, 160),
    border = Color3.fromRGB(55, 55, 55),
    selection = Color3.fromRGB(0, 120, 215),
    hover = Color3.fromRGB(60, 60, 60),
    button = Color3.fromRGB(50, 50, 50),
    buttonHover = Color3.fromRGB(70, 70, 70),
    success = Color3.fromRGB(0, 200, 100),
    warning = Color3.fromRGB(255, 170, 0),
    error = Color3.fromRGB(255, 80, 80)
}

-- Ícones modernos (usando Material Design Icons via asset IDs)
local icons = {
    close = "rbxassetid://5054663650",
    minimize = "rbxassetid://5034768003",
    maximize = "rbxassetid://5060023708",
    search = "rbxassetid://5034718129",
    refresh = "rbxassetid://5642310344",
    folder = "rbxassetid://6578933307",
    script = "rbxassetid://6579106223",
    model = "rbxassetid://6578933307",
    part = "rbxassetid://6578871732",
    player = "rbxassetid://6578933307",
    settings = "rbxassetid://6578871732",
    expand = "rbxassetid://5642383285",
    collapse = "rbxassetid://5642383285",
    property = "rbxassetid://6578933307",
    delete = "rbxassetid://6578871732",
    copy = "rbxassetid://6579106223",
    paste = "rbxassetid://6578933307",
    play = "rbxassetid://6578871732",
    stop = "rbxassetid://6579106223"
}

-- Função para proteger GUI
local function protectGui(gui)
    if syn and syn.protect_gui then
        syn.protect_gui(gui)
        gui.Parent = game:GetService("CoreGui")
    elseif gethui then
        gui.Parent = gethui()
    else
        gui.Parent = player:WaitForChild("PlayerGui")
    end
end

-- Classe Signal (eventos personalizados)
local Signal = {}
Signal.__index = Signal

function Signal.new()
    local self = setmetatable({}, Signal)
    self._connections = {}
    return self
end

function Signal:Connect(callback)
    local connection = { callback = callback, disconnected = false }
    table.insert(self._connections, connection)
    return {
        Disconnect = function()
            connection.disconnected = true
        end
    }
end

function Signal:Fire(...)
    for _, connection in ipairs(self._connections) do
        if not connection.disconnected then
            task.spawn(connection.callback, ...)
        end
    end
end

-- Classe Window (janela estilizada)
local Window = {}
Window.__index = Window

function Window.new(title, size)
    local self = setmetatable({}, Window)
    
    self.position = UDim2.new(0.5, -size.X.Offset/2, 0.5, -size.Y.Offset/2)
    self.size = size
    self.minSize = UDim2.new(0, 300, 0, 200)
    self.dragging = false
    self.resizing = false
    self.minimized = false
    self.originalSize = size
    self.originalPos = self.position
    self.elements = {}
    self.signals = {
        onClose = Signal.new(),
        onResize = Signal.new(),
        onMinimize = Signal.new()
    }
    
    self:_build(title)
    return self
end

function Window:_build(title)
    -- ScreenGui
    self.gui = Instance.new("ScreenGui")
    self.gui.Name = "DexWindow_" .. title
    self.gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    protectGui(self.gui)
    
    -- Main frame
    self.main = Instance.new("Frame")
    self.main.Name = "Main"
    self.main.Size = self.size
    self.main.Position = self.position
    self.main.BackgroundColor3 = colors.background
    self.main.BorderSizePixel = 0
    self.main.ClipsDescendants = true
    self.main.Parent = self.gui
    
    -- Sombra/Outline
    local shadow = Instance.new("ImageLabel")
    shadow.Name = "Shadow"
    shadow.Image = "rbxassetid://1427967925"
    shadow.ScaleType = Enum.ScaleType.Slice
    shadow.SliceCenter = Rect.new(6, 6, 25, 25)
    shadow.Size = UDim2.new(1, 10, 1, 10)
    shadow.Position = UDim2.new(0, -5, 0, -5)
    shadow.BackgroundTransparency = 1
    shadow.ImageColor3 = Color3.fromRGB(20, 20, 20)
    shadow.Parent = self.main
    
    -- Barra de título
    self.titleBar = Instance.new("Frame")
    self.titleBar.Name = "TitleBar"
    self.titleBar.Size = UDim2.new(1, 0, 0, 30)
    self.titleBar.BackgroundColor3 = colors.backgroundDark
    self.titleBar.BorderSizePixel = 0
    self.titleBar.Parent = self.main
    
    -- Linha inferior da barra
    local titleLine = Instance.new("Frame")
    titleLine.Size = UDim2.new(1, 0, 0, 1)
    titleLine.Position = UDim2.new(0, 0, 1, -1)
    titleLine.BackgroundColor3 = colors.accent
    titleLine.BorderSizePixel = 0
    titleLine.Parent = self.titleBar
    
    -- Título
    self.titleLabel = Instance.new("TextLabel")
    self.titleLabel.Size = UDim2.new(1, -90, 1, 0)
    self.titleLabel.Position = UDim2.new(0, 10, 0, 0)
    self.titleLabel.BackgroundTransparency = 1
    self.titleLabel.Font = Enum.Font.GothamSemibold
    self.titleLabel.TextSize = 14
    self.titleLabel.TextColor3 = colors.text
    self.titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    self.titleLabel.Text = title
    self.titleLabel.Parent = self.titleBar
    
    -- Botões da janela
    self:_createWindowButtons()
    
    -- Área de conteúdo
    self.content = Instance.new("Frame")
    self.content.Name = "Content"
    self.content.Size = UDim2.new(1, 0, 1, -30)
    self.content.Position = UDim2.new(0, 0, 0, 30)
    self.content.BackgroundColor3 = colors.backgroundLight
    self.content.BorderSizePixel = 0
    self.content.Parent = self.main
    
    -- Handles de redimensionamento
    self:_createResizeHandles()
    
    -- Eventos de arrasto
    self:_setupDragging()
end

function Window:_createWindowButtons()
    local buttonSize = 20
    local buttonY = 5
    
    -- Fechar
    self.closeBtn = self:_createTitleButton(
        UDim2.new(1, -25, 0, buttonY),
        buttonSize,
        icons.close,
        colors.error
    )
    self.closeBtn.MouseButton1Click:Connect(function()
        self.signals.onClose:Fire()
        self:Hide()
    end)
    
    -- Maximizar/Restaurar
    self.maximizeBtn = self:_createTitleButton(
        UDim2.new(1, -50, 0, buttonY),
        buttonSize,
        icons.maximize,
        colors.textDim
    )
    self.maximizeBtn.MouseButton1Click:Connect(function()
        self:ToggleMaximize()
    end)
    
    -- Minimizar
    self.minimizeBtn = self:_createTitleButton(
        UDim2.new(1, -75, 0, buttonY),
        buttonSize,
        icons.minimize,
        colors.textDim
    )
    self.minimizeBtn.MouseButton1Click:Connect(function()
        self:ToggleMinimize()
    end)
end

function Window:_createTitleButton(position, size, icon, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, size, 0, size)
    btn.Position = position
    btn.BackgroundColor3 = colors.backgroundDark
    btn.BackgroundTransparency = 1
    btn.AutoButtonColor = false
    btn.Text = ""
    btn.Parent = self.titleBar
    
    local iconImg = Instance.new("ImageLabel")
    iconImg.Size = UDim2.new(0, size-4, 0, size-4)
    iconImg.Position = UDim2.new(0, 2, 0, 2)
    iconImg.BackgroundTransparency = 1
    iconImg.Image = icon
    iconImg.ImageColor3 = color
    iconImg.Parent = btn
    
    -- Hover effect
    btn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            tweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 0.7}):Play()
        end
    end)
    
    btn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            tweenService:Create(btn, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
        end
    end)
    
    return btn
end

function Window:_createResizeHandles()
    local handles = {
        {name = "N", pos = UDim2.new(0, 5, 0, 0), size = UDim2.new(1, -10, 0, 5)},
        {name = "S", pos = UDim2.new(0, 5, 1, -5), size = UDim2.new(1, -10, 0, 5)},
        {name = "E", pos = UDim2.new(1, -5, 0, 5), size = UDim2.new(0, 5, 1, -10)},
        {name = "W", pos = UDim2.new(0, 0, 0, 5), size = UDim2.new(0, 5, 1, -10)},
        {name = "NE", pos = UDim2.new(1, -5, 0, 0), size = UDim2.new(0, 5, 0, 5)},
        {name = "NW", pos = UDim2.new(0, 0, 0, 0), size = UDim2.new(0, 5, 0, 5)},
        {name = "SE", pos = UDim2.new(1, -5, 1, -5), size = UDim2.new(0, 5, 0, 5)},
        {name = "SW", pos = UDim2.new(0, 0, 1, -5), size = UDim2.new(0, 5, 0, 5)}
    }
    
    for _, data in ipairs(handles) do
        local handle = Instance.new("TextButton")
        handle.Name = "Resize_" .. data.name
        handle.Size = data.size
        handle.Position = data.pos
        handle.BackgroundTransparency = 1
        handle.AutoButtonColor = false
        handle.Text = ""
        handle.Parent = self.main
        
        self:_setupResizeHandle(handle, data.name)
    end
end

function Window:_setupResizeHandle(handle, direction)
    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            self.resizing = true
            self.resizeDir = direction
            self.resizeStart = input.Position
            self.resizeStartPos = self.main.Position
            self.resizeStartSize = self.main.Size
        end
    end)
end

function Window:_setupDragging()
    self.titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            self.dragging = true
            self.dragStart = input.Position
            self.dragStartPos = self.main.Position
        end
    end)
    
    userInput.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            if self.dragging then
                local delta = input.Position - self.dragStart
                self.main.Position = UDim2.new(
                    0, self.dragStartPos.X.Offset + delta.X,
                    0, self.dragStartPos.Y.Offset + delta.Y
                )
            elseif self.resizing then
                self:_handleResize(input.Position - self.resizeStart)
            end
        end
    end)
    
    userInput.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            self.dragging = false
            self.resizing = false
        end
    end)
end

function Window:_handleResize(delta)
    local newSize = self.resizeStartSize
    local newPos = self.resizeStartPos
    local dir = self.resizeDir
    
    if dir:find("E") then
        newSize = UDim2.new(0, self.resizeStartSize.X.Offset + delta.X, 0, newSize.Y.Offset)
    end
    if dir:find("W") then
        newSize = UDim2.new(0, self.resizeStartSize.X.Offset - delta.X, 0, newSize.Y.Offset)
        newPos = UDim2.new(0, self.resizeStartPos.X.Offset + delta.X, 0, newPos.Y.Offset)
    end
    if dir:find("S") then
        newSize = UDim2.new(0, newSize.X.Offset, 0, self.resizeStartSize.Y.Offset + delta.Y)
    end
    if dir:find("N") then
        newSize = UDim2.new(0, newSize.X.Offset, 0, self.resizeStartSize.Y.Offset - delta.Y)
        newPos = UDim2.new(0, newPos.X.Offset, 0, self.resizeStartPos.Y.Offset + delta.Y)
    end
    
    -- Aplicar tamanho mínimo
    if newSize.X.Offset < self.minSize.X.Offset then
        newSize = UDim2.new(0, self.minSize.X.Offset, 0, newSize.Y.Offset)
    end
    if newSize.Y.Offset < self.minSize.Y.Offset then
        newSize = UDim2.new(0, newSize.X.Offset, 0, self.minSize.Y.Offset)
    end
    
    self.main.Size = newSize
    self.main.Position = newPos
    self.signals.onResize:Fire(newSize)
end

function Window:ToggleMinimize()
    self.minimized = not self.minimized
    if self.minimized then
        self.originalSize = self.main.Size
        self.originalPos = self.main.Position
        tweenService:Create(self.main, TweenInfo.new(0.2), {
            Size = UDim2.new(0, self.originalSize.X.Offset, 0, 30)
        }):Play()
    else
        tweenService:Create(self.main, TweenInfo.new(0.2), {
            Size = self.originalSize,
            Position = self.originalPos
        }):Play()
    end
    self.signals.onMinimize:Fire(self.minimized)
end

function Window:ToggleMaximize()
    -- Implementar maximizar se desejar
end

function Window:Show()
    self.gui.Enabled = true
end

function Window:Hide()
    self.gui.Enabled = false
end

function Window:AddElement(name, element)
    self.elements[name] = element
    element.Parent = self.content
    return element
end

-- Classe TreeView (Explorer de instâncias)
local TreeView = {}
TreeView.__index = TreeView

function TreeView.new(window)
    local self = setmetatable({}, TreeView)
    self.window = window
    self.selectedItem = nil
    self.nodes = {}
    self.signals = {
        onSelect = Signal.new()
    }
    
    self:_build()
    self:_scanInstances()
    self:_setupConnections()
    return self
end

function TreeView:_build()
    -- Frame principal
    self.main = Instance.new("Frame")
    self.main.Size = UDim2.new(1, 0, 1, 0)
    self.main.BackgroundColor3 = colors.backgroundLight
    self.main.BorderSizePixel = 0
    self.main.Parent = self.window.content
    
    -- Barra de ferramentas
    self.toolbar = Instance.new("Frame")
    self.toolbar.Size = UDim2.new(1, 0, 0, 35)
    self.toolbar.BackgroundColor3 = colors.backgroundDark
    self.toolbar.BorderSizePixel = 0
    self.toolbar.Parent = self.main
    
    -- Campo de busca
    self.searchFrame = Instance.new("Frame")
    self.searchFrame.Size = UDim2.new(1, -10, 0, 25)
    self.searchFrame.Position = UDim2.new(0, 5, 0, 5)
    self.searchFrame.BackgroundColor3 = colors.background
    self.searchFrame.BorderSizePixel = 0
    self.searchFrame.Parent = self.toolbar
    
    local searchIcon = Instance.new("ImageLabel")
    searchIcon.Size = UDim2.new(0, 16, 0, 16)
    searchIcon.Position = UDim2.new(0, 5, 0.5, -8)
    searchIcon.BackgroundTransparency = 1
    searchIcon.Image = icons.search
    searchIcon.ImageColor3 = colors.textDim
    searchIcon.Parent = self.searchFrame
    
    self.searchBox = Instance.new("TextBox")
    self.searchBox.Size = UDim2.new(1, -30, 1, 0)
    self.searchBox.Position = UDim2.new(0, 25, 0, 0)
    self.searchBox.BackgroundTransparency = 1
    self.searchBox.Font = Enum.Font.SourceSans
    self.searchBox.TextSize = 14
    self.searchBox.TextColor3 = colors.text
    self.searchBox.PlaceholderColor3 = colors.textDim
    self.searchBox.PlaceholderText = "Pesquisar..."
    self.searchBox.ClearTextOnFocus = false
    self.searchBox.Text = ""
    self.searchBox.TextXAlignment = Enum.TextXAlignment.Left
    self.searchBox.Parent = self.searchFrame
    
    -- Botão refresh
    self.refreshBtn = Instance.new("TextButton")
    self.refreshBtn.Size = UDim2.new(0, 25, 0, 25)
    self.refreshBtn.Position = UDim2.new(1, -30, 0, 5)
    self.refreshBtn.BackgroundColor3 = colors.background
    self.refreshBtn.BackgroundTransparency = 1
    self.refreshBtn.AutoButtonColor = false
    self.refreshBtn.Text = ""
    self.refreshBtn.Parent = self.toolbar
    
    local refreshIcon = Instance.new("ImageLabel")
    refreshIcon.Size = UDim2.new(0, 16, 0, 16)
    refreshIcon.Position = UDim2.new(0.5, -8, 0.5, -8)
    refreshIcon.BackgroundTransparency = 1
    refreshIcon.Image = icons.refresh
    refreshIcon.ImageColor3 = colors.textDim
    refreshIcon.Parent = self.refreshBtn
    
    -- Árvore (ScrollingFrame)
    self.tree = Instance.new("ScrollingFrame")
    self.tree.Size = UDim2.new(1, 0, 1, -35)
    self.tree.Position = UDim2.new(0, 0, 0, 35)
    self.tree.BackgroundColor3 = colors.backgroundLight
    self.tree.BorderSizePixel = 0
    self.tree.CanvasSize = UDim2.new(0, 0, 0, 0)
    self.tree.AutomaticCanvasSize = Enum.AutomaticSize.Y
    self.tree.ScrollBarThickness = 8
    self.tree.ScrollBarImageColor3 = colors.border
    self.tree.Parent = self.main
    
    -- Layout da árvore
    self.treeLayout = Instance.new("UIListLayout")
    self.treeLayout.Parent = self.tree
    self.treeLayout.Padding = UDim.new(0, 1)
end

function TreeView:_scanInstances()
    -- Escaneia todas as instâncias do jogo
    local function scan(obj, parentNode, depth)
        if depth > 50 then return end -- Limite de profundidade
        
        local node = {
            obj = obj,
            parent = parentNode,
            children = {},
            expanded = false
        }
        
        self.nodes[obj] = node
        if parentNode then
            table.insert(parentNode.children, node)
        end
        
        for _, child in ipairs(obj:GetChildren()) do
            scan(child, node, depth + 1)
        end
    end
    
    scan(game, nil, 0)
    self:_buildTree()
end

function TreeView:_buildTree(filter)
    -- Limpa árvore atual
    for _, child in ipairs(self.tree:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    -- Recria árvore com filtro opcional
    local function addNode(node, level)
        if filter and not node.obj.Name:lower():find(filter:lower()) then
            return
        end
        
        local item = self:_createTreeItem(node, level)
        item.Parent = self.tree
        
        if node.expanded then
            for _, child in ipairs(node.children) do
                addNode(child, level + 1)
            end
        end
    end
    
    addNode(self.nodes[game], 0)
end

function TreeView:_createTreeItem(node, level)
    local item = Instance.new("Frame")
    item.Name = "Item_" .. node.obj.Name
    item.Size = UDim2.new(1, 0, 0, 24)
    item.BackgroundColor3 = colors.backgroundLight
    item.BackgroundTransparency = 1
    item.BorderSizePixel = 0
    
    -- Ícone de expansão
    if #node.children > 0 then
        local expandBtn = Instance.new("TextButton")
        expandBtn.Size = UDim2.new(0, 20, 0, 20)
        expandBtn.Position = UDim2.new(0, level * 15 + 2, 0, 2)
        expandBtn.BackgroundTransparency = 1
        expandBtn.AutoButtonColor = false
        expandBtn.Text = ""
        expandBtn.Parent = item
        
        local expandIcon = Instance.new("ImageLabel")
        expandIcon.Size = UDim2.new(0, 12, 0, 12)
        expandIcon.Position = UDim2.new(0, 4, 0, 4)
        expandIcon.BackgroundTransparency = 1
        expandIcon.Image = icons.expand
        expandIcon.ImageRectOffset = node.expanded and Vector2.new(16, 16) or Vector2.new(0, 16)
        expandIcon.ImageRectSize = Vector2.new(16, 16)
        expandIcon.ImageColor3 = colors.textDim
        expandIcon.Parent = expandBtn
        
        expandBtn.MouseButton1Click:Connect(function()
            node.expanded = not node.expanded
            self:_buildTree(self.searchBox.Text)
        end)
    end
    
    -- Ícone do objeto
    local icon = Instance.new("ImageLabel")
    icon.Size = UDim2.new(0, 16, 0, 16)
    icon.Position = UDim2.new(0, level * 15 + 25, 0, 4)
    icon.BackgroundTransparency = 1
    icon.Image = self:_getIconForClass(node.obj.ClassName)
    icon.ImageColor3 = colors.textDim
    icon.Parent = item
    
    -- Nome do objeto
    local name = Instance.new("TextLabel")
    name.Size = UDim2.new(1, -(level * 15 + 45), 0, 24)
    name.Position = UDim2.new(0, level * 15 + 45, 0, 0)
    name.BackgroundTransparency = 1
    name.Font = Enum.Font.SourceSans
    name.TextSize = 14
    name.TextColor3 = colors.text
    name.TextXAlignment = Enum.TextXAlignment.Left
    name.Text = node.obj.Name
    name.Parent = item
    
    -- Classe (opcional)
    local class = Instance.new("TextLabel")
    class.Size = UDim2.new(0, 100, 0, 24)
    class.Position = UDim2.new(1, -105, 0, 0)
    class.BackgroundTransparency = 1
    class.Font = Enum.Font.SourceSans
    class.TextSize = 12
    class.TextColor3 = colors.textDim
    class.TextXAlignment = Enum.TextXAlignment.Right
    class.Text = "[" .. node.obj.ClassName .. "]"
    class.Parent = item
    
    -- Botão de seleção
    local selectBtn = Instance.new("TextButton")
    selectBtn.Size = UDim2.new(1, 0, 1, 0)
    selectBtn.BackgroundTransparency = 1
    selectBtn.AutoButtonColor = false
    selectBtn.Text = ""
    selectBtn.Parent = item
    
    -- Hover effect
    selectBtn.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            item.BackgroundColor3 = colors.hover
            item.BackgroundTransparency = 0.3
        end
    end)
    
    selectBtn.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            if self.selectedItem == node then
                item.BackgroundColor3 = colors.selection
                item.BackgroundTransparency = 0
            else
                item.BackgroundColor3 = colors.backgroundLight
                item.BackgroundTransparency = 1
            end
        end
    end)
    
    selectBtn.MouseButton1Click:Connect(function()
        if self.selectedItem then
            self.selectedItem.BackgroundColor3 = colors.backgroundLight
            self.selectedItem.BackgroundTransparency = 1
        end
        
        self.selectedItem = item
        item.BackgroundColor3 = colors.selection
        item.BackgroundTransparency = 0
        self.signals.onSelect:Fire(node.obj)
    end)
    
    return item
end

function TreeView:_getIconForClass(className)
    -- Mapeia classes para ícones
    local iconMap = {
        ["Folder"] = icons.folder,
        ["Script"] = icons.script,
        ["LocalScript"] = icons.script,
        ["ModuleScript"] = icons.script,
        ["Model"] = icons.model,
        ["Part"] = icons.part,
        ["MeshPart"] = icons.part,
        ["Player"] = icons.player
    }
    
    return iconMap[className] or icons.folder
end

function TreeView:_setupConnections()
    -- Conexões para atualização em tempo real
    game.DescendantAdded:Connect(function(descendant)
        self:_scanInstances()
        self:_buildTree(self.searchBox.Text)
    end)
    
    game.DescendantRemoving:Connect(function(descendant)
        self:_scanInstances()
        self:_buildTree(self.searchBox.Text)
    end)
    
    -- Filtro de pesquisa
    local debounce
    self.searchBox.Changed:Connect(function(prop)
        if prop == "Text" then
            if debounce then debounce:Cancel() end
            debounce = task.delay(0.3, function()
                self:_buildTree(self.searchBox.Text)
            end)
        end
    end)
    
    -- Refresh button
    self.refreshBtn.MouseButton1Click:Connect(function()
        self:_scanInstances()
        self:_buildTree(self.searchBox.Text)
    end)
end

-- Classe PropertyGrid (Inspetor de propriedades)
local PropertyGrid = {}
PropertyGrid.__index = PropertyGrid

function PropertyGrid.new(window)
    local self = setmetatable({}, PropertyGrid)
    self.window = window
    self.currentObject = nil
    self.properties = {}
    
    self:_build()
    return self
end

function PropertyGrid:_build()
    self.main = Instance.new("Frame")
    self.main.Size = UDim2.new(1, 0, 1, 0)
    self.main.BackgroundColor3 = colors.backgroundLight
    self.main.BorderSizePixel = 0
    self.main.Parent = self.window.content
    
    -- Título
    self.title = Instance.new("TextLabel")
    self.title.Size = UDim2.new(1, 0, 0, 30)
    self.title.BackgroundColor3 = colors.backgroundDark
    self.title.Font = Enum.Font.GothamSemibold
    self.title.TextSize = 14
    self.title.TextColor3 = colors.text
    self.title.Text = "Propriedades"
    self.title.Parent = self.main
    
    -- Área de propriedades
    self.grid = Instance.new("ScrollingFrame")
    self.grid.Size = UDim2.new(1, 0, 1, -30)
    self.grid.Position = UDim2.new(0, 0, 0, 30)
    self.grid.BackgroundColor3 = colors.backgroundLight
    self.grid.BorderSizePixel = 0
    self.grid.CanvasSize = UDim2.new(0, 0, 0, 0)
    self.grid.AutomaticCanvasSize = Enum.AutomaticSize.Y
    self.grid.ScrollBarThickness = 8
    self.grid.ScrollBarImageColor3 = colors.border
    self.grid.Parent = self.main
    
    -- Layout
    self.gridLayout = Instance.new("UIListLayout")
    self.gridLayout.Parent = self.grid
    self.gridLayout.Padding = UDim.new(0, 1)
end

function PropertyGrid:SetObject(obj)
    self.currentObject = obj
    self:_loadProperties()
end

function PropertyGrid:_loadProperties()
    -- Limpa propriedades antigas
    for _, child in ipairs(self.grid:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    if not self.currentObject then return end
    
    -- Obtém propriedades do objeto
    local props = self:_getObjectProperties(self.currentObject)
    
    for _, prop in ipairs(props) do
        self:_createPropertyItem(prop)
    end
end

function PropertyGrid:_getObjectProperties(obj)
    -- Lista de propriedades relevantes
    local props = {}
    
    -- Propriedades básicas
    local basicProps = {
        "Name", "ClassName", "Parent", "Archivable"
    }
    
    for _, name in ipairs(basicProps) do
        local success, value = pcall(function()
            return obj[name]
        end)
        
        if success then
            table.insert(props, {
                name = name,
                value = value,
                type = typeof(value)
            })
        end
    end
    
    -- Propriedades específicas por classe
    if obj:IsA("BasePart") then
        local partProps = {
            "Position", "Size", "Color", "Material",
            "Transparency", "Reflectance", "Anchored",
            "CanCollide", "Locked"
        }
        
        for _, name in ipairs(partProps) do
            local success, value = pcall(function()
                return obj[name]
            end)
            
            if success then
                table.insert(props, {
                    name = name,
                    value = value,
                    type = typeof(value)
                })
            end
        end
    elseif obj:IsA("Humanoid") then
        local humanoidProps = {
            "Health", "MaxHealth", "WalkSpeed", "JumpPower",
            "AutoRotate", "HipHeight", "DisplayDistanceType"
        }
        
        for _, name in ipairs(humanoidProps) do
            local success, value = pcall(function()
                return obj[name]
            end)
            
            if success then
                table.insert(props, {
                    name = name,
                    value = value,
                    type = typeof(value)
                })
            end
        end
    end
    
    return props
end

function PropertyGrid:_createPropertyItem(prop)
    local item = Instance.new("Frame")
    item.Size = UDim2.new(1, 0, 0, 30)
    item.BackgroundColor3 = colors.backgroundLight
    item.BackgroundTransparency = 1
    item.BorderSizePixel = 0
    item.Parent = self.grid
    
    -- Nome da propriedade
    local name = Instance.new("TextLabel")
    name.Size = UDim2.new(0.4, -5, 1, 0)
    name.Position = UDim2.new(0, 5, 0, 0)
    name.BackgroundTransparency = 1
    name.Font = Enum.Font.SourceSans
    name.TextSize = 14
    name.TextColor3 = colors.textDim
    name.TextXAlignment = Enum.TextXAlignment.Left
    name.Text = prop.name
    name.Parent = item
    
    -- Valor
    local value = Instance.new("TextBox")
    value.Size = UDim2.new(0.6, -10, 0, 20)
    value.Position = UDim2.new(0.4, 0, 0.5, -10)
    value.BackgroundColor3 = colors.background
    value.BorderColor3 = colors.border
    value.Font = Enum.Font.SourceSans
    value.TextSize = 14
    value.TextColor3 = colors.text
    value.Text = tostring(prop.value)
    value.ClearTextOnFocus = false
    value.Parent = item
    
    value.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            self:_setProperty(prop.name, value.Text)
        end
    end)
end

function PropertyGrid:_setProperty(name, value)
    if not self.currentObject then return end
    
    local success, result = pcall(function()
        -- Tenta converter para o tipo apropriado
        local converted = self:_convertValue(value, self.currentObject[name])
        self.currentObject[name] = converted
    end)
    
    if success then
        -- Atualiza display
        self:_loadProperties()
    end
end

function PropertyGrid:_convertValue(str, original)
    local type = typeof(original)
    
    if type == "number" then
        return tonumber(str) or original
    elseif type == "boolean" then
        return str == "true" or str == "1"
    elseif type == "Vector3" then
        local x, y, z = str:match("([%d.-]+),? ([%d.-]+),? ([%d.-]+)")
        if x and y and z then
            return Vector3.new(tonumber(x), tonumber(y), tonumber(z))
        end
    elseif type == "Color3" then
        local r, g, b = str:match("([%d.-]+),? ([%d.-]+),? ([%d.-]+)")
        if r and g and b then
            return Color3.new(tonumber(r), tonumber(g), tonumber(b))
        end
    end
    
    return str
end

-- Classe Console (Janela de console)
local Console = {}
Console.__index = Console

function Console.new(window)
    local self = setmetatable({}, Console)
    self.window = window
    self.history = {}
    
    self:_build()
    self:_setupLogging()
    return self
end

function Console:_build()
    self.main = Instance.new("Frame")
    self.main.Size = UDim2.new(1, 0, 1, 0)
    self.main.BackgroundColor3 = colors.backgroundLight
    self.main.BorderSizePixel = 0
    self.main.Parent = self.window.content
    
    -- Área de saída
    self.output = Instance.new("ScrollingFrame")
    self.output.Size = UDim2.new(1, -10, 1, -50)
    self.output.Position = UDim2.new(0, 5, 0, 5)
    self.output.BackgroundColor3 = colors.background
    self.output.BorderSizePixel = 0
    self.output.CanvasSize = UDim2.new(0, 0, 0, 0)
    self.output.AutomaticCanvasSize = Enum.AutomaticSize.Y
    self.output.ScrollBarThickness = 8
    self.output.ScrollBarImageColor3 = colors.border
    self.output.Parent = self.main
    
    self.outputLayout = Instance.new("UIListLayout")
    self.outputLayout.Parent = self.output
    self.outputLayout.Padding = UDim.new(0, 2)
    
    -- Linha de comando
    self.inputFrame = Instance.new("Frame")
    self.inputFrame.Size = UDim2.new(1, -10, 0, 30)
    self.inputFrame.Position = UDim2.new(0, 5, 1, -35)
    self.inputFrame.BackgroundColor3 = colors.background
    self.inputFrame.BorderSizePixel = 0
    self.inputFrame.Parent = self.main
    
    self.input = Instance.new("TextBox")
    self.input.Size = UDim2.new(1, -10, 1, -10)
    self.input.Position = UDim2.new(0, 5, 0, 5)
    self.input.BackgroundTransparency = 1
    self.input.Font = Enum.Font.Code
    self.input.TextSize = 14
    self.input.TextColor3 = colors.text
    self.input.PlaceholderColor3 = colors.textDim
    self.input.PlaceholderText = "> Digite um comando Lua..."
    self.input.ClearTextOnFocus = false
    self.input.Text = ""
    self.input.Parent = self.inputFrame
    
    self.input.FocusLost:Connect(function(enterPressed)
        if enterPressed and self.input.Text ~= "" then
            self:Execute(self.input.Text)
            self.input.Text = ""
        end
    end)
end

function Console:_setupLogging()
    -- Captura output do jogo
    local logService = game:GetService("LogService")
    
    logService.MessageOut:Connect(function(message, messageType)
        self:Print(message, messageType)
    end)
end

function Console:Print(message, messageType)
    local colors = {
        [Enum.MessageType.MessageOutput] = Color3.fromRGB(200, 200, 200),
        [Enum.MessageType.MessageWarning] = Color3.fromRGB(255, 200, 0),
        [Enum.MessageType.MessageError] = Color3.fromRGB(255, 80, 80),
        [Enum.MessageType.MessageInfo] = Color3.fromRGB(100, 200, 255)
    }
    
    local color = colors[messageType] or Color3.fromRGB(200, 200, 200)
    
    local line = Instance.new("TextLabel")
    line.Size = UDim2.new(1, -10, 0, 20)
    line.Position = UDim2.new(0, 5, 0, 0)
    line.BackgroundTransparency = 1
    line.Font = Enum.Font.Code
    line.TextSize = 13
    line.TextColor3 = color
    line.TextXAlignment = Enum.TextXAlignment.Left
    line.Text = os.date("[%H:%M:%S] ") .. message
    line.TextTruncate = Enum.TextTruncate.None
    line.RichText = true
    line.Parent = self.output
    
    table.insert(self.history, line)
    
    -- Limita histórico
    if #self.history > 200 then
        local old = table.remove(self.history, 1)
        old:Destroy()
    end
    
    -- Auto scroll
    task.wait()
    self.output.CanvasPosition = Vector2.new(0, self.output.CanvasSize.Y.Offset)
end

function Console:Execute(command)
    self:Print("> " .. command, Enum.MessageType.MessageInfo)
    
    local func, err = loadstring(command)
    if not func then
        self:Print("Erro: " .. err, Enum.MessageType.MessageError)
        return
    end
    
    local success, result = pcall(func)
    if not success then
        self:Print("Erro: " .. result, Enum.MessageType.MessageError)
    elseif result ~= nil then
        self:Print(tostring(result), Enum.MessageType.MessageOutput)
    end
end

-- Criação da interface principal
local function createDex()
    -- Janela principal
    local mainWindow = Window.new("DeX Explorer", UDim2.new(0, 1000, 0, 600))
    
    -- Divisor entre Explorer e Properties
    local splitter = Instance.new("Frame")
    splitter.Size = UDim2.new(0, 2, 1, -30)
    splitter.Position = UDim2.new(0.35, 0, 0, 30)
    splitter.BackgroundColor3 = colors.border
    splitter.BorderSizePixel = 0
    splitter.Parent = mainWindow.content
    
    -- Explorer (35% da largura)
    local explorerFrame = Instance.new("Frame")
    explorerFrame.Size = UDim2.new(0.35, -2, 1, -30)
    explorerFrame.Position = UDim2.new(0, 0, 0, 30)
    explorerFrame.BackgroundColor3 = colors.backgroundLight
    explorerFrame.BorderSizePixel = 0
    explorerFrame.Parent = mainWindow.content
    
    local explorerWindow = Window.new("Explorer", UDim2.new(0, 350, 0, 570))
    explorerWindow.main.Parent = explorerFrame
    explorerWindow.titleBar.Visible = false
    explorerWindow.content.Size = UDim2.new(1, 0, 1, 0)
    explorerWindow.content.Position = UDim2.new(0, 0, 0, 0)
    
    -- Properties (65% da largura)
    local propsFrame = Instance.new("Frame")
    propsFrame.Size = UDim2.new(0.65, 0, 1, -30)
    propsFrame.Position = UDim2.new(0.35, 2, 0, 30)
    propsFrame.BackgroundColor3 = colors.backgroundLight
    propsFrame.BorderSizePixel = 0
    propsFrame.Parent = mainWindow.content
    
    local propsWindow = Window.new("Properties", UDim2.new(0, 650, 0, 570))
    propsWindow.main.Parent = propsFrame
    propsWindow.titleBar.Visible = false
    propsWindow.content.Size = UDim2.new(1, 0, 1, 0)
    propsWindow.content.Position = UDim2.new(0, 0, 0, 0)
    
    -- Console separado
    local consoleWindow = Window.new("Console", UDim2.new(0, 600, 0, 300))
    consoleWindow.main.Position = UDim2.new(0.5, -300, 0.8, -150)
    
    -- Inicializa componentes
    local treeView = TreeView.new(explorerWindow)
    local propertyGrid = PropertyGrid.new(propsWindow)
    local console = Console.new(consoleWindow)
    
    -- Conecta seleção do explorer ao property grid
    treeView.signals.onSelect:Connect(function(obj)
        propertyGrid:SetObject(obj)
    end)
    
    -- Mostra as janelas
    mainWindow:Show()
    consoleWindow:Show()
    
    return {
        main = mainWindow,
        console = consoleWindow,
        tree = treeView,
        properties = propertyGrid,
        consoleObj = console
    }
end

-- Inicia o Dex
local dex = createDex()
print("DeX Explorer 2.0 iniciado com sucesso!")
