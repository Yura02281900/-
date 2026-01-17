-- Brainlot v7 - Script Tab Edition
local player = game.Players.LocalPlayer
local pGui = player:WaitForChild("PlayerGui")

-- 古いUIを削除
local old = pGui:FindFirstChild("BrainlotV7") or game.CoreGui:FindFirstChild("BrainlotV7")
if old then old:Destroy() end

local userInputService = game:GetService("UserInputService")
local teleportService = game:GetService("TeleportService")
local httpService = game:GetService("HttpService")
local runService = game:GetService("RunService")

-- UI基盤
local sg = Instance.new("ScreenGui")
sg.Name = "BrainlotV7"
sg.ResetOnSpawn = false
sg.Parent = pGui

-- メインフレーム (位置固定)
local main = Instance.new("Frame")
main.Name = "MainFrame"
main.Size = UDim2.new(0, 350, 0, 220)
main.Position = UDim2.new(1, -370, 0, 20)
main.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = false
main.Parent = sg

local mainCorner = Instance.new("UICorner", main)
mainCorner.CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", main)
stroke.Thickness = 3
stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- 虹色アニメーション
task.spawn(function()
    local h = 0
    while true do
        stroke.Color = Color3.fromHSV(h, 0.7, 1)
        h = (h + 0.005) % 1
        runService.RenderStepped:Wait()
    end
end)

-- サイドバー
local sidebar = Instance.new("Frame", main)
sidebar.Size = UDim2.new(0, 100, 1, 0)
sidebar.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
Instance.new("UICorner", sidebar).CornerRadius = UDim.new(0, 10)

-- コンテンツエリア
local content = Instance.new("Frame", main)
content.Position = UDim2.new(0, 110, 0, 10)
content.Size = UDim2.new(1, -120, 1, -20)
content.BackgroundTransparency = 1

local function createPage()
    local f = Instance.new("Frame", content)
    f.Size = UDim2.new(1, 0, 1, 0)
    f.BackgroundTransparency = 1
    f.Visible = false
    return f
end

local pageMain = createPage()
local pageScan = createPage()
local pageScript = createPage()
pageMain.Visible = true

-- タブ切り替え
local function createTab(name, pos, targetPage)
    local b = Instance.new("TextButton", sidebar)
    b.Size = UDim2.new(0.9, 0, 0, 35)
    b.Position = UDim2.new(0.05, 0, 0, pos)
    b.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
    b.Text = name
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    b.TextSize = 12
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    b.MouseButton1Click:Connect(function()
        pageMain.Visible = false
        pageScan.Visible = false
        pageScript.Visible = false
        targetPage.Visible = true
    end)
end
createTab("MAIN", 10, pageMain)
createTab("SCANNER", 50, pageScan)
createTab("SCRIPT", 90, pageScript)

-- ボタン作成関数
local function createBtn(name, pos, color, parent)
    local b = Instance.new("TextButton", parent)
    b.Size = UDim2.new(1, 0, 0, 35)
    b.Position = pos
    b.BackgroundColor3 = color
    b.Text = name
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamBold
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    return b
end

-- MAINページ
local btnJump = createBtn("INF JUMP: OFF", UDim2.new(0, 0, 0, 0), Color3.fromRGB(200, 50, 50), pageMain)
local btnResp = createBtn("RESPAWN", UDim2.new(0, 0, 0, 45), Color3.fromRGB(180, 130, 40), pageMain)
local btnKick = createBtn("KICK (ERROR)", UDim2.new(0, 0, 0, 90), Color3.fromRGB(50, 50, 50), pageMain)
local btnHop = createBtn("SERVER HOP", UDim2.new(0, 0, 0, 135), Color3.fromRGB(70, 70, 140), pageMain)

-- SCANNERページ
local inputBox = Instance.new("TextBox", pageScan)
inputBox.Size = UDim2.new(1, 0, 0, 40)
inputBox.PlaceholderText = "金額 (10M, 500K等)"
inputBox.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
inputBox.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", inputBox)
local statusLabel = Instance.new("TextLabel", pageScan)
statusLabel.Size = UDim2.new(1, 0, 0, 30)
statusLabel.Position = UDim2.new(0, 0, 0, 50)
statusLabel.Text = "待機中..."
statusLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
statusLabel.BackgroundTransparency = 1
local btnStartScan = createBtn("SCAN & HOP", UDim2.new(0, 0, 0, 90), Color3.fromRGB(50, 120, 50), pageScan)

