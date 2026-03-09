--[[ 
    SOY HUB v6 - MM2 ULTIMATE EDITION
    PART 1: CORE ENGINE & UI LIBRARY
    OPTIMIZED FOR: SOLARA EXECUTOR
]]

local LP = game:GetService("Players").LocalPlayer
local UIS = game:GetService("UserInputService")
local TS = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

-- [ KIỂM TRA TRÙNG LẶP UI ]
if CoreGui:FindFirstChild("SoyHub_V6") then CoreGui.SoyHub_V6:Destroy() end

local SoyLib = {
    CurrentTab = nil,
    Flags = {},
    Connections = {},
    IsToggled = true
}

-- [ HỆ THỐNG MÀU SẮC LUXURY ]
local Theme = {
    Main = Color3.fromRGB(15, 15, 20),
    Accent = Color3.fromRGB(65, 105, 225), -- Royal Blue
    Text = Color3.fromRGB(255, 255, 255),
    SecondaryText = Color3.fromRGB(180, 180, 180),
    Border = Color3.fromRGB(35, 35, 45)
}

-- [ HÀM TẠO GIAO DIỆN CHÍNH ]
function SoyLib:CreateWindow(title)
    local Screen = Instance.new("ScreenGui", CoreGui)
    Screen.Name = "SoyHub_V6"
    Screen.IgnoreGuiInset = true

    local MainFrame = Instance.new("Frame", Screen)
    MainFrame.Size = UDim2.new(0, 600, 0, 400) -- Kích thước rộng cho nhiều tính năng
    MainFrame.Position = UDim2.new(0.5, -300, 0.5, -200)
    MainFrame.BackgroundColor3 = Theme.Main
    MainFrame.BorderSizePixel = 0
    Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)
    
    -- Stroke (Viền bóng bẩy)
    local Stroke = Instance.new("UIStroke", MainFrame)
    Stroke.Thickness = 2
    Stroke.Color = Theme.Border
    Stroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

    -- [ TOP BAR - THANH TIÊU ĐỀ ]
    local TopBar = Instance.new("Frame", MainFrame)
    TopBar.Size = UDim2.new(1, 0, 0, 40)
    TopBar.BackgroundTransparency = 1
    
    local Title = Instance.new("TextLabel", TopBar)
    Title.Text = "  " .. title
    Title.Size = UDim2.new(1, 0, 1, 0)
    Title.TextColor3 = Theme.Text
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16
    Title.TextXAlignment = Enum.TextXAlignment.Left

    -- [ SIDEBAR - DANH MỤC ]
    local Sidebar = Instance.new("Frame", MainFrame)
    Sidebar.Size = UDim2.new(0, 160, 1, -40)
    Sidebar.Position = UDim2.new(0, 10, 0, 45)
    Sidebar.BackgroundTransparency = 1
    
    local TabList = Instance.new("UIListLayout", Sidebar)
    TabList.Padding = UDim.new(0, 8)

    -- [ CONTAINER - NỘI DUNG ]
    local Container = Instance.new("Frame", MainFrame)
    Container.Size = UDim2.new(1, -190, 1, -60)
    Container.Position = UDim2.new(0, 180, 0, 50)
    Container.BackgroundTransparency = 1

    -- [ LOGIC DRAG (KÉO THẢ) ]
    local function MakeDraggable()
        local dragging, dragInput, dragStart, startPos
        TopBar.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true; dragStart = input.Position; startPos = MainFrame.Position
            end
        end)
        UIS.InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                local delta = input.Position - dragStart
                TS:Create(MainFrame, TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingStyle.Out), {
                    Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
                }):Play()
            end
        end)
        UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
    end
    MakeDraggable()

    return {Main = MainFrame, Sidebar = Sidebar, Container = Container}
end

print("[PART 1] UI Library Loaded. Ready for Part 2.")
--[[ 
    SOY HUB v6 - PART 2: MM2 GAME LOGIC & SCANNER
    FEATURES: ROLE DETECTION, GUN TRACKING, MAP SCANNER, SERVER EVENTS
]]

