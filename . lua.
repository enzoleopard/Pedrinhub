-- Pedrin Hub - Versão Final Funcional (Key = "Pedrin")
-- LocalScript em StarterPlayerScripts

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Estado global
local state = {
    floatOn = false,
    floatBV = nil,
    floatSpeed = 50,
    moveVector = Vector3.new(0,0,0),
    hitLagOn = false,
    tweenFixOn = false,
    platformOn = false,
    platform = nil,
    slowBlocks = {},
    slowTweenSequence = false,
    slowTweening = false,
    antiKickOn = false,
    antiKickConnection = nil
}

-- Character references
local Character, Humanoid, HRP
local function updateChar(char)
    Character = char
    Humanoid = char:WaitForChild("Humanoid")
    HRP = char:WaitForChild("HumanoidRootPart")
end

if LocalPlayer.Character then updateChar(LocalPlayer.Character) end
LocalPlayer.CharacterAdded:Connect(updateChar)

-- Feedback helper
local function animateFeedback(text)
    local fb = Instance.new("TextLabel")
    fb.Size = UDim2.new(0,0,0,36)
    fb.Position = UDim2.new(0.5,0,0,60)
    fb.AnchorPoint = Vector2.new(0.5,0)
    fb.BackgroundColor3 = Color3.fromRGB(0,0,0)
    fb.BackgroundTransparency = 0.2
    fb.TextColor3 = Color3.new(1,1,1)
    fb.Font = Enum.Font.SourceSansBold
    fb.TextScaled = true
    fb.Text = text
    fb.Parent = PlayerGui
    Instance.new("UICorner", fb).CornerRadius = UDim.new(0,6)
    local openTween = TweenService:Create(fb, TweenInfo.new(0.28, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {Size = UDim2.new(0,260,0,36)})
    openTween:Play()
    openTween.Completed:Wait()
    task.delay(1, function()
        local close = TweenService:Create(fb, TweenInfo.new(0.28), {Size = UDim2.new(0,0,0,36)})
        close:Play()
        close.Completed:Wait()
        fb:Destroy()
    end)
end

-- Button helpers
local function createButton(parent,text,callback,x,y)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,130,0,30)
    btn.Position = UDim2.new(0,x,0,y)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(170,0,0)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextScaled = true
    btn.Parent = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
    btn.MouseButton1Click:Connect(function()
        local press = TweenService:Create(btn, TweenInfo.new(0.09, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0,150,0,35)})
        press:Play()
        press.Completed:Wait()
        local back = TweenService:Create(btn, TweenInfo.new(0.09, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size = UDim2.new(0,130,0,30)})
        back:Play()
        pcall(callback)
        animateFeedback(text.." ativado com sucesso!")
    end)
    return btn
end

local function createToggleButton(parent,text,callback,x,y)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0,130,0,30)
    btn.Position = UDim2.new(0,x,0,y)
    btn.Text = text
    btn.BackgroundColor3 = Color3.fromRGB(170,0,0)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextScaled = true
    btn.Parent = parent
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
    local onColor = Color3.fromRGB(0,255,0)
    local offColor = Color3.fromRGB(170,0,0)
    local toggled = false
    btn.MouseButton1Click:Connect(function()
        local press = TweenService:Create(btn, TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size=UDim2.new(0,150,0,35)})
        press:Play()
        press.Completed:Wait()
        local back = TweenService:Create(btn, TweenInfo.new(0.08, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Size=UDim2.new(0,130,0,30)})
        back:Play()
        toggled = not toggled
        btn.BackgroundColor3 = toggled and onColor or offColor
        pcall(callback,toggled)
        animateFeedback(text .. (toggled and " ligado" or " desligado"))
    end)
    return btn
end

-- Anti Kick
local function antiKick(on)
    state.antiKickOn = on
    if on and not state.antiKickConnection then
        state.antiKickConnection = LocalPlayer.Idled:Connect(function()
            game:GetService("VirtualUser"):ClickButton2(Vector2.new(0,0))
        end)
    elseif not on and state.antiKickConnection then
        state.antiKickConnection:Disconnect()
        state.antiKickConnection = nil
    end
end

