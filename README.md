-- DASH HUB BETA V18.2 (RE-RESTORED + FULL FIX)
-- SHORTCUT: RIGHT SHIFT | [X] TO CLOSE

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local Lighting = game:GetService("Lighting")

local lp = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local pGui = lp:WaitForChild("PlayerGui")

-- Cache de performance
local WorldToViewportPoint = Camera.WorldToViewportPoint
local GetPlayers = Players.GetPlayers
local RaycastParamsNew = RaycastParams.new

-- ================= CONFIGS & LANGS =================
local states = { 
    showFOV = false, wallhack = false, aim = false, trigger = false, visible = true,
    norecoil = false, tpMagnet = false, 
    isTPKeyDown = false, spinSpeed = 30,
    aimStrength = 0.15, aimFOV = 150,
    espColor = Color3.fromRGB(0, 100, 255), -- AZUL DASH
    lang = "PT", triggerDebounce = false,
    fly = false, infAmmo = false, instantReload = false, flySpeed = 50,
    noclip = false, 
    hideKills = false,
    identitySpoof = false 
}

-- ================= ANTI-DETECÇÃO (PROTEÇÃO DE METATABLE & SPOOF) =================
local raw_name = "DashHubBeta"
if getgenv and hookmetamethod then
    local old; old = hookmetamethod(game, "__index", function(self, key)
        if not checkcaller() then
            if (key == "Name" or key == "Parent") and self == pGui:FindFirstChild(raw_name) then
                return nil
            end
            if states.identitySpoof and self == lp then
                if key == "Name" or key == "DisplayName" or key == "CharacterAppearanceId" then 
                    return "Player" 
                end
                if key == "UserId" then 
                    return 1 
                end
            end
        end
        return old(self, key)
    end)
end

if pGui:FindFirstChild("DashHubBeta") then pGui.DashHubBeta:Destroy() end

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2 
FOVCircle.Color = Color3.new(1, 1, 1)
FOVCircle.Transparency = 1 
FOVCircle.Visible = false
FOVCircle.Filled = false

