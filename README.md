-- DASH HUB BETA V17.5 (INTERACTIVE + CREDITS)
-- SHORTCUT: RIGHT SHIFT | [X] TO CLOSE

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Stats = game:GetService("Stats")

local lp = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local pGui = lp:WaitForChild("PlayerGui")

if pGui:FindFirstChild("DashHubBeta") then pGui.DashHubBeta:Destroy() end

-- ================= CONFIGS & LANGS =================
local states = { 
    showFOV = false, wallhack = false, aim = false, visible = true,
    aimStrength = 0.15, aimFOV = 150,
    espColor = Color3.fromRGB(0, 170, 255),
    lang = "PT"
}

local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 1
FOVCircle.Color = Color3.new(1, 1, 1)
FOVCircle.Transparency = 0.7
FOVCircle.Visible = false

local langData = {
    PT = {home="🏠 INÍCIO", aim="🎯 MIRA", esp="👁️ VISUAL", conf="⚙️ CONFIG", user="USUÁRIO", help="[RightShift] Minimizar / [X] Fechar", fpsB="🚀 FPS BOOSTER (Icy_HD)"},
    EN = {home="🏠 HOME", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ CONFIG", user="USER", help="[RightShift] Minimize / [X] Close", fpsB="🚀 FPS BOOSTER (Icy_HD)"},
    ES = {home="🏠 INICIO", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ AJUSTES", user="USUARIO", help="[RightShift] Minimizar / [X] Cerrar", fpsB="🚀 FPS BOOSTER (Icy_HD)"},
    FR = {home="🏠 ACCUEIL", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ REGLAGES", user="JOUEUR", help="[RightShift] Réduire / [X] Fermer", fpsB="🚀 BOOSTER FPS (Icy_HD)"},
    DE = {home="🏠 START", aim="🎯 AIM BOT", esp="👁️ ESP", conf="⚙️ EINSTELL", user="SPIELER", help="[RightShift] Minimieren / [X] Schließen", fpsB="🚀 FPS BOOSTER (Icy_HD)"}
}

-- ================= GUI BASE =================
local gui = Instance.new("ScreenGui", pGui)
gui.Name = "DashHubBeta"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.fromOffset(480, 580) 
main.Position = UDim2.new(0.5, -240, 0.5, -290)
main.BackgroundColor3 = Color3.fromRGB(15, 15, 22)
main.Active = true; main.Draggable = true
Instance.new("UICorner", main)

local stroke = Instance.new("UIStroke", main)
stroke.Thickness = 3; stroke.Color = Color3.fromRGB(0, 170, 255)

local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.fromOffset(30, 30); closeBtn.Position = UDim2.new(1, -40, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50); closeBtn.Text = "X"; closeBtn.TextColor3 = Color3.new(1,1,1); closeBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", closeBtn)
closeBtn.MouseButton1Click:Connect(function() FOVCircle:Remove(); gui:Destroy() end)

-- Sidebar
local sidebar = Instance.new("Frame", main)
sidebar.Size = UDim2.new(0, 140, 1, 0); sidebar.BackgroundColor3 = Color3.fromRGB(10, 10, 15)
Instance.new("UICorner", sidebar)

local container = Instance.new("Frame", main)
container.Position = UDim2.new(0, 150, 0, 50); container.Size = UDim2.new(1, -160, 1, -60); container.BackgroundTransparency = 1

local frames = {}
local tabButtons = {}

local function createTab(name)
    local f = Instance.new("Frame", container)
    f.Size = UDim2.new(1, 0, 1, 0); f.BackgroundTransparency = 1; f.Visible = (name == "HOME")
    frames[name] = f
    return f
end

local homeF = createTab("HOME"); local aimF = createTab("AIM BOT"); local espF = createTab("ESP"); local confF = createTab("CONFIG")

local function createTabBtn(id, name, y)
    local btn = Instance.new("TextButton", sidebar)
    btn.Size = UDim2.new(1, -15, 0, 45); btn.Position = UDim2.fromOffset(7, y)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 35); btn.Text = name; btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold; btn.TextSize = 12; Instance.new("UICorner", btn)
    
    -- Efeito Interativo
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(40, 40, 60) end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(25, 25, 35) end)
    
    btn.MouseButton1Click:Connect(function()
        for k, v in pairs(frames) do v.Visible = (k == id) end
    end)
    tabButtons[id] = btn
end

createTabBtn("HOME", "🏠 HOME", 90); createTabBtn("AIM BOT", "🎯 AIM BOT", 140); createTabBtn("ESP", "👁️ ESP", 190); createTabBtn("CONFIG", "⚙️ CONFIG", 240)

-- ================= HOME INFO & FPS BOOSTER =================
local info = Instance.new("TextLabel", homeF)
info.Size = UDim2.new(1, 0, 0, 350); info.BackgroundColor3 = Color3.fromRGB(25, 25, 40); info.TextColor3 = Color3.new(1,1,1)
info.Font = Enum.Font.GothamBold; info.TextSize = 16; info.RichText = true; Instance.new("UICorner", info)

local fpsBtn = Instance.new("TextButton", confF)
fpsBtn.Size = UDim2.new(1, 0, 0, 55); fpsBtn.Position = UDim2.new(0, 0, 1, -60)
fpsBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 100); fpsBtn.Font = Enum.Font.GothamBold; fpsBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", fpsBtn)
fpsBtn.TextSize = 14