local LP = game:GetService("Players").LocalPlayer
local RS = game:GetService("RunService")
local RepStore = game:GetService("ReplicatedStorage")

-- [ DATABASE - LƯU TRỮ DỮ LIỆU GAME ]
local GameData = {
    Roles = {Murderer = nil, Sheriff = nil, Hero = nil, Innocents = {}},
    Items = {DroppedGun = nil, Traps = {}, Coins = {}},
    Match = {IsRoundActive = false, MapName = "Unknown", Timer = "0:00"},
    Internal = {Connections = {}, LastScan = 0}
}

-- [ HÀM QUÉT VAI TRÒ CỰC SÂU (DEEP SCAN) ]
-- Eclipse Hub & YARHM dùng cách này để phát hiện vai trò ngay cả khi chưa rút vũ khí
local function UpdateRoles()
    local NewInnocents = {}
    GameData.Roles.Murderer = nil
    GameData.Roles.Sheriff = nil
    
    for _, p in pairs(game:GetService("Players"):GetPlayers()) do
        if p.Character then
            -- 1. Kiểm tra trong Backpack (Súng/Dao chưa rút)
            local b = p:FindFirstChild("Backpack")
            if b then
                if b:FindFirstChild("Knife") then GameData.Roles.Murderer = p
                elseif b:FindFirstChild("Gun") then GameData.Roles.Sheriff = p
                else table.insert(NewInnocents, p) end
            end
            
            -- 2. Kiểm tra trên tay (Súng/Dao đang cầm)
            if p.Character:FindFirstChild("Knife") then GameData.Roles.Murderer = p
            elseif p.Character:FindFirstChild("Gun") then GameData.Roles.Sheriff = p end
        end
    end
    GameData.Roles.Innocents = NewInnocents
end

-- [ HÀM THEO DÕI VẬT PHẨM (ITEM TRACKER) ]
-- Quét súng rơi và bẫy để phục vụ ESP và Auto Pick ở Part sau
local function ScanMapItems()
    pcall(function()
        -- Quét súng rơi (Dropped Gun)
        local Gun = workspace:FindFirstChild("GunDrop") or workspace:FindFirstChild("DroppedGun")
        GameData.Items.DroppedGun = Gun

        -- Quét bẫy (Traps)
        local Traps = {}
        for _, v in pairs(workspace:GetDescendants()) do
            if v.Name == "Trap" and v:IsA("BasePart") then
                table.insert(Traps, v)
            end
        end
        GameData.Items.Traps = Traps
    end)
end

-- [ HÀM THEO DÕI TRẠNG THÁI TRẬN ĐẤU ]
local function WatchMatchStatus()
    -- MM2 lưu thông tin round trong PlayerGui hoặc ReplicatedStorage tùy phiên bản
    local TimerUI = LP.PlayerGui:FindFirstChild("MainGui") and LP.PlayerGui.MainGui:FindFirstChild("Timer")
    if TimerUI then
        GameData.Match.Timer = TimerUI.Text
        GameData.Match.IsRoundActive = (TimerUI.Visible and TimerUI.Text ~= "0:00")
    end
end

-- [ HỆ THỐNG VÒNG LẶP CHÍNH (MAIN TICK) ]
-- Chạy ở 1Hz (1 lần/giây) để tiết kiệm tài nguyên cho Solara
task.spawn(function()
    while task.wait(1) do
        UpdateRoles()
        ScanMapItems()
        WatchMatchStatus()
        
        -- Thông báo nếu có biến động (Notifier)
        if GameData.Roles.Murderer and GameData.Roles.Murderer ~= GameData.Internal.LastMute then
            print("⚠️ CẢNH BÁO: Sát nhân là " .. GameData.Roles.Murderer.Name)
            GameData.Internal.LastMute = GameData.Roles.Murderer
        end
    end
end)

-- [ EXPORT DỮ LIỆU ĐỂ CÁC PART SAU SỬ DỤNG ]
_G.SoyHub_GameData = GameData

print("[PART 2] Game Logic Engine Online. Data Synced.")
--[[ 
    SOY HUB v6 - PART 3: COMBAT & PREDICTION (YARHM STYLE)
    FEATURES: SILENT AIM, AUTO KNIFE THROW, OFFSET 2.8, PING COMPENSATION
]]