local langData = {
    PT = {home="🏠 INÍCIO", aim="🎯 MIRA", esp="👁️ VISUAL", conf="⚙️ CONFIG", user="USUÁRIO", help="[RightShift] Minimizar / [X] Fechar / [E] TP Spin", fps="LIMPAR MAPA (FPS)"},
    EN = {home="🏠 HOME", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ CONFIG", user="USER", help="[RightShift] Minimize / [X] Close / [E] TP Spin", fps="FPS BOOSTER"},
    ES = {home="🏠 INICIO", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ AJUSTES", user="USUARIO", help="[RightShift] Minimizar / [X] Cerrar", fps="OPTIMIZAR"},
    FR = {home="🏠 ACCUEIL", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ REGLAGES", user="JOUEUR", help="[RightShift] Réduire / [X] Fermer", fps="BOOSTER FPS"},
    DE = {home="🏠 START", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ EINSTELL", user="SPIELER", help="[RightShift] Minimieren / [X] Schließen", fps="FPS BOOST"}
}

-- ================= LOGIC FUNCTIONS =================

local function isVisible(part)
    local char = lp.Character
    if not char then return false end
    local params = RaycastParamsNew()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {char, part.Parent}
    local result = workspace:Raycast(Camera.CFrame.Position, part.Position - Camera.CFrame.Position, params)
    return result == nil
end

-- MANTIDO PARA O AIMBOT (PRECISA DO MOUSE)
local function getTarget(ignoreWalls)
    local target, shortestDist = nil, states.aimFOV
    local mousePos = UIS:GetMouseLocation()
    for _, p in ipairs(GetPlayers(Players)) do
        if p ~= lp and p.Team ~= lp.Team and p.Character then
            local head = p.Character:FindFirstChild("Head")
            if head then
                local pos, on = WorldToViewportPoint(Camera, head.Position)
                if on or ignoreWalls then
                    local mag = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                    if mag < shortestDist then
                        if ignoreWalls or isVisible(head) then
                            target = head; shortestDist = mag
                        end
                    end
                end
            end
        end
    end
    return target
end

-- NOVA FUNÇÃO PARA O TELEPORT (NÃO PRECISA DO MOUSE)
local function getClosestEnemyRoot()
    local target, shortestDist = nil, math.huge
    local myHRP = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not myHRP then return nil end

    for _, p in ipairs(GetPlayers(Players)) do
        if p ~= lp and p.Team ~= lp.Team and p.Character then
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            if hrp then
                local dist = (myHRP.Position - hrp.Position).Magnitude
                if dist < shortestDist then
                    target = hrp; shortestDist = dist
                end
            end
        end
    end
    return target
end

-- ================= GUI BASE (DASH HUB WHITE & BLUE) =================
local gui = Instance.new("ScreenGui")
gui.Name = raw_name
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = pGui 

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromOffset(480, 580)
main.Position = UDim2.new(0.5, -240, 0.5, -290)
main.BackgroundColor3 = Color3.fromRGB(255, 255, 255) 
main.Active = true
main.Draggable = true
main.Visible = true 
Instance.new("UICorner", main)

local stroke = Instance.new("UIStroke", main)
stroke.Thickness = 3; stroke.Color = Color3.fromRGB(0, 100, 255) 

local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.fromOffset(30, 30); closeBtn.Position = UDim2.new(1, -40, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50); closeBtn.Text = "X"; closeBtn.TextColor3 = Color3.new(1,1,1); closeBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", closeBtn)
closeBtn.MouseButton1Click:Connect(function() FOVCircle:Remove(); gui:Destroy() end)

local sidebar = Instance.new("Frame", main)
sidebar.Size = UDim2.new(0, 140, 1, 0); sidebar.BackgroundColor3 = Color3.fromRGB(240, 240, 245); Instance.new("UICorner", sidebar)

local container = Instance.new("Frame", main)
container.Position = UDim2.new(0, 150, 0, 50); container.Size = UDim2.new(1, -160, 1, -60); container.BackgroundTransparency = 1

local frames = {}
local tabButtons = {}

local function createTab(name)
    local f = Instance.new("ScrollingFrame", container)
    f.Size = UDim2.new(1, 0, 1, 0); f.BackgroundTransparency = 1; f.Visible = (name == "HOME")
    f.CanvasSize = UDim2.new(0,0,2,0); f.ScrollBarThickness = 2
    frames[name] = f
    return f
end

local homeF = createTab("HOME"); local aimF = createTab("AIM BOT"); local espF = createTab("ESP"); local confF = createTab("CONFIG")

local function createTabBtn(id, name, y)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(1, -15, 0, 45); btn.Position = UDim2.fromOffset(7, y)
    btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255); btn.Text = name; btn.TextColor3 = Color3.fromRGB(0, 100, 255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 11
    Instance.new("UICorner", btn)
    local bStroke = Instance.new("UIStroke", btn)
    bStroke.Color = Color3.fromRGB(0, 100, 255); bStroke.Thickness = 1
    btn.MouseButton1Click:Connect(function() 
        for k, v in pairs(frames) do v.Visible = (k == id) end 
    end)
    tabButtons[id] = btn
end

createTabBtn("HOME", "🏠 INÍCIO", 90); createTabBtn("AIM BOT", "🎯 MIRA", 140); createTabBtn("ESP", "👁️ VISUAL", 190); createTabBtn("CONFIG", "⚙️ CONFIG", 240)

local info = Instance.new("TextLabel", homeF)
info.Size = UDim2.new(1, 0, 1, 0); info.BackgroundColor3 = Color3.fromRGB(255, 255, 255); info.TextColor3 = Color3.fromRGB(30, 30, 30); info.Font = Enum.Font.GothamBold; info.TextSize = 14; info.RichText = true; info.TextYAlignment = Enum.TextYAlignment.Top; Instance.new("UICorner", info)

task.spawn(function()
    while task.wait(1) do
        if not gui.Parent then break end
        local l = langData[states.lang] or langData.EN
        pcall(function()
            local ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            local fps = math.floor(Stats.Workspace.Heartbeat:GetValue())
            local displayUser = states.identitySpoof and "Player" or (states.hideKills and "[PROTECTED]" or lp.Name)
            local displayStatus = states.hideKills and "STATUS: HIDDEN" or "ACCOUNT AGE: " .. lp.AccountAge .. " days"
            info.Text = string.format([[
<font size="30" color="#00AAFF">DASH HUB</font>
<br/><br/>
<b>👤 %s:</b> %s
<b>🆔 ID:</b> %s
<b>📅 %s</b>
<br/>
<font color="#00FF00">-- SERVER STATUS --</font>
<b>📊 FPS:</b> %d | <b>📡 PING:</b> %d ms
<br/><br/>
<font color="#0064FF">%s</font>
]], l.user, displayUser, (states.identitySpoof and "1" or "[PROTECTED]"), displayStatus, fps, ping, l.help)
            tabButtons.HOME.Text = l.home; tabButtons["AIM BOT"].Text = l.aim; tabButtons.ESP.Text = l.esp; tabButtons.CONFIG.Text = l.conf
        end)
    end
end)

-- ================= TOOLS =================
local function createToggle(txt, y, parent, stateKey)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 40); btn.Position = UDim2.fromOffset(0, y)
    btn.BackgroundColor3 = states[stateKey] and Color3.fromRGB(0, 100, 255) or Color3.fromRGB(245, 245, 250)
    btn.Text = (states[stateKey] and "🔵 " or "⚪ ") .. txt; btn.TextColor3 = states[stateKey] and Color3.new(1,1,1) or Color3.fromRGB(0, 100, 255); btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(function()
        states[stateKey] = not states[stateKey]
        btn.Text = (states[stateKey] and "🔵 " or "⚪ ") .. txt
        btn.BackgroundColor3 = states[stateKey] and Color3.fromRGB(0, 100, 255) or Color3.fromRGB(245, 245, 250)
        btn.TextColor3 = states[stateKey] and Color3.new(1,1,1) or Color3.fromRGB(0, 100, 255)
    end)
end

local function createSlider(txt, y, start, max, key, parent)
    local sFrame = Instance.new("Frame", parent); sFrame.Position = UDim2.fromOffset(0, y); sFrame.Size = UDim2.new(1,0,0,50); sFrame.BackgroundTransparency = 1
    local sBar = Instance.new("Frame", sFrame); sBar.Size = UDim2.new(0.9,0,0,5); sBar.Position = UDim2.new(0.05,0,0.7,0); sBar.BackgroundColor3 = Color3.fromRGB(200, 200, 200)
    local sDot = Instance.new("TextButton", sBar); sDot.Size = UDim2.fromOffset(16,16); sDot.Position = UDim2.new(start/max, -8, 0.5, -8); sDot.Text = ""; sDot.BackgroundColor3 = Color3.fromRGB(0, 100, 255); Instance.new("UICorner", sDot)
    local sLab = Instance.new("TextLabel", sFrame); sLab.Size = UDim2.new(1,0,0,20); sLab.Text = "⚡ " .. txt .. ": " .. start; sLab.TextColor3 = Color3.fromRGB(30,30,30); sLab.BackgroundTransparency = 1; sLab.Font = Enum.Font.GothamBold; sLab.TextSize = 12
    sDot.MouseButton1Down:Connect(function()
        local move; move = UIS.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                local pos = math.clamp((input.Position.X - sBar.AbsolutePosition.X) / sBar.AbsoluteSize.X, 0, 1)
                sDot.Position = UDim2.new(pos, -8, 0.5, -8)
                local val = pos * max; states[key] = val; sLab.Text = "⚡ " .. txt .. ": " .. (max > 1 and math.floor(val) or math.floor(val*100)/100)
            end
        end)
        UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then move:Disconnect() end end)
    end)
end

-- ABA CONFIG
local flags = {PT="🇧🇷 PT", EN="🇺🇸 EN", ES="🇪🇸 ES", FR="🇫🇷 FR", DE="🇩🇪 DE"}
for i, l in ipairs({"PT", "EN", "ES", "FR", "DE"}) do
    local b = Instance.new("TextButton", confF); b.Size = UDim2.new(0.18, 0, 0, 35); b.Position = UDim2.new((i-1)*0.2, 0, 0, 0)
    b.Text = flags[l]; b.BackgroundColor3 = Color3.fromRGB(240,240,245); b.TextColor3 = Color3.fromRGB(0,100,255); b.Font = Enum.Font.GothamBold; b.TextSize = 10; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function() states.lang = l end)
