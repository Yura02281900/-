-- Rayfield UIライブラリを読み込む
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- UIウィンドウの作成
local Window = Rayfield:CreateWindow({
    Name = "ブレインロットをかくせいさせないで",
    LoadingTitle = "高速滑らか移動版",
    LoadingSubtitle = "速いけど検知されにくい",
    ConfigurationSaving = { Enabled = false }
})

-- ローカルプレイヤー
local Player = game:GetService("Players").LocalPlayer

-- 移動設定
local MOVE_SETTINGS = {
    SPEED = 120, -- 高速移動（studs/秒）
    INTERVAL = 0.01, -- 更新間隔（秒）→ 100FPS相当
    HEIGHT_OFFSET = 2 -- 地面からの高さ
}

-- 移動状態
local IsMoving = false
local MoveThread = nil

-- 高速滑らか移動関数
local function FastSmoothTeleport(targetPosition)
    if IsMoving then
        Rayfield:Notify({
            Title = "移動中",
            Content = "既に移動中です",
            Duration = 2
        })
        return
    end
    
    local character = Player.Character
    if not character then return end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    IsMoving = true
    
    -- 開始位置と計算
    local startPos = humanoidRootPart.Position
    local distance = (startPos - targetPosition).Magnitude
    local moveTime = distance / MOVE_SETTINGS.SPEED
    
    Rayfield:Notify({
        Title = "高速移動開始",
        Content = string.format("約%.2f秒で移動します", moveTime),
        Duration = 2
    })
    
    -- 高速移動スレッド
    MoveThread = task.spawn(function()
        local direction = (targetPosition - startPos).Unit
        local totalSteps = math.floor(moveTime / MOVE_SETTINGS.INTERVAL)
        
        if totalSteps <= 0 then
            -- 非常に近い場合は直接移動
            humanoidRootPart.CFrame = CFrame.new(targetPosition)
        else
            -- 段階的に移動
            for step = 1, totalSteps do
                if not IsMoving then break end
                
                local progress = step / totalSteps
                local currentPos = startPos + (direction * distance * progress)
                
                -- スムーズに移動
                humanoidRootPart.CFrame = CFrame.new(currentPos)
                task.wait(MOVE_SETTINGS.INTERVAL)
            end
            
            -- 最終位置に正確に移動
            if IsMoving then
                humanoidRootPart.CFrame = CFrame.new(targetPosition)
            end
        end
        
        -- 移動完了
        if IsMoving then
            IsMoving = false
            Rayfield:Notify({
                Title = "移動完了",
                Content = "目的地に到着しました",
                Duration = 2
            })
        end
    end)
end

-- === テレポートタブ ===
local TeleportTab = Window:CreateTab("テレポート")

-- 移動設定
TeleportTab:CreateSection("移動設定")

TeleportTab:CreateSlider({
    Name = "テレポート移動速度",
    Range = {80, 250},
    Increment = 10,
    Suffix = "studs/秒",
    CurrentValue = MOVE_SETTINGS.SPEED,
    Callback = function(value)
        MOVE_SETTINGS.SPEED = value
        Rayfield:Notify({
            Title = "速度変更",
            Content = string.format("テレポート速度を %d studs/秒 に設定", value),
            Duration = 2
        })
    end
})

-- 移動ボタン
TeleportTab:CreateSection("移動先")

TeleportTab:CreateButton({
    Name = "左へ高速移動",
    Callback = function()
        FastSmoothTeleport(Vector3.new(-146.35, MOVE_SETTINGS.HEIGHT_OFFSET, -16.94))
    end
})

TeleportTab:CreateButton({
    Name = "中央へ高速移動",
    Callback = function()
        FastSmoothTeleport(Vector3.new(-7.49, MOVE_SETTINGS.HEIGHT_OFFSET, -15.37))
    end
})

TeleportTab:CreateButton({
    Name = "右へ高速移動",
    Callback = function()
        FastSmoothTeleport(Vector3.new(152.07, MOVE_SETTINGS.HEIGHT_OFFSET, -16.95))
    end
})

-- 移動停止ボタン（単独で配置）
TeleportTab:CreateButton({
    Name = "移動を停止",
    Callback = function()
        IsMoving = false
        if MoveThread then
            task.cancel(MoveThread)
            MoveThread = nil
        end
        Rayfield:Notify({
            Title = "移動停止",
            Content = "移動を中断しました",
            Duration = 2
        })
    end
})

-- === プレイヤータブ ===
local PlayerTab = Window:CreateTab("プレイヤー")

-- スピード設定
PlayerTab:CreateSection("スピード設定")

local SpeedEnabled = false
local SpeedValue = 16

PlayerTab:CreateToggle({
    Name = "スピード有効化",
    CurrentValue = false,
    Callback = function(value)
        SpeedEnabled = value
    end
})

PlayerTab:CreateSlider({
    Name = "速度調整",
    Range = {16, 200},
    Increment = 1,
    CurrentValue = 16,
    Callback = function(value)
        SpeedValue = value
    end
})

-- ジャンプ設定
PlayerTab:CreateSection("ジャンプ設定")

local JumpEnabled = false
local JumpValue = 50
local InfiniteJump = false

PlayerTab:CreateToggle({
    Name = "ジャンプ有効化",
    CurrentValue = false,
    Callback = function(value)
        JumpEnabled = value
    end
})

PlayerTab:CreateSlider({
    Name = "ジャンプ力調整",
    Range = {50, 200},
    Increment = 1,
    CurrentValue = 50,
    Callback = function(value)
        JumpValue = value
    end
})

PlayerTab:CreateToggle({
    Name = "無限ジャンプ",
    CurrentValue = false,
    Callback = function(value)
        InfiniteJump = value
    end
})

-- フライ設定
PlayerTab:CreateSection("フライ")
PlayerTab:CreateButton({
    Name = "Fly GUIを実行",
    Callback = function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-fly-gui-v3-46328"))()
    end
})

-- FOV設定
PlayerTab:CreateSection("画面設定")

local FOVEnabled = false
local FOVValue = 70

PlayerTab:CreateToggle({
    Name = "FOV有効化",
    CurrentValue = false,
    Callback = function(value)
        FOVEnabled = value
    end
})

PlayerTab:CreateSlider({
    Name = "FOV調整",
    Range = {30, 120},
    Increment = 1,
    CurrentValue = 70,
    Callback = function(value)
        FOVValue = value
    end
})

-- === メインループ ===
game:GetService("RunService").Heartbeat:Connect(function()
    if IsMoving then return end -- 移動中は他の機能を一時停止
    
    local character = Player.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            if SpeedEnabled then
                humanoid.WalkSpeed = SpeedValue
            else
                humanoid.WalkSpeed = 16
            end
            
            if JumpEnabled then
                humanoid.JumpPower = JumpValue
            else
                humanoid.JumpPower = 50
            end
        end
    end
    
    local camera = workspace.CurrentCamera
    if camera then
        if FOVEnabled then
            camera.FieldOfView = FOVValue
        else
            camera.FieldOfView = 70
        end
    end
end)

-- 無限ジャンプ
game:GetService("UserInputService").JumpRequest:Connect(function()
    if InfiniteJump and not IsMoving then
        local character = Player.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end
    end
end)

-- 初期化
Rayfield:Notify({
    Title = "スクリプト起動完了",
    Content = string.format("テレポート移動速度: %d studs/秒", MOVE_SETTINGS.SPEED),
    Duration = 4
})