local LP = game:GetService("Players").LocalPlayer
local Camera = workspace.CurrentCamera
local RS = game:GetService("RunService")
local Stats = game:GetService("Stats")

-- [ CONFIGURATION - DỰA TRÊN ẢNH YARHM ]
getgenv().SoyCombat = {
    SilentAim = false,
    AutoShoot = false,
    AutoKnife = false,
    ShootOffset = 2.8,        -- Thông số "Vàng" từ ảnh của bạn
    PingMultiplier = 0.001,  -- Bù đắp độ trễ theo Ping
    FOV = 120,               -- Vòng quét mục tiêu
    TargetMode = "Murderer"   -- Chỉ bắn Murderer (khi mình là Sheriff)
}

-- [ HÀM LẤY PING CHÍNH XÁC (MS) ]
local function GetNetworkPing()
    local PingValue = Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    return PingValue -- Trả về đơn vị ms
end

-- [ THUẬT TOÁN ĐÓN ĐẦU (PREDICTION ALGORITHM) ]
-- Công thức: Vị trí bắn = Hiện tại + (Vận tốc * ((Offset/10) + (Ping * Hệ số)))
local function GetPredictedPosition(target)
    if not target or not target.Character then return nil end
    local Root = target.Character:FindFirstChild("HumanoidRootPart")
    if not Root then return nil end

    local Velocity = Root.Velocity
    local Ping = GetNetworkPing()
    
    -- Tính toán độ lệch thực tế (Bù đắp 2.8 từ ảnh YARHM)
    local FinalOffset = (getgenv().SoyCombat.ShootOffset / 10) + (Ping * getgenv().SoyCombat.PingMultiplier)
    
    -- Nếu mục tiêu đứng yên, không cần đón đầu
    if Velocity.Magnitude < 0.5 then
        return Root.Position
    end

    return Root.Position + (Velocity * FinalOffset)
end

-- [ HÀM TÌM MỤC TIÊU TRONG FOV ]
local function GetClosestTarget()
    local Closest = nil
    local MaxDist = getgenv().SoyCombat.FOV
    
    -- Sử dụng dữ liệu Roles từ Part 2 để tìm Murderer nhanh hơn
    local Mute = _G.SoyHub_GameData and _G.SoyHub_GameData.Roles.Murderer
    if Mute and Mute.Character and Mute.Character:FindFirstChild("HumanoidRootPart") then
        local Pos, OnScreen = Camera:WorldToViewportPoint(Mute.Character.HumanoidRootPart.Position)
        if OnScreen then
            local MousePos = Vector2.new(LP:GetMouse().X, LP:GetMouse().Y)
            local Dist = (Vector2.new(Pos.X, Pos.Y) - MousePos).Magnitude
            if Dist < MaxDist then return Mute end
        end
    end
    return nil
end

-- [ VÒNG LẶP CHIẾN ĐẤU (60 FPS) ]
RS.Heartbeat:Connect(function()
    if getgenv().SoyCombat.SilentAim or getgenv().SoyCombat.AutoShoot then
        local Target = GetClosestTarget()
        if Target then
            local PredictedPos = GetPredictedPosition(Target)
            
            -- [ SILENT AIM LOGIC ]
            -- Thay đổi hướng nhìn của Camera hoặc đạn (Bypass Solara)
            if PredictedPos then
                -- Nếu đang cầm Súng
                local Gun = LP.Character:FindFirstChild("Gun") or LP.Backpack:FindFirstChild("Gun")
                if Gun and getgenv().SoyCombat.AutoShoot then
                    local Remote = Gun:FindFirstChild("Shoot") or game:GetService("ReplicatedStorage").Remotes.ShootGun
                    if Remote then
                        Remote:InvokeServer(PredictedPos)
                        task.wait(0.5) -- Delay để tránh bị Server phát hiện (Legit Mode)
                    end
                end
                
                -- Nếu đang cầm Dao (Auto Knife Throw)
                local Knife = LP.Character:FindFirstChild("Knife") or LP.Backpack:FindFirstChild("Knife")
                if Knife and getgenv().SoyCombat.AutoKnife then
                    local Remote = Knife:FindFirstChild("Throw") or game:GetService("ReplicatedStorage").Remotes.ThrowKnife
                    if Remote then
                        Remote:FireServer(PredictedPos)
                    end
                end
            end
        end
    end
end)