end

createToggle("Identity Mask (Spoof)", 110, confF, "identitySpoof")
createToggle("No Clip (Paredes)", 155, confF, "noclip")
createToggle("Ocultar Kills/Nick HUD", 200, confF, "hideKills")

local fpsBtn = Instance.new("TextButton", confF)
fpsBtn.Size = UDim2.new(1, 0, 0, 50); fpsBtn.Position = UDim2.fromOffset(0, 50); fpsBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100); fpsBtn.Text = "🚀 FPS BOOSTER (CLEAN MAP)"; fpsBtn.TextColor3 = Color3.new(1,1,1); fpsBtn.Font = Enum.Font.GothamBold; Instance.new("UICorner", fpsBtn)
fpsBtn.MouseButton1Click:Connect(function()
    for _, v in ipairs(game:GetDescendants()) do
        if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic
        elseif v:IsA("Decal") or v:IsA("Texture") then v:Destroy()
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled = false end
    end
    Lighting.GlobalShadows = false; settings().Rendering.QualityLevel = 1
end)

-- ABA AIM BOT
createToggle("Soft Aim Assist", 0, aimF, "aim")
createToggle("Trigger Bot", 45, aimF, "trigger")
createToggle("No Recoil (Universal)", 90, aimF, "norecoil")
createToggle("Fly (W,S,A,D)", 345, aimF, "fly")
createSlider("FLY SPEED", 390, states.flySpeed, 500, "flySpeed", aimF)
createToggle("Infinite Ammo", 445, aimF, "infAmmo")
createToggle("Instant Reload", 490, aimF, "instantReload")