fpsBtn.MouseButton1Click:Connect(function() 
    fpsBtn.Text = "⚡ LOADING..."
    loadstring(game:HttpGet("https://raw.githubusercontent.com/CasperFlyModz/discord.gg-rips/main/FPSBooster.lua"))() 
    task.wait(1)
    fpsBtn.Text = "✅ OPTIMIZED"
end)

task.spawn(function()
    while task.wait(0.5) do
        local l = langData[states.lang]
        local ping = math.floor(Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
        local fps = math.floor(Stats.Workspace.Heartbeat:GetValue())
        info.Text = string.format([[
<font size="35" color="#00AAFF">DASH HUB</font>
<br/><br/>
<b>👤 %s:</b> %s
<b>🆔 ID:</b> %d
<b>📅 AGE:</b> %d days
<br/>
<font color="#00FF00">-- SERVER STATUS --</font>
<b>📊 FPS:</b> %d | <b>📡 PING:</b> %d ms
<b>👥 ONLINE:</b> %d
<br/>
<font color="#FFAA00">%s</font>
]], l.user, lp.Name, lp.UserId, lp.AccountAge, fps, ping, #Players:GetPlayers(), l.help)
        tabButtons.HOME.Text = l.home; tabButtons["AIM BOT"].Text = l.aim; tabButtons.ESP.Text = l.esp; tabButtons.CONFIG.Text = l.conf
        if not fpsBtn.Text:find("✅") then fpsBtn.Text = l.fpsB end
    end
end)

-- ================= LOGICA AIMBOT =================
local function getTarget()
    local target, shortestDist = nil, states.aimFOV
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp and p.Team ~= lp.Team and p.Character and p.Character:FindFirstChild("Head") then
            local pos, on = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if on then
                local mag = (Vector2.new(pos.X, pos.Y) - UIS:GetMouseLocation()).Magnitude
                if mag < shortestDist then 
                    target = p.Character.Head
                    shortestDist = mag 
                end
            end
        end
    end
    return target
end

-- ================= CONTEÚDO INTERATIVO =================
local function createToggle(txt, y, parent, stateKey)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1, 0, 0, 50); btn.Position = UDim2.fromOffset(0, y)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 45); btn.Text = "⚪ " .. txt; btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.GothamBold; btn.TextSize = 14; Instance.new("UICorner", btn)
    
    btn.MouseButton1Click:Connect(function()
        states[stateKey] = not states[stateKey]
        btn.Text = (states[stateKey] and "🔵 " or "⚪ ") .. txt
        btn.BackgroundColor3 = states[stateKey] and Color3.fromRGB(0, 170, 255) or Color3.fromRGB(30, 30, 45)
    end)
end

createToggle("Soft Aim Assist", 0, aimF, "aim")