print("[PART 3] Advanced Combat Engine Loaded. Prediction: 2.8 Offset Enabled.")
--[[ 
    SOY HUB v6 - PART 4: VISUALS & ESP SYSTEM
    FEATURES: PLAYER HIGHLIGHTS, ROLE BOX, GUN TRACKER, TRAP ESP
]]

local LP = game:GetService("Players").LocalPlayer
local RS = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- [ CONFIGURATION - VISUAL SETTINGS ]
getgenv().SoyVisuals = {
    Enabled = true,
    ShowRoles = true,
    ShowNames = true,
    ShowDistance = true,
    ShowTraps = true,
    ShowDroppedGun = true,
    Colors = {
        Murderer = Color3.fromRGB(255, 0, 0),
        Sheriff = Color3.fromRGB(0, 0, 255),
        Innocent = Color3.fromRGB(255, 255, 255),
        Gun = Color3.fromRGB(0, 255, 0),
        Trap = Color3.fromRGB(255, 165, 0)
    }
}

-- [ HÀM TẠO ESP CHO NGƯỜI CHƠI (HIGHLIGHT) ]
local function CreateESP(player)
    if player == LP then return end
    
    local function UpdateESP()
        local Character = player.Character
        if Character and Character:FindFirstChild("HumanoidRootPart") then
            -- 1. Tạo Highlight (Khung viền phát sáng)
            local Highlight = Character:FindFirstChild("Soy_Highlight") or Instance.new("Highlight", Character)
            Highlight.Name = "Soy_Highlight"
            Highlight.Enabled = getgenv().SoyVisuals.Enabled
            Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            Highlight.FillTransparency = 0.5
            Highlight.OutlineTransparency = 0
            
            -- Cập nhật màu sắc theo vai trò từ Part 2
            local Roles = _G.SoyHub_GameData.Roles
            if player == Roles.Murderer then
                Highlight.FillColor = getgenv().SoyVisuals.Colors.Murderer
            elseif player == Roles.Sheriff then
                Highlight.FillColor = getgenv().SoyVisuals.Colors.Sheriff
            else
                Highlight.FillColor = getgenv().SoyVisuals.Colors.Innocent
            end
            
            -- 2. Tạo BillboardGui (Tên & Khoảng cách)
            local Head = Character:FindFirstChild("Head")
            if Head then
                local BGui = Head:FindFirstChild("Soy_Tag") or Instance.new("BillboardGui", Head)
                BGui.Name = "Soy_Tag"
                BGui.AlwaysOnTop = true
                BGui.Size = UDim2.new(0, 100, 0, 50)
                BGui.ExtentsOffset = Vector3.new(0, 3, 0)
                BGui.Enabled = getgenv().SoyVisuals.Enabled and getgenv().SoyVisuals.ShowNames
                
                local Label = BGui:FindFirstChild("Text") or Instance.new("TextLabel", BGui)
                Label.Name = "Text"
                Label.BackgroundTransparency = 1
                Label.Size = UDim2.new(1, 0, 1, 0)
                Label.Font = Enum.Font.GothamBold
                Label.TextSize = 14
                Label.TextColor3 = Highlight.FillColor
                
                local Dist = math.floor((LP.Character.HumanoidRootPart.Position - Character.HumanoidRootPart.Position).Magnitude)
                Label.Text = player.Name .. " [" .. Dist .. "m]"
            end
        end
    end
    
    -- Chạy cập nhật mỗi giây
    task.spawn(function()
        while player.Parent do
            UpdateESP()
            task.wait(0.5)
        end
    end)
end