local tpBtn = Instance.new("TextButton", aimF)
tpBtn.Size = UDim2.new(1, 0, 0, 40); tpBtn.Position = UDim2.fromOffset(0, 135); tpBtn.BackgroundColor3 = states.tpMagnet and Color3.fromRGB(0, 200, 255) or Color3.fromRGB(0, 100, 255)
tpBtn.Text = "🧲 SPIN MAGNET (HOLD E)"; tpBtn.TextColor3 = Color3.new(1,1,1); tpBtn.Font = Enum.Font.GothamBold; Instance.new("UICorner", tpBtn)
tpBtn.MouseButton1Click:Connect(function()
    states.tpMagnet = not states.tpMagnet
    tpBtn.BackgroundColor3 = states.tpMagnet and Color3.fromRGB(0, 200, 255) or Color3.fromRGB(0, 100, 255)
end)

createSlider("AIM STRENGTH", 185, states.aimStrength, 1, "aimStrength", aimF)
createSlider("FOV SIZE", 240, states.aimFOV, 800, "aimFOV", aimF)
createToggle("Show FOV Circle", 300, aimF, "showFOV")

-- ABA ESP
createToggle("Enemy Wall Hack", 0, espF, "wallhack")
local grid = Instance.new("Frame", espF); grid.Position = UDim2.fromOffset(0, 50); grid.Size = UDim2.new(1, 0, 0, 100); grid.BackgroundTransparency = 1
Instance.new("UIGridLayout", grid).CellSize = UDim2.fromOffset(40, 40)
for _, c in ipairs({Color3.new(0,0.7,1), Color3.new(1,0,0), Color3.new(0,1,0), Color3.new(1,1,0), Color3.new(1,1,1)}) do
    local b = Instance.new("TextButton", grid); b.BackgroundColor3 = c; b.Text = ""; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function() states.espColor = c end)
end

-- ================= SYSTEM LÓGICA =================