-- Slow Tween Mini GUI
local function createSlowTweenMini(ContentFrame)
    if state.slowTweening then return end
    state.slowTweening = true
    local miniGui = Instance.new("Frame", ContentFrame)
    miniGui.Size = UDim2.new(0,220,0,180)
    miniGui.Position = UDim2.new(0,300,0,10)
    miniGui.BackgroundColor3 = Color3.fromRGB(30,30,30)
    Instance.new("UICorner", miniGui).CornerRadius = UDim.new(0,6)

    createButton(miniGui,"Criar Bloco",function()
        if HRP and HRP.Parent then
            local part = Instance.new("Part")
            part.Size = Vector3.new(4,1,4)
            part.Anchored = true
            part.Position = HRP.Position - Vector3.new(0,6,0)
            part.BrickColor = BrickColor.Random()
            part.Parent = Workspace
            table.insert(state.slowBlocks, part)
        end
    end,10,10)

    createButton(miniGui,"Tween Sequência",function()
        if state.slowTweenSequence then return end
        state.slowTweenSequence = true
        task.spawn(function()
            for _,part in ipairs(state.slowBlocks) do
                if not state.slowTweenSequence then break end
                if HRP and HRP.Parent and part and part.Parent then
                    local dist = (part.Position - HRP.Position).Magnitude
                    local duration = math.clamp(dist/20,0.45,1.2)
                    local tw = TweenService:Create(HRP, TweenInfo.new(duration, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {CFrame = part.CFrame + Vector3.new(0,3,0)})
                    tw:Play()
                    tw.Completed:Wait()
                end
            end
            state.slowTweenSequence = false
        end)
    end,10,50)

    createButton(miniGui,"Parar Tween",function()
        state.slowTweenSequence = false
        animateFeedback("Tween Sequência parada")
    end,10,90)

    createButton(miniGui,"Limpar Blocos",function()
        for _,part in ipairs(state.slowBlocks) do
            if part and part.Parent then part:Destroy() end
        end
        state.slowBlocks = {}
    end,10,130)
end

-- Main GUI
local function createMainGUI()
    if PlayerGui:FindFirstChild("PedrinHub") then return end
    local MainGui = Instance.new("ScreenGui", PlayerGui)
    MainGui.Name = "PedrinHub"
    MainGui.ResetOnSpawn = false

    local MainFrame = Instance.new("Frame", MainGui)
    MainFrame.Size = UDim2.new(0,560,0,340)
    MainFrame.Position = UDim2.new(0.5,-280,0.2,0)
    MainFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
    MainFrame.Active = true
    MainFrame.Draggable = true
    Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,14)

    local TitleLabel = Instance.new("TextLabel", MainFrame)
    TitleLabel.Size = UDim2.new(0,240,0,36)
    TitleLabel.Position = UDim2.new(0.5,-120,0,8)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Text = "Pedrin Hub"
    TitleLabel.TextColor3 = Color3.new(1,1,1)
    TitleLabel.Font = Enum.Font.SourceSansBold
    TitleLabel.TextScaled = true

    local TabsFrame = Instance.new("Frame", MainFrame)
    TabsFrame.Size = UDim2.new(0,130,1,-64)
    TabsFrame.Position = UDim2.new(0,12,0,52)
    TabsFrame.BackgroundTransparency = 1

    local ContentFrame = Instance.new("Frame", MainFrame)
    ContentFrame.Size = UDim2.new(1,-150,1,-64)
    ContentFrame.Position = UDim2.new(0,148,0,52)
    ContentFrame.BackgroundColor3 = Color3.fromRGB(18,18,18)
    Instance.new("UICorner", ContentFrame).CornerRadius = UDim.new(0,8)

    local function clearContent()
        for _,c in ipairs(ContentFrame:GetChildren()) do
            if c:IsA("GuiObject") then c:Destroy() end
        end
    end

    -- createTab helper
    local function createTab(name, y, callback)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0,130,0,30)
        btn.Position = UDim2.new(0,0,0,y)
        btn.Text = name
        btn.BackgroundColor3 = Color3.fromRGB(170,0,0)
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Font = Enum.Font.SourceSansBold
        btn.TextScaled = true
        btn.Parent = TabsFrame
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
        btn.MouseButton1Click:Connect(function()
            for _,b in ipairs(TabsFrame:GetChildren()) do
                if b:IsA("TextButton") then b.BackgroundColor3 = Color3.fromRGB(170,0,0) end
            end
            btn.BackgroundColor3 = Color3.fromRGB(0,255,0)
            clearContent()
            callback(ContentFrame)
        end)
        return btn
    end

    -- Steal tab
    createTab("Steal", 0, function(cf)
        createToggleButton(cf,"Float",function(on)
            state.floatOn = on
            if on then
                task.spawn(function()
                    while state.floatOn do
                        if HRP and HRP.Parent then
                            RunService.RenderStepped:Wait()
                            if not state.floatBV then
                                local bv = Instance.new("BodyVelocity")
                                bv.MaxForce = Vector3.new(1e5,1e5,1e5)
                                bv.Velocity = Vector3.new(0,0,0)
                                bv.Parent = HRP
                                state.floatBV = bv
                            end
                            state.floatBV.Velocity = Vector3.new(state.moveVector.X*state.floatSpeed, 0, state.moveVector.Z*state.floatSpeed)
                        else
                            break
                        end
                    end
                    if state.floatBV then state.floatBV:Destroy(); state.floatBV=nil end
                end)
            end
        end,10,10)

        createToggleButton(cf,"Hit Lag",function(on)
            state.hitLagOn = on
            if on then
                task.spawn(function()
                    while state.hitLagOn do
                        if Humanoid and Humanoid.Parent then
                            Humanoid.WalkSpeed = 0
                            task.wait(0.05)
                            Humanoid.WalkSpeed = 16
                            task.wait(0.05)
                        else
                            break
                        end
                    end
                    if Humanoid then Humanoid.WalkSpeed = 16 end
                end)
            else
                if Humanoid then Humanoid.WalkSpeed = 16 end
            end
        end,10,50)

        createToggleButton(cf,"Tween Fix",function(on)
            state.tweenFixOn = on
            if on then
                task.spawn(function()
                    while state.tweenFixOn do
                        if Character and Character.Parent then
                            for _,part in ipairs(Character:GetDescendants()) do
                                if part:IsA("BasePart") then
                                    part.CanCollide = false
                                end
                            end
                        else
                            break
                        end
                        task.wait(0.1)
                    end
                    if Character then
                        for _,part in ipairs(Character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.CanCollide = true
                            end
                        end
                    end
                end)
            else
                if Character then
                    for _,part in ipairs(Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = true
                        end
                    end
                end
            end
        end,10,90)

        createToggleButton(cf,"Plataforma",function(on)
            state.platformOn = on
            if on then
                task.spawn(function()
                    while state.platformOn do
                        if HRP and HRP.Parent then
                            if not state.platform then
                                local plat = Instance.new("Part")
                                plat.Anchored = true
                                plat.Size = Vector3.new(6,1,6)
                                plat.Position = HRP.Position - Vector3.new(0,3,0)
                                plat.BrickColor = BrickColor.new("Really black")
                                plat.Parent = Workspace
                                state.platform = plat
                            else
                                state.platform.Position = HRP.Position - Vector3.new(0,3,0)
                            end
                        else
                            break
                        end
                        task.wait()
                    end
                    if state.platform then state.platform:Destroy(); state.platform=nil end
                end)
            else
                if state.platform then state.platform:Destroy(); state.platform=nil end
            end
        end,10,130)

        createToggleButton(cf,"Slow Tween",function(on)
            if on then createSlowTweenMini(cf) else state.slowTweening=false end
        end,10,170)
    end)

    -- Help tab
    createTab("Help", 40, function(cf)
        createToggleButton(cf,"Anti Kick",antiKick,10,10)
        local info = Instance.new("TextLabel", cf)
        info.Size = UDim2.new(1,-20,0,40)
        info.Position = UDim2.new(0,10,0,50)
        info.BackgroundTransparency = 1
        info.Text = "Use as opções com responsabilidade."
        info.TextColor3 = Color3.new(1,1,1)
        info.Font = Enum.Font.SourceSans
        info.TextScaled = true
    end)

    -- Créditos tab
    createTab("Créditos", 80, function(cf)
        local creditLabel = Instance.new("TextLabel", cf)
        creditLabel.Size = UDim2.new(1,-20,1,-20)
        creditLabel.Position = UDim2.new(0,10,0,10)
        creditLabel.BackgroundTransparency = 1
        creditLabel.Text = "Neckhurt e Drakes 🇧🇷"
        creditLabel.TextColor3 = Color3.new(1,1,1)
        creditLabel.Font = Enum.Font.SourceSansBold
        creditLabel.TextScaled = true
    end)
end

-- Key Panel
local function createKeyPanel()
    if PlayerGui:FindFirstChild("PedrinKeyGui") then return end

    local gui = Instance.new("ScreenGui", PlayerGui)
    gui.Name = "PedrinKeyGui"
    gui.ResetOnSpawn = false

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0,320,0,170)
    frame.Position = UDim2.new(0.5,-160,0.4,0)
    frame.BackgroundColor3 = Color3.fromRGB(10,10,10)
    frame.Active = true
    frame.Draggable = true
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0,10)

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1,0,0,40)
    title.Position = UDim2.new(0,0,0,8)
    title.BackgroundTransparency = 1
    title.Text = "Digite a Key:"
    title.TextColor3 = Color3.new(1,1,1)
    title.Font = Enum.Font.SourceSansBold
    title.TextScaled = true

    local box = Instance.new("TextBox", frame)
    box.Size = UDim2.new(0.85,0,0,32)
    box.Position = UDim2.new(0.075,0,0,60)
    box.PlaceholderText = "Digite a Key"
    box.Text = ""
    box.BackgroundColor3 = Color3.fromRGB(20,20,20)
    box.TextColor3 = Color3.new(1,1,1)
    box.ClearTextOnFocus = false
    Instance.new("UICorner", box).CornerRadius = UDim.new(0,6)

    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.5,0,0,30)
    btn.Position = UDim2.new(0.25,0,0,105)
    btn.Text = "Confirmar"
    btn.BackgroundColor3 = Color3.fromRGB(170,0,0)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextScaled = true
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)

    btn.MouseButton1Click:Connect(function()
        if box.Text == "Pedrin" then
            gui:Destroy()
            animateFeedback("Key correta! Carregando Hub...")
            createMainGUI()
        else
            animateFeedback("Key incorreta!")
        end
    end)
end

-- Start
createKeyPanel()