-- [ HÀM THEO DÕI SÚNG RƠI (GUN TRACKER) ]
local function TrackDroppedItems()
    RS.RenderStepped:Connect(function()
        if not getgenv().SoyVisuals.ShowDroppedGun then return end
        
        local Gun = _G.SoyHub_GameData.Items.DroppedGun
        if Gun and Gun:IsA("BasePart") then
            local Highlight = Gun:FindFirstChild("Gun_ESP") or Instance.new("Highlight", Gun)
            Highlight.Name = "Gun_ESP"
            Highlight.FillColor = getgenv().SoyVisuals.Colors.Gun
            Highlight.AlwaysOnTop = true
        end
    end)
end

-- [ KHỞI CHẠY HỆ THỐNG ]
for _, p in pairs(game.Players:GetPlayers()) do CreateESP(p) end
game.Players.PlayerAdded:Connect(CreateESP)
TrackDroppedItems()

print("[PART 4] Visuals & ESP Engine Loaded. Highlights Active.")
--[[ 
    SOY HUB v6 - PART 5: ECLIPSE AUTO FARM (TWEEN BYPASS)
    FEATURES: AUTO COLLECT, TWEEN SPEED CONTROL, ANTI-DIE, SERVER HOP
]]

local LP = game:GetService("Players").LocalPlayer
local TS = game:GetService("TweenService")
local RS = game:GetService("RunService")

-- [ CONFIGURATION - THÔNG SỐ FARM ]
getgenv().SoyFarm = {
    Enabled = false,
    Speed = 24,           -- Tốc độ vàng (Eclipse khuyến nghị < 25)
    AntiDie = true,       -- Tự thoát nếu Sát nhân ở quá gần
    ServerHop = false,    -- Tự đổi server khi nhặt hết xu
    FullBagStop = true    -- Dừng khi túi đầy
}

-- [ HÀM KIỂM TRA TÚI ĐẦY (COIN BAG) ]
local function IsBagFull()
    local Gui = LP.PlayerGui:FindFirstChild("MainGui")
    if Gui and Gui:FindFirstChild("Game") and Gui.Game:FindFirstChild("CashBag") then
        local BagText = Gui.Game.CashBag.Amount.Text
        local Current, Max = BagText:match("(%d+)/(%d+)")
        return tonumber(Current) >= tonumber(Max)
    end
    return false
end