-- SCRIPTページ (新設)
local btnLag = createBtn("LAG SCRIPT", UDim2.new(0, 0, 0, 0), Color3.fromRGB(100, 50, 150), pageScript)
btnLag.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet("https://raw.githubusercontent.com/Dxvz7dt/NewN7ghts/refs/heads/main/Update-Fps%20killers"))()
end)

--- 収納機能 ---
local openBtn = Instance.new("TextButton", sg)
openBtn.Size = UDim2.new(0, 35, 0, 60)
openBtn.Position = UDim2.new(1, -35, 0, 100)
openBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
openBtn.Text = "＜"
openBtn.TextColor3 = Color3.new(1, 1, 1)
openBtn.Visible = false
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", openBtn).Color = Color3.new(1, 1, 1)

local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.new(0, 25, 0, 25)
closeBtn.Position = UDim2.new(1, -30, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(150, 50, 50)
closeBtn.Text = "×"
closeBtn.TextColor3 = Color3.new(1, 1, 1)
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)

closeBtn.MouseButton1Click:Connect(function()
    main:TweenPosition(UDim2.new(1, 10, 0, 20), "Out", "Quad", 0.4, true, function()
        main.Visible = false
        openBtn.Visible = true
    end)
end)
openBtn.MouseButton1Click:Connect(function()
    main.Visible = true
    openBtn.Visible = false
    main:TweenPosition(UDim2.new(1, -370, 0, 20), "Out", "Quad", 0.4, true)
end)

--- ロジック実行 ---
local function serverHop()
    statusLabel.Text = "サーバー収集中..."
    local servers = {}
    local url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100"
    local s, r = pcall(function() return httpService:JSONDecode(game:HttpGet(url)) end)
    if s and r.data then
        for _, v in pairs(r.data) do
            if v.id ~= game.JobId and v.playing < v.maxPlayers then
                table.insert(servers, v.id)
            end
        end
    end
    if #servers > 0 then
        statusLabel.Text = "転送中..."
        teleportService:TeleportToPlaceInstance(game.PlaceId, servers[math.random(1, #servers)], player)
    else
        teleportService:Teleport(game.PlaceId)
    end
end
btnHop.MouseButton1Click:Connect(serverHop)

btnStartScan.MouseButton1Click:Connect(function()
    local t = inputBox.Text:upper():gsub(",", "")
    local target = t:find("M") and tonumber(t:gsub("M", "")) * 1000000 or t:find("K") and tonumber(t:gsub("K", "")) * 1000 or tonumber(t)
    if not target then statusLabel.Text = "数値エラー" return end
    statusLabel.Text = "スキャン中..."
    local found = false
    for _, p in pairs(game.Players:GetPlayers()) do
        local stats = p:FindFirstChild("leaderstats") or p:FindFirstChildOfClass("Configuration")
        if stats then
            for _, v in pairs(stats:GetDescendants()) do
                if (v:IsA("NumberValue") or v:IsA("IntValue")) and v.Value >= target then
                    found = true break
                end
            end
        end
    end
    if found then
        statusLabel.Text = "発見！ホップ停止"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
    else
        statusLabel.Text = "不在。ホップします"
        task.wait(0.5)
        serverHop()
    end
end)

local infJumpEnabled = false
userInputService.JumpRequest:Connect(function()
    if not infJumpEnabled then return end
    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local p = Instance.new("Part", workspace)
        p.Size = Vector3.new(4, 0.5, 4)
        p.CFrame = player.Character.HumanoidRootPart.CFrame * CFrame.new(0, -3.2, 0)
        p.Transparency = 1
        p.Anchored = true
        player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Landing)
        task.delay(0.1, function() p:Destroy() end)
    end
end)
btnJump.MouseButton1Click:Connect(function()
    infJumpEnabled = not infJumpEnabled
    btnJump.Text = infJumpEnabled and "INF JUMP: ON" or "INF JUMP: OFF"
    btnJump.BackgroundColor3 = infJumpEnabled and Color3.fromRGB(50, 180, 50) or Color3.fromRGB(200, 50, 50)
end)
btnResp.MouseButton1Click:Connect(function() if player.Character then player.Character.Humanoid.Health = 0 end end)
btnKick.MouseButton1Click:Connect(function() player:Kick("Error 0x8004100E") end)