local function createSlider(txt, y, start, max, key, parent)
    local sFrame = Instance.new("Frame", parent); sFrame.Position = UDim2.fromOffset(0, y); sFrame.Size = UDim2.new(1,0,0,50); sFrame.BackgroundTransparency = 1
    local sBar = Instance.new("Frame", sFrame); sBar.Size = UDim2.new(0.9,0,0,5); sBar.Position = UDim2.new(0.05,0,0.7,0); sBar.BackgroundColor3 = Color3.new(0.2,0.2,0.3)
    local sDot = Instance.new("TextButton", sBar); sDot.Size = UDim2.fromOffset(15,15); sDot.Position = UDim2.new(start/max, -7, 0.5, -7); sDot.Text = ""; Instance.new("UICorner", sDot)
    local sLab = Instance.new("TextLabel", sFrame); sLab.Size = UDim2.new(1,0,0,20); sLab.Text = "⚡ " .. txt .. ": " .. start; sLab.TextColor3 = Color3.new(1,1,1); sLab.BackgroundTransparency = 1; sLab.Font = Enum.Font.GothamBold

    sDot.MouseButton1Down:Connect(function()
        local move; move = UIS.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement then
                local pos = math.clamp((input.Position.X - sBar.AbsolutePosition.X) / sBar.AbsoluteSize.X, 0, 1)
                sDot.Position = UDim2.new(pos, -7, 0.5, -7)
                local val = pos * max
                if max > 1 then val = math.floor(val) else val = math.floor(val * 100) / 100 end
                states[key] = val; sLab.Text = "⚡ " .. txt .. ": " .. val
            end
        end)
        UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then move:Disconnect() end end)
    end)
end

createSlider("AIM STRENGTH", 60, states.aimStrength, 1, "aimStrength", aimF)
createSlider("FOV SIZE", 120, states.aimFOV, 800, "aimFOV", aimF)
createToggle("Show FOV Circle", 180, aimF, "showFOV")

createToggle("Enemy Wall Hack (Highlights)", 0, espF, "wallhack")

local grid = Instance.new("Frame", espF); grid.Position = UDim2.fromOffset(0, 70); grid.Size = UDim2.new(1, 0, 0, 150); grid.BackgroundTransparency = 1
Instance.new("UIGridLayout", grid).CellSize = UDim2.fromOffset(45, 45)
for _, c in ipairs({Color3.new(0,0.7,1), Color3.new(1,0,0), Color3.new(0,1,0), Color3.new(1,1,0), Color3.new(1,1,1)}) do
    local b = Instance.new("TextButton", grid); b.BackgroundColor3 = c; b.Text = ""; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function() states.espColor = c end)
end

-- Idiomas com Bandeiras
local flags = {PT="🇧🇷 PT", EN="🇺🇸 EN", ES="🇪🇸 ES", FR="🇫🇷 FR", DE="🇩🇪 DE"}
for i, l in ipairs({"PT", "EN", "ES", "FR", "DE"}) do
    local b = Instance.new("TextButton", confF); b.Size = UDim2.new(1,0,0,35); b.Position = UDim2.fromOffset(0, (i-1)*40); b.Text = flags[l]; b.BackgroundColor3 = Color3.fromRGB(40,40,55); b.TextColor3 = Color3.new(1,1,1); b.Font = Enum.Font.GothamBold
    b.MouseButton1Click:Connect(function() states.lang = l end); Instance.new("UICorner", b)
end

-- ================= MAIN LOOP =================
RunService.RenderStepped:Connect(function()
    FOVCircle.Radius = states.aimFOV
    FOVCircle.Position = UIS:GetMouseLocation()
    FOVCircle.Visible = (states.visible and states.showFOV)

    if states.aim then
        local t = getTarget()
        if t then Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, t.Position), states.aimStrength) end
    end
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local h = p.Character:FindFirstChild("Highlight")
            if (p.Team ~= lp.Team) and states.wallhack then
                if not h then h = Instance.new("Highlight", p.Character) end
                h.FillColor = states.espColor
            elseif h then h:Destroy() end
        end
    end
end)

UIS.InputBegan:Connect(function(i, g) 
    if not g and i.KeyCode == Enum.KeyCode.RightShift then 
        states.visible = not states.visible 
        main.Visible = states.visible
    end 
end)