RunService.Stepped:Connect(function()
    if states.noclip and lp.Character then
        for _, v in ipairs(lp.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

UIS.InputBegan:Connect(function(i, g) 
    if not g then
        if i.KeyCode == Enum.KeyCode.RightShift then 
            states.visible = not states.visible; main.Visible = states.visible
        elseif i.KeyCode == Enum.KeyCode.E then 
            states.isTPKeyDown = true
        end
    end
end)

UIS.InputEnded:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.E then states.isTPKeyDown = false end
end)

-- ESP SYSTEM
task.spawn(function()
    while task.wait(0.5) do
        for _, p in ipairs(GetPlayers(Players)) do
            if p ~= lp and p.Character then
                local highlight = p.Character:FindFirstChild("DashHighlight")
                if states.wallhack and p.Team ~= lp.Team then
                    if not highlight then
                        highlight = Instance.new("Highlight", p.Character)
                        highlight.Name = "DashHighlight"
                    end
                    highlight.Enabled = true
                    highlight.FillColor = states.espColor
                else
                    if highlight then highlight.Enabled = false end
                end
            end
        end
    end
end)

-- FLY & AMMO
RunService.Stepped:Connect(function()
    if states.fly and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = lp.Character.HumanoidRootPart
        local moveDir = Vector3.zero
        if UIS:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + Camera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - Camera.CFrame.LookVector end
        if UIS:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - Camera.CFrame.RightVector end
        if UIS:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + Camera.CFrame.RightVector end
        hrp.Velocity = moveDir * states.flySpeed
    end
end)

task.spawn(function()
    while task.wait(0.1) do
        pcall(function()
            if states.infAmmo or states.instantReload then
                local tool = lp.Character and lp.Character:FindFirstChildOfClass("Tool")
                if tool then
                    for _, v in ipairs(tool:GetDescendants()) do
                        if states.infAmmo and (v.Name:find("Ammo") or v.Name:find("Clip")) then v.Value = 999 end
                        if states.instantReload and (v.Name:find("Reload") or v.Name:find("Time")) then v.Value = 0 end
                    end
                end
            end
        end)
    end
end)

local spinAngle = 0

RunService.RenderStepped:Connect(function()
    if not gui.Parent then return end
    
    FOVCircle.Radius = states.aimFOV
    FOVCircle.Position = UIS:GetMouseLocation()
    FOVCircle.Visible = (states.visible and states.showFOV)
    FOVCircle.Color = states.espColor

    -- CORREÇÃO: TELEPORT SEM PRECISAR MIRAR
    if states.tpMagnet and states.isTPKeyDown then
        local targetHRP = getClosestEnemyRoot() -- Busca automática por distância
        if targetHRP and targetHRP.Parent then
            local char = lp.Character
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            
            if hrp then
                spinAngle = spinAngle + states.spinSpeed
                local angleRad = math.rad(spinAngle)
                -- 4 studs de distância e 1.5 para cima
                local offset = Vector3.new(math.cos(angleRad) * 4, 1.5, math.sin(angleRad) * 4)
                local finalPos = targetHRP.Position + offset
                
                hrp.CFrame = CFrame.lookAt(finalPos, targetHRP.Position)
                hrp.Velocity = Vector3.zero
                Camera.CFrame = CFrame.lookAt(Camera.CFrame.Position, targetHRP.Position)
            end
        end
    else
        spinAngle = 0
    end

    -- AIMBOT (MANTIDO COM FOV/MIRA)
    if states.aim then
        local targetHead = getTarget(false)
        if targetHead then
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, targetHead.Position), states.aimStrength) 
        end
    end

    -- NO RECOIL & TRIGGER
    if states.norecoil and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
        local rot = Camera.CFrame.Rotation
        task.spawn(function()
            RunService.RenderStepped:Wait()
            Camera.CFrame = CFrame.new(Camera.CFrame.Position) * rot
        end)
    end

    if states.trigger and not states.triggerDebounce then
        local mt = lp:GetMouse().Target
        if mt and mt.Parent and mt.Parent:FindFirstChild("Humanoid") then
            local tp = Players:GetPlayerFromCharacter(mt.Parent)
            if tp and tp.Team ~= lp.Team then
                states.triggerDebounce = true
                if mouse1press then mouse1press() task.wait(0.05) mouse1release() end
                task.wait(0.1)
                states.triggerDebounce = false
            end
        end
    end
end)