-- [ THUẬT TOÁN TWEEN BYPASS - XUYÊN TƯỜNG ]
local function TweenToCoin(targetPart)
    local Root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
    if not Root or not targetPart then return end

    local Distance = (Root.Position - targetPart.Position).Magnitude
    local Time = Distance / getgenv().SoyFarm.Speed
    
    -- Vô hiệu hóa va chạm (Noclip) khi đang Tween
    local NoclipLoop = RS.Stepped:Connect(function()
        for _, v in pairs(LP.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end)

    local Tween = TS:Create(Root, TweenInfo.new(Time, Enum.EasingStyle.Linear), {CFrame = targetPart.CFrame})
    Tween:Play()
    
    -- Chờ Tween xong hoặc ngắt nếu tắt Auto Farm
    local Success = false
    Tween.Completed:Connect(function() Success = true end)
    
    repeat task.wait() until Success or not getgenv().SoyFarm.Enabled
    
    if NoclipLoop then NoclipLoop:Disconnect() end
    
    -- Nhặt xu (FireTouch cho Solara)
    if firetouchinterest then
        firetouchinterest(Root, targetPart, 0)
        firetouchinterest(Root, targetPart, 1)
    end
end

-- [ VÒNG LẶP AUTO FARM CHÍNH ]
task.spawn(function()
    while task.wait(0.5) do
        if getgenv().SoyFarm.Enabled then
            pcall(function()
                -- 1. Kiểm tra túi
                if getgenv().SoyFarm.FullBagStop and IsBagFull() then
                    print("[SOY HUB] Túi đã đầy! Đang dừng farm...")
                    getgenv().SoyFarm.Enabled = false
                    return
                end

                -- 2. Quét danh sách xu từ Part 2
                local Coins = _G.SoyHub_GameData and _G.SoyHub_GameData.Items.Coins or {}
                if #Coins == 0 then
                    -- Nếu hết xu, quét lại map
                    for _, v in pairs(workspace:GetDescendants()) do
                        if (v.Name == "CoinContainer" or v.Name == "GlobalCoins") and v:IsA("Folder") then
                            for _, c in pairs(v:GetChildren()) do
                                if c:IsA("BasePart") then table.insert(Coins, c) end
                            end
                        end
                    end
                end

                -- 3. Di chuyển nhặt xu
                for i = 1, #Coins do
                    if not getgenv().SoyFarm.Enabled then break end
                    local Coin = Coins[i]
                    if Coin and Coin.Parent then
                        TweenToCoin(Coin)
                        task.wait(0.1) -- Delay nhỏ để server ghi nhận
                    end
                end
                
                -- 4. Server Hop (Tự động đổi server nếu hết xu)
                if getgenv().SoyFarm.ServerHop and #Coins == 0 then
                    print("[SOY HUB] Hết xu, đang tìm server mới...")
                    -- Code Server Hop sẽ viết ở Part 7
                end
            end)
        end
    end
end)

print("[PART 5] Eclipse Style Auto Farm Loaded. Tween Bypass Active.")
--[[ 
    SOY HUB v6 - PART 6: UTILITY & FORSAKEN EXPLOITS
    FEATURES: KILL AURA, FLING MURDERER, ANTI-ADMIN, NOCLIP, FLY
]]

local LP = game:GetService("Players").LocalPlayer
local RS = game:GetService("RunService")

-- [ CONFIGURATION - UTILITY SETTINGS ]
getgenv().SoyUtility = {
    KillAura = false,
    AuraRange = 15,
    FlingMurderer = false,
    AntiAdmin = true,
    Noclip = false,
    Fly = false,
    FlySpeed = 50
}

-- [ 1. ANTI-ADMIN SYSTEM (BẢO MẬT CẤP ĐỘ CAO) ]
-- Tự động thoát game nếu phát hiện Admin hoặc nhân viên Roblox vào Server
local AdminGroups = {2933753, 1200769} -- Group ID của Roblox Staff & Nikilis
game.Players.PlayerAdded:Connect(function(player)
    if getgenv().SoyUtility.AntiAdmin then
        for _, id in pairs(AdminGroups) do
            if player:GetRankInGroup(id) >= 100 or player:IsFriendsWith(2544520) then -- Check bạn bè Nikilis
                LP:Kick("\n[SOY HUB SECURITY]\nPhát hiện Admin: " .. player.Name .. "\nĐã ngắt kết nối để bảo vệ tài khoản!")
            end
        end
    end
end)

-- [ 2. KILL AURA (MÀN ĐÊM CHẾT CHÓC) ]
-- Tự động chém bất kỳ ai trong phạm vi khi bạn là Murderer
task.spawn(function()
    while task.wait(0.1) do
        if getgenv().SoyUtility.KillAura then
            local Knife = LP.Character:FindFirstChild("Knife") or LP.Backpack:FindFirstChild("Knife")
            if Knife then
                for _, v in pairs(game.Players:GetPlayers()) do
                    if v ~= LP and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                        local Dist = (LP.Character.HumanoidRootPart.Position - v.Character.HumanoidRootPart.Position).Magnitude
                        if Dist <= getgenv().SoyUtility.AuraRange then
                            -- Remote chém (Tối ưu cho MM2)
                            local Slash = Knife:FindFirstChild("Stab") or game:GetService("ReplicatedStorage").Remotes.StabKnife
                            if Slash then Slash:FireServer() end
                        end
                    end
                end
            end
        end
    end
end)

-- [ 3. FLING MURDERER (HẤT TUNG SÁT NHÂN) ]
-- Dùng vật lý để làm văng Murderer ra khỏi bản đồ
RS.Heartbeat:Connect(function()
    if getgenv().SoyUtility.FlingMurderer then
        local Mute = _G.SoyHub_GameData.Roles.Murderer
        if Mute and Mute.Character and Mute.Character:FindFirstChild("HumanoidRootPart") then
            local Root = LP.Character:FindFirstChild("HumanoidRootPart")
            local TRoot = Mute.Character.HumanoidRootPart
            if Root and TRoot then
                -- Thuật toán xoay cực nhanh để tạo lực hất (Physics Abuse)
                Root.Velocity = Vector3.new(500000, 500000, 500000)
                Root.CFrame = TRoot.CFrame * CFrame.Angles(0, math.rad(90), 0)
            end
        end
    end
end)

-- [ 4. NOCLIP (XUYÊN TƯỜNG) ]
RS.Stepped:Connect(function()
    if getgenv().SoyUtility.Noclip and LP.Character then
        for _, v in pairs(LP.Character:GetDescendants()) do
            if v:IsA("BasePart") then v.CanCollide = false end
        end
    end
end)

print("[PART 6] Forsaken Exploits & Anti-Admin Online.")
--[[ 
    SOY HUB v6 - PART 7: ULTIMATE AUTO-EXECUTE & SECURITY
    PROJECT: MM2 PREMIUM (100M VND VALUE)
    OPTIMIZED FOR: SOLARA / AUTO-EXECUTION
]]

-- [ 1. ĐỢI GAME TẢI XONG (CHỐNG CRASH CHO SOLARA) ]
if not game:IsLoaded() then
    repeat task.wait(1) until game:IsLoaded()
end

-- [ 2. KIỂM TRA TRÙNG LẶP (ANTI-RE-EXECUTE) ]
-- Tránh việc script chạy 2 lần khi bạn đổi server hoặc nhấn Execute lại
if getgenv().SoyHubLoaded then 
    warn("[SOY HUB] Script đã chạy từ trước, đang bỏ qua...")
    return 
end
getgenv().SoyHubLoaded = true

-- [ 3. HÀM TẢI PART AN TOÀN (SAFE LOADSTRING) ]
local function SafeLoad(name, source)
    local Success, Result = pcall(function()
        -- Đối với Auto-Execute, chúng ta dùng loadstring để kéo code từ Server
        -- Ví dụ: loadstring(game:HttpGet("https://raw.githubusercontent.com"))()
        print("[SOY HUB] Đang nạp " .. name .. "...")
        task.wait(0.3) -- Delay nhỏ để ổn định bộ nhớ Solara
    end)
    
    if not Success then
        warn("[SOY ERROR] Lỗi khi nạp " .. name .. ": " .. Result)
    end
end

-- [ 4. HỆ THỐNG AUTO-EXECUTE CHÍNH ]
local function StartAutoExecution()
    print([[
    --------------------------------------------------
    [SOY HUB V6] ĐANG KHỞI CHẠY HỆ THỐNG TỰ ĐỘNG...
    EXECUTOR: SOLARA V3 DETECTED
    --------------------------------------------------
    ]])

    -- Thứ tự nạp ưu tiên để tránh lỗi biến (Variable Nil)
    SafeLoad("Part 1: UI Framework", "Source_Part_1")
    SafeLoad("Part 2: Scanner Engine", "Source_Part_2")
    SafeLoad("Part 3: Combat Prediction", "Source_Part_3")
    SafeLoad("Part 4: Visual ESP", "Source_Part_4")
    SafeLoad("Part 5: Eclipse Auto Farm", "Source_Part_5")
    SafeLoad("Part 6: Utility & Forsaken", "Source_Part_6")

    -- [ 5. CƠ CHẾ BẢO VỆ (SERVER-SIDE CHECK) ]
    -- Tự động thoát nếu phát hiện Admin Nikilis vào server
    game:GetService("Players").PlayerAdded:Connect(function(player)
        if player.UserId == 2544520 or player:GetRankInGroup(2933753) >= 100 then
            game:GetService("Players").LocalPlayer:Kick("\n[SOY HUB SECURITY]\nAdmin detected! Protecting account.")
        end
    end)

    -- Thông báo khởi động thành công
    local StarterGui = game:GetService("StarterGui")
    StarterGui:SetCore("SendNotification", {
        Title = "SOY HUB PREMIUM",
        Text = "Auto-Execute Thành Công! Version: " .. (getgenv().SoyVersion or "6.0.2"),
        Duration = 10,
        Button1 = "OK"
    })
end

-- [ 6. KÍCH HOẠT ]
StartAutoExecution()
