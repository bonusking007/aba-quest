-- ==========================================
-- 🔒 User Whitelist Check (เช็คชื่อก่อนรัน)
-- ==========================================
local ALLOWED_USERS = {
    "Bunowaiau359", -- เปลี่ยนเป็น Username ของคุณ (เช่น "Player1")
    -- "ใส่ชื่อไอดีรองตรงนี้",     -- เอาเครื่องหมาย -- ออกถ้าต้องการเพิ่มไอดี Alt
}

local Players = game:GetService("Players")

local LocalPlayer = Players.LocalPlayer
while not LocalPlayer do
    task.wait()
    LocalPlayer = Players.LocalPlayer
end

local isAuthorized = false
for _, allowedName in ipairs(ALLOWED_USERS) do
    if string.lower(LocalPlayer.Name) == string.lower(allowedName) then
        isAuthorized = true
        break
    end
end

if not isAuthorized then
    warn("❌ [Access Denied] ผู้เล่น " .. LocalPlayer.Name .. " ไม่มีสิทธิ์รันสคริปต์นี้")
    return
end

-- ==========================================
-- 🚀 Script Farm Logic (เริ่มทำงานเมื่อผ่านการตรวจสอบ)
-- ==========================================
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local HttpService = game:GetService("HttpService")
local UserInputService = game:GetService("UserInputService")
local StarterGui = game:GetService("StarterGui")

-- ล้างค่า flag ใน getgenv เพื่อไม่ให้ค้างข้ามเซิฟบนมือถือ
if getgenv then
    getgenv()._ABA_Currently_In_PS = nil
end

-- ==========================================
-- 🔄 Auto Execute on Hop / Teleport
-- ==========================================
local function setupAutoExecuteOnHop()
    local queue = (syn and syn.queue_on_teleport) or queue_on_teleport or (fluxus and fluxus.queue_on_teleport)
    if not queue then return end

    local loaderScript = [[
        repeat task.wait() until game:IsLoaded()
        task.wait(1)
        local candidates = {
            "ABA_QuestFarm_TP_Notify.lua",
            "abaquestfarm.lua",
            "abaquestfarm/main.lua"
        }
        for _, file in ipairs(candidates) do
            if isfile and isfile(file) then
                loadstring(readfile(file))()
                return
            end
        end
    ]]

    pcall(function()
        queue(loaderScript)
    end)

    LocalPlayer.OnTeleport:Connect(function()
        pcall(function()
            queue(loaderScript)
        end)
    end)
end
setupAutoExecuteOnHop()

local vector3 = Vector3.new(0, 0, -3)
local t1 = {
	MAIN_USERNAME = "",
	SELECTED_QUEST_SLOT = 4,
	AUTO_REFRESH_QUESTS = true,
	GLITCH_QUEST_1 = 1,
	GLITCH_QUEST_2 = 2,
	REQUIRED_PLAYERS = 2,
	TP_DELAY = 3,
	PRIVATE_SERVER = "JblH87",
	PRIVATE_SERVER_DELAY = 3,
	M1_CLICK_DELAY = 0.2,
	M1_HOLD_DURATION = 0.05,
	ALT_TP_INTERVAL = 0.5,
	ALT_TP_OFFSET = vector3,
	AUTO_SKILLS = false,
	SKILL_1 = true,
	SKILL_2 = true,
	SKILL_3 = true,
	SKILL_4 = true,
	SKILL_DELAY = 0.5,
	AUTO_M1 = true,
	AUTO_TP_PLAYERS = true,
	MAIN_TP_INTERVAL = 0.2,
	MAIN_TP_DISTANCE = 3,
	ENGINE_ENABLED = false,
	AUTO_HIDE_UI = true,
	TOGGLE_UI_KEY = "L",
	TOGGLE_ENGINE_KEY = "R"
}

pcall(function()
    if not isfile or not readfile then
        return
    end

    if isfile("abaquestfarm/config.json") then
        local data = HttpService:JSONDecode(readfile("abaquestfarm/config.json"))

        if type(data) == "table" then
            for k, v in pairs(data) do
                if t1[k] ~= nil then
                    t1[k] = v
                end
            end
            print("⚙️ Configuration successfully loaded from workspace!")
        end
    end
end)

if not t1.PRIVATE_SERVER or t1.PRIVATE_SERVER == "" or t1.PRIVATE_SERVER == "Enter PS Code (Optional)" then
    t1.PRIVATE_SERVER = "JblH87"
end

local function notify(message)
    task.spawn(function()
        for _ = 1, 5 do
            local ok = pcall(function()
                StarterGui:SetCore("SendNotification", {Title = "ABA Quest Farm", Text = message, Duration = 4})
            end)
            if ok then return end
            task.wait(0.3)
        end
    end)
end

-- ==========================================
-- 🛠️ Helper Functions สำหรับตรวจสอบและเลือกตัวละคร
-- ==========================================
local function getInput()
    local bp = LocalPlayer:FindFirstChild("Backpack")
    local c = LocalPlayer.Character
    return (bp and bp:FindFirstChild("Input")) or (c and c:FindFirstChild("Input")) or LocalPlayer:FindFirstChild("Input")
end

-- ฟังก์ชันตรวจสอบว่าตัวละครปัจจุบันตรงกับชื่อเควสต์หรือไม่
local function isCharacterMatching(targetChar)
    if not targetChar or targetChar == "" then return true end
    local target = string.lower(string.gsub(targetChar, "%s+", ""))

    local char = LocalPlayer.Character
    if not char then return false end

    -- 1. เช็ค StringValue หรือ Attribute ภายใน Character Model
    local charVal = char:FindFirstChild("Character")
    if charVal and charVal:IsA("StringValue") and charVal.Value ~= "" then
        local val = string.lower(string.gsub(charVal.Value, "%s+", ""))
        if string.find(val, target, 1, true) or string.find(target, val, 1, true) then
            return true
        end
    end

    local attr = char:GetAttribute("Character")
    if attr then
        local val = string.lower(string.gsub(tostring(attr), "%s+", ""))
        if string.find(val, target, 1, true) or string.find(target, val, 1, true) then
            return true
        end
    end

    -- 2. เช็คชื่อโฟลเดอร์/สคริปต์/วัตถุภายใน Character
    for _, child in ipairs(char:GetChildren()) do
        local cName = string.lower(string.gsub(child.Name, "%s+", ""))
        if string.find(cName, target, 1, true) then
            return true
        end
        if (child:IsA("StringValue") or child:IsA("ObjectValue")) and child.Value then
            local vName = string.lower(string.gsub(tostring(child.Value), "%s+", ""))
            if string.find(vName, target, 1, true) then
                return true
            end
        end
    end

    -- 3. เช็คจาก Backpack (ชื่อสกิล หรือ Value ตัวละคร)
    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        local bpVal = bp:FindFirstChild("Character")
        if bpVal and bpVal:IsA("StringValue") and bpVal.Value ~= "" then
            local val = string.lower(string.gsub(bpVal.Value, "%s+", ""))
            if string.find(val, target, 1, true) or string.find(target, val, 1, true) then
                return true
            end
        end
        for _, item in ipairs(bp:GetChildren()) do
            local iName = string.lower(string.gsub(item.Name, "%s+", ""))
            if string.find(iName, target, 1, true) then
                return true
            end
        end
    end

    -- 4. เช็คจากข้อความบน HUD ใน PlayerGui
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    local hud = pg and pg:FindFirstChild("HUD")
    if hud then
        for _, desc in ipairs(hud:GetDescendants()) do
            if desc:IsA("TextLabel") and desc.Text ~= "" then
                local tName = string.lower(string.gsub(desc.Text, "%s+", ""))
                if string.find(tName, target, 1, true) then
                    return true
                end
            elseif desc:IsA("StringValue") and desc.Value ~= "" then
                local vName = string.lower(string.gsub(desc.Value, "%s+", ""))
                if string.find(vName, target, 1, true) then
                    return true
                end
            end
        end
    end

    return false
end

-- ฟังก์ชันสแปม Remote เปลี่ยนตัวละคร พร้อม Reset และหน่วงเวลาป้องกันแลค
local function ensureCharacter(targetChar, runningCheck)
    if not targetChar or targetChar == "" then return true end
    if isCharacterMatching(targetChar) then
        print("✅ Current character already matches: " .. targetChar)
        return true
    end

    notify("Mismatch! Switching to: " .. targetChar)
    print("🔄 Character mismatch detected. Starting switch process to: " .. targetChar)

    local attempt = 0
    local maxAttempts = 15

    while (runningCheck and runningCheck() or true) and attempt < maxAttempts do
        attempt += 1
        print(string.format("🔄 Switching character (Attempt %d/%d) to '%s'", attempt, maxAttempts, targetChar))
        notify(string.format("Switching to %s (%d/%d)...", targetChar, attempt, maxAttempts))

        -- 1. สั่งเปิดหน้าเลือกตัวละคร (Menu / CharacterSelect)
        pcall(function()
            local inp = getInput()
            if inp then
                inp:FireServer("Menu")
                task.wait(0.05)
                inp:FireServer("CharacterSelect")
            end
        end)

        -- 2. สั่ง Reset เลือด 0 และยิง Loaded เพื่อให้เกมเข้าสู่สถานะ Respawn / Menu
        pcall(function()
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                hum.Health = 0
            end
            local loaded = ReplicatedStorage:FindFirstChild("Loaded")
            if loaded then
                loaded:FireServer()
            end
        end)

        -- 3. สแปมส่ง Remote เปลี่ยนตัวละครพร้อม Delay 0.25s
        local cycleDeadline = os.clock() + 7
        while (runningCheck and runningCheck() or true) and os.clock() < cycleDeadline do
            -- ส่งผ่าน Backpack.ServerTraits.Choose
            pcall(function()
                local bp = LocalPlayer:FindFirstChild("Backpack")
                local st = bp and bp:FindFirstChild("ServerTraits")
                local ch = st and st:FindFirstChild("Choose")
                if ch then
                    ch:FireServer(targetChar)
                    task.wait(0.05)
                    ch:FireServer("PLAY")
                end
            end)

            -- ส่งผ่าน Input Remote
            pcall(function()
                local inp = getInput()
                if inp then
                    inp:FireServer("CharacterButton", targetChar)
                    task.wait(0.05)
                    inp:FireServer("ClickPlay")
                end
            end)

            -- กดยืนยัน Respawn Done ถ้ามี GUI
            pcall(function()
                local pg = LocalPlayer:FindFirstChild("PlayerGui")
                local r = pg and pg:FindFirstChild("Respawning")
                if r and r:FindFirstChild("Done") then
                    r.Done:FireServer()
                end
            end)

            -- ตรวจสอบว่าเกิดใหม่และเป็นตัวละครที่ถูกต้องหรือยัง
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if char and hum and hum.Health > 0 then
                if isCharacterMatching(targetChar) then
                    print("🎉 Character successfully switched to: " .. targetChar)
                    notify("Switched to " .. targetChar .. " successfully!")
                    task.wait(0.8)
                    return true
                end
            end

            task.wait(0.25) -- Delay 0.25 วินาที เพื่อป้องกันบัคและไม่ให้เกมแลค
        end

        task.wait(0.5)
    end

    return isCharacterMatching(targetChar)
end

local runId = 0
local u9 = false
local isJoiningVIP = false

local function joinPrivateServer(customCode)
    if isJoiningVIP then return end
    local code = customCode or t1.PRIVATE_SERVER
    if not code or code == "" or code == "Enter PS Code (Optional)" then
        code = "JblH87"
    end

    isJoiningVIP = true
    notify("Joining VIP server (" .. code .. ")...")
    print("🚀 Teleporting to VIP server: " .. code)

    task.spawn(function()
        local PS = ReplicatedStorage:WaitForChild("PS", 15) or ReplicatedStorage:FindFirstChild("PS")
        if PS then
            for _ = 1, 10 do
                pcall(function()
                    PS:FireServer("join", code)
                end)
                task.wait(2)
                if not isJoiningVIP then break end
            end
        else
            warn("❌ ReplicatedStorage.PS not found!")
        end
        isJoiningVIP = false
    end)
end

local function u11(p2)
    local num = tonumber(p2)
    if num then
        return "Q" .. tostring(num)
    end
    return tostring(p2)
end

local function v12()
    runId += 1
    local currentRun = runId
    local function running()
        return u9 and runId == currentRun
    end
    local function v10(seconds)
        local deadline = os.clock() + seconds
        while running() and os.clock() < deadline do task.wait(0.1) end
    end

    local v51 = t1.MAIN_USERNAME == ""
    if not v51 then
        v51 = t1.MAIN_USERNAME == "EnterMainUsernameHere"
    end

    if v51 then
        warn("⚠️ Auto Farm aborted: Please set your MAIN_USERNAME in the UI!")
        notify("Set Main Username before starting.")
        return
    end

    if string.lower(LocalPlayer.Name) == string.lower(t1.MAIN_USERNAME) then
        print("🟢 MAIN account detected and active: " .. LocalPlayer.Name)
        notify("Main detected. Loading quest data...")
        task.spawn(function()
            if not game:IsLoaded() then
                game.Loaded:Wait()
            end

            if not running() then
                return
            end

            local QuestStuff = ReplicatedStorage:WaitForChild("QuestStuff", 999)
            local ReplicatedStats = LocalPlayer:WaitForChild("ReplicatedStats", 10)
            local v85 = ReplicatedStats and ReplicatedStats:WaitForChild("DailyQuest", 10)

            if not QuestStuff or not v85 then
                return
            end

            local v86 = false
            local u87 = false
            local s1 = "Kills"
            local n2 = 3
            local s2 = ""
            local v91 = u11(t1.SELECTED_QUEST_SLOT)
            local v92 = u11(t1.GLITCH_QUEST_1)
            local v93 = u11(t1.GLITCH_QUEST_2)
            local n3 = 0

            while true do
                local v95 = not v86
                if v95 then
                    v95 = running() and n3 < 5
                end
                if not v95 then
                    break
                end

                n3 += 1

                if t1.AUTO_REFRESH_QUESTS and n3 == 1 then
                    notify("Refreshing quest slots...")
                    for _ = 1, 4 do
                        if not running() then
                            return
                        end
                        QuestStuff:FireServer("Take", v92)
                        task.wait(0.05)
                        QuestStuff:FireServer("Take", v93)
                        task.wait(0.05)
                    end
                    task.wait(0.1)
                end

                if not running() then
                    return
                end

                notify("Taking quest " .. v91 .. " (attempt " .. n3 .. ")")
                QuestStuff:FireServer("Take", v91)
                task.wait(0.4)
                QuestStuff:FireServer("Close")

                local s3 = ""
                for _ = 1, 25 do
                    if not running() then
                        return
                    end
                    local Value = v85.Value
                    if Value then
                        Value = v85.Value ~= ""
                    end
                    if Value then
                        s3 = tostring(v85.Value)
                        break
                    end
                    task.wait(0.1)
                end

                if s3 ~= "" then
                    local ok, result = pcall(function()
                        return HttpService:JSONDecode(s3)
                    end)

                    if ok and type(result) == "table" then
                        local v102 = result[v91]
                        if v102 and v102.Requirements then
                            s2 = v102.Requirements.Character or ""
                            if v102.Requirements.Combo then
                                s1 = "Combo"
                                n2 = tonumber(v102.Requirements.Combo) or 0
                            elseif v102.Requirements.Damage then
                                s1 = "Damage"
                                n2 = tonumber(v102.Requirements.Damage) or 0
                            elseif v102.Requirements.Points then
                                s1 = "Points"
                                n2 = tonumber(v102.Requirements.Points) or 0
                            elseif v102.Requirements.Kills then
                                s1 = "Kills"
                                n2 = tonumber(v102.Requirements.Kills) or 0
                            end

                            local Description = v102.Description
                            if Description then
                                Description = v102.Description:lower():find("mode") or v102.Description:lower():find("awaken")
                            end
                            if Description then
                                s1 = "ModeKills"
                            end

                            v86 = true
                        end
                    end
                end

                if not v86 then
                    print("⚠️ Quest data sync delayed, retrying... (Attempt " .. n3 .. "/5)")
                    task.wait(0.8)
                end
            end

            if not v86 or not running() then
                warn("❌ Failed to load quest data after multiple attempts.")
                if running() then notify("Quest data unavailable. Stop and retry.") end
                return
            end

            print(string.format("🎯 Target Loaded | Mode: %s | Target: %d | Character: '%s'", s1, n2, s2))
            notify("Quest loaded: " .. s1 .. " | Target: " .. n2)

            -- ตรวจสอบผู้เล่น ถ้ามีไม่ถึง 2 คน ให้รอและวาปเข้า VIP
            if #Players:GetPlayers() < t1.REQUIRED_PLAYERS and running() then
                notify(string.format("Waiting for %d players... (%ds before VIP)", t1.REQUIRED_PLAYERS, t1.PRIVATE_SERVER_DELAY))
                local waitElapsed = 0
                while running() and #Players:GetPlayers() < t1.REQUIRED_PLAYERS do
                    task.wait(0.5)
                    waitElapsed += 0.5
                    if waitElapsed >= t1.PRIVATE_SERVER_DELAY and not isJoiningVIP then
                        joinPrivateServer()
                        break
                    end
                end
            end

            -- ตรวจสอบเมื่อมีผู้เล่นครบ >= 2 คนขึ้นไป
            if v86 and #Players:GetPlayers() >= t1.REQUIRED_PLAYERS and running() then
                
                -- ตรวจสอบและสลับตัวละครให้ตรงกับเควสต์ก่อนเริ่มสู้
                if s2 and s2 ~= "" and running() then
                    if not isCharacterMatching(s2) then
                        ensureCharacter(s2, running)
                    else
                        print("✅ Current character already matches quest: " .. s2)
                    end
                end

                if not running() then return end

                -- เริ่มระบบตรวจสอบ Leaderstats
                task.spawn(function()
                    local leaderstats = LocalPlayer:WaitForChild("leaderstats", 10)
                    local u117 = (leaderstats and leaderstats:FindFirstChild("Kills") and tonumber(leaderstats.Kills.Value)) or 0
                    local v120 = (leaderstats and leaderstats:FindFirstChild("Damage") and tonumber(leaderstats.Damage.Value)) or 0
                    local v121 = (leaderstats and leaderstats:FindFirstChild("Points") and tonumber(leaderstats.Points.Value)) or 0
                    local v123 = (s1 == "Kills" or s1 == "ModeKills")
                    local u124 = u117 + (v123 and n2 or 0)
                    local v125 = v120 + (s1 == "Damage" and n2 or 0)
                    local v128 = v121 + ((s1 == "Points" and n2) or 0)

                    local respawnConnection = LocalPlayer.CharacterAdded:Connect(function()
                        if not u87 and leaderstats and running() then
                            task.wait(0.5)
                            local Kills = leaderstats:FindFirstChild("Kills")
                            u117 = (Kills and tonumber(Kills.Value)) or 0
                            u124 = u117 + ((s1 == "Kills" or s1 == "ModeKills") and n2 or 0)
                        end
                    end)

                    while running() and not u87 do
                        local v129 = false
                        local leaderstats2 = LocalPlayer:FindFirstChild("leaderstats")

                        if leaderstats2 and n2 > 0 then
                            if s1 == "Kills" or s1 == "ModeKills" then
                                local Kills = leaderstats2:FindFirstChild("Kills")
                                local v134 = (Kills and tonumber(Kills.Value)) or 0
                                v129 = (u124 <= v134) and (v134 > u117)
                            elseif s1 == "Damage" then
                                local Damage = leaderstats2:FindFirstChild("Damage")
                                local v136 = (Damage and tonumber(Damage.Value)) or 0
                                v129 = (v125 <= v136) and (v120 < v136)
                            elseif s1 == "Points" then
                                local Points = leaderstats2:FindFirstChild("Points")
                                local v138 = (Points and tonumber(Points.Value)) or 0
                                v129 = (v128 <= v138) and (v121 < v138)
                            end
                        end

                        if v129 and not u87 then
                            u87 = true
                            print("✅ Quest goal achieved via Leaderstats!")
                            notify("Target reached. Combat stopped; waiting to return...")
                            v10((math.max(0, 1 + t1.TP_DELAY)))

                            if running() then
                                local Train = ReplicatedStorage:WaitForChild("Train", 10)
                                if Train then
                                    notify("Returning to Training...")
                                    Train:FireServer()
                                end
                            end
                            break
                        end
                        task.wait(0.2)
                    end
                    respawnConnection:Disconnect()
                end)

                -- เริ่มระบบ Combat Loop
                task.spawn(function()
                    print("⚔️ Combat Loop ACTIVE for mode: " .. s1)

                    local t2 = {
						Enum.KeyCode.One,
						Enum.KeyCode.Two,
						Enum.KeyCode.Three,
						Enum.KeyCode.Four
					}
                    notify("Farming started. M1: " .. tostring(t1.AUTO_M1) .. " | Skills: " .. tostring(t1.AUTO_SKILLS))
                    local targetPlayer
                    local lastTeleport = 0
                    local waitingForTarget = false

                    local function aliveRoot(player)
                        local character = player and player.Character
                        local humanoid = character and character:FindFirstChildOfClass("Humanoid")
                        local root = character and character:FindFirstChild("HumanoidRootPart")
                        if humanoid and humanoid.Health > 0 and root then return root end
                    end

                    local n4 = 1
                    local v142 = n4

                    while running() and not u87 do
                        if #Players:GetPlayers() < t1.REQUIRED_PLAYERS then
                            if not waitingForTarget then
                                waitingForTarget = true
                                notify("Combat paused: < 2 players. Rejoining VIP in " .. t1.PRIVATE_SERVER_DELAY .. "s...")
                                task.spawn(function()
                                    task.wait(t1.PRIVATE_SERVER_DELAY)
                                    if running() and #Players:GetPlayers() < t1.REQUIRED_PLAYERS then
                                        joinPrivateServer()
                                    end
                                end)
                            end
                            task.wait(0.5)
                            continue
                        end

                        if t1.AUTO_TP_PLAYERS then
                            local ownRoot = aliveRoot(LocalPlayer)
                            local targetRoot = targetPlayer and targetPlayer.Parent == Players and aliveRoot(targetPlayer)
                            if not targetRoot and ownRoot then
                                local closest = math.huge
                                targetPlayer = nil
                                for _, player in ipairs(Players:GetPlayers()) do
                                    if player ~= LocalPlayer then
                                        local root = aliveRoot(player)
                                        if root then
                                            local distance = (root.Position - ownRoot.Position).Magnitude
                                            if distance < closest then
                                                closest = distance
                                                targetPlayer = player
                                                targetRoot = root
                                            end
                                        end
                                    end
                                end
                                if targetPlayer then notify("Teleporting to: " .. targetPlayer.Name) end
                            end

                            if not ownRoot or not targetRoot then
                                if not waitingForTarget then
                                    waitingForTarget = true
                                    notify("Waiting for a living character or target...")
                                end
                                task.wait(0.2)
                                continue
                            end

                            if os.clock() - lastTeleport >= t1.MAIN_TP_INTERVAL then
                                local position = (targetRoot.CFrame * CFrame.new(0, 0, t1.MAIN_TP_DISTANCE)).Position
                                ownRoot.CFrame = CFrame.lookAt(position, targetRoot.Position)
                                ownRoot.AssemblyLinearVelocity = Vector3.zero
                                lastTeleport = os.clock()
                            end
                        end

                        if waitingForTarget then
                            waitingForTarget = false
                            notify("Target ready. Resuming combat...")
                        end

                        if s1 == "ModeKills" then
                            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.G, false, game)
                            task.wait(0.02)
                            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.G, false, game)
                        end

                        if s1 == "Combo" then
                            local Stats = LocalPlayer:FindFirstChild("Stats")
                            local v144 = Stats and Stats:FindFirstChild("Combo")
                            local v145 = (v144 and tonumber(v144.Value)) or 0

                            if v145 >= n2 then
                                print("🛑 Combo target reached (" .. v145 .. "). Holding for drop...")
                                notify("Combo target reached. Holding for 6 seconds...")
                                v10(6)

                                if not u87 and running() then
                                    u87 = true
                                    v10((math.max(0, 1 + t1.TP_DELAY)))
                                    if not running() then return end
                                    local Train = ReplicatedStorage:WaitForChild("Train", 10)
                                    if Train then
                                        notify("Returning to Training...")
                                        Train:FireServer()
                                    end
                                    return
                                end
                            end
                        end

                        if not u87 and running() then
                            if t1.AUTO_SKILLS then
                                local v147 = false
                                for i = 1, 4 do
                                    if not running() or u87 then break end
                                    local v149 = (v142 + i - 2) % 4 + 1
                                    if ({
										function() return t1.SKILL_1 end,
										function() return t1.SKILL_2 end,
										function() return t1.SKILL_3 end,
										function() return t1.SKILL_4 end
									})[v149]() then
                                        VirtualInputManager:SendKeyEvent(true, t2[v149], false, game)
                                        task.wait(0.05)
                                        VirtualInputManager:SendKeyEvent(false, t2[v149], false, game)
                                        v142 = v149 % 4 + 1
                                        v147 = true
                                        task.wait(t1.SKILL_DELAY)
                                        break
                                    end
                                end
                                if not v147 then task.wait(0.2) end
                            end

                            if not u87 and running() and t1.AUTO_M1 then
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                                task.wait(t1.M1_HOLD_DURATION)
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                            end

                            task.wait(t1.M1_CLICK_DELAY)
                        end
                    end
                end)
            end
        end)
        return
    end

    print("🔵 ALT detected and active: " .. LocalPlayer.Name)
    notify("Alt detected. Following Main: " .. t1.MAIN_USERNAME)
    task.spawn(function()
        pcall(function()
            if getconnections then
                for _, v in pairs(getconnections(LocalPlayer.Idled)) do
                    pcall(function() v:Disable() end)
                    pcall(function() v:Disconnect() end)
                end
            end

            local v157 = getgenv and getgenv() or _G
            if v157._AIO_AntiAfkConnection then
                pcall(function() v157._AIO_AntiAfkConnection:Disconnect() end)
            end

            v157._AIO_AntiAfkConnection = LocalPlayer.Idled:Connect(function()
                local VirtualInputManager2 = Instance.new("VirtualInputManager")
                VirtualInputManager2:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                VirtualInputManager2:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                VirtualInputManager2:Destroy()
            end)
        end)
    end)

    local function v52(p3)
        if p3 and p3.Character then
            return p3.Character:FindFirstChild("HumanoidRootPart")
        end
        return nil
    end

    task.spawn(function()
        while running() do
            local t1MAIN_USERNAME = Players:FindFirstChild(t1.MAIN_USERNAME)
            if t1MAIN_USERNAME and t1MAIN_USERNAME ~= LocalPlayer then
                local v108 = v52(LocalPlayer)
                local v109 = v52(t1MAIN_USERNAME)
                if v108 and v109 then
                    v108.CFrame = v109.CFrame * CFrame.new(t1.ALT_TP_OFFSET)
                    v108.AssemblyLinearVelocity = Vector3.zero
                end
            end
            v10(t1.ALT_TP_INTERVAL)
        end
    end)
end

local ok, result = pcall(function()
    return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
end)

if not ok or not result then
    warn("⚠ Failed to load WindUI! Running script without GUI interface.")
    if t1.ENGINE_ENABLED then
        u9 = true
        task.spawn(v12)
    end
    return
end

local v19 = result:CreateWindow({
	Title = "ABA Auto Quest Farm",
	Icon = "swords",
	Author = "Standalone",
	Folder = "ABAQuestFarm",
	Size = UDim2.fromOffset(580, 460),
	Transparent = true,
	Theme = "Dark",
	SideBarWidth = 160
})

local v20 = v19:Tab({ Title = "Main Setup", Icon = "home" })
local v21 = v19:Tab({ Title = "Combat Settings", Icon = "sword" })

local function saveConfig()
    pcall(function()
        if not isfolder or not writefile then return end
        if not isfolder("abaquestfarm") then makefolder("abaquestfarm") end
        writefile("abaquestfarm/config.json", HttpService:JSONEncode(t1))
    end)
end

v20:Input({
	Title = "Main Username",
	Desc = "Enter the exact username of your Main account",
	Value = t1.MAIN_USERNAME,
	Placeholder = "Username...",
	Callback = function(p4)
        t1.MAIN_USERNAME = p4
        saveConfig()
    end
})

v20:Input({
	Title = "Quest Slot Number",
	Desc = "Target quest slot number (1 to 6)",
	Value = tostring(t1.SELECTED_QUEST_SLOT),
	Placeholder = "4",
	Callback = function(p5)
        local num = tonumber(p5)
        if num then
            t1.SELECTED_QUEST_SLOT = num
            saveConfig()
        end
    end
})

v20:Toggle({
	Title = "Auto Refresh Quests",
	Desc = "Enable instant quest refresh glitch",
	Value = t1.AUTO_REFRESH_QUESTS,
	Callback = function(p6)
        t1.AUTO_REFRESH_QUESTS = p6
        saveConfig()
    end
})

v20:Input({
	Title = "Glitch Quest Slot 1 Number",
	Desc = "First quest slot number for glitching (e.g. 1)",
	Value = tostring(t1.GLITCH_QUEST_1),
	Placeholder = "1",
	Callback = function(p7)
        local num = tonumber(p7)
        if num then
            t1.GLITCH_QUEST_1 = num
            saveConfig()
        end
    end
})

v20:Input({
	Title = "Glitch Quest Slot 2 Number",
	Desc = "Second quest slot number for glitching (e.g. 2)",
	Value = tostring(t1.GLITCH_QUEST_2),
	Placeholder = "2",
	Callback = function(p8)
        local num = tonumber(p8)
        if num then
            t1.GLITCH_QUEST_2 = num
            saveConfig()
        end
    end
})

v20:Input({
	Title = "Private Server Code",
	Desc = "Optional private server code to join automatically",
	Value = t1.PRIVATE_SERVER,
	Placeholder = "Code here...",
	Callback = function(p9)
        t1.PRIVATE_SERVER = (p9 and p9 ~= "") and p9 or "JblH87"
        saveConfig()
    end
})

v21:Toggle({
    Title = "Auto TP to Players",
    Desc = "Follow a living target during farming",
    Value = t1.AUTO_TP_PLAYERS,
    Callback = function(value)
        t1.AUTO_TP_PLAYERS = value
        saveConfig()
    end
})

local v30 = v20:Toggle({
	Title = "Start / Stop Engine",
	Desc = "Unified toggle to start and stop the farming engine",
	Value = t1.ENGINE_ENABLED,
	Callback = function(p10)
        t1.ENGINE_ENABLED = p10
        u9 = p10
        saveConfig()

        if p10 then
            print("🚀 Farming Engine STARTED via UI toggle.")
            notify("Engine started.")
            task.spawn(v12)
            return
        end

        print("🛑 Farming Engine STOPPED via UI toggle.")
        runId += 1
        notify("Engine stopped.")
    end
})

v20:Toggle({
	Title = "Auto Hide UI on Start",
	Desc = "Hides UI automatically when the script executes",
	Value = t1.AUTO_HIDE_UI,
	Callback = function(p11)
        t1.AUTO_HIDE_UI = p11
        saveConfig()
    end
})

v20:Keybind({
	Title = "Menu Toggle Key",
	Desc = "Key to open or close the UI menu",
	Value = t1.TOGGLE_UI_KEY,
	Callback = function(p12)
        t1.TOGGLE_UI_KEY = (typeof(p12) == "EnumItem") and p12.Name or tostring(p12)
        saveConfig()
    end
})

v20:Keybind({
	Title = "Engine Start/Stop Key",
	Desc = "Key to start or stop the farming engine",
	Value = t1.TOGGLE_ENGINE_KEY,
	Callback = function(p13)
        t1.TOGGLE_ENGINE_KEY = (typeof(p13) == "EnumItem") and p13.Name or tostring(p13)
        saveConfig()
    end
})

v21:Toggle({
	Title = "Auto M1 Attacks",
	Desc = "Enable auto attacking",
	Value = t1.AUTO_M1,
	Callback = function(p14)
        t1.AUTO_M1 = p14
        saveConfig()
    end
})

v21:Toggle({
	Title = "Auto Skills",
	Desc = "Enable automatic skill spamming",
	Value = t1.AUTO_SKILLS,
	Callback = function(p15)
        t1.AUTO_SKILLS = p15
        saveConfig()
    end
})

v21:Toggle({
	Title = "Use Skill 1",
	Value = t1.SKILL_1,
	Callback = function(p16)
        t1.SKILL_1 = p16
        saveConfig()
    end
})

v21:Toggle({
	Title = "Use Skill 2",
	Value = t1.SKILL_2,
	Callback = function(p17)
        t1.SKILL_2 = p17
        saveConfig()
    end
})

v21:Toggle({
	Title = "Use Skill 3",
	Value = t1.SKILL_3,
	Callback = function(p18)
        t1.SKILL_3 = p18
        saveConfig()
    end
})

v21:Toggle({
	Title = "Use Skill 4",
	Value = t1.SKILL_4,
	Callback = function(p19)
        t1.SKILL_4 = p19
        saveConfig()
    end
})

local function v42()
    if not v19 then return end
    if type(v19.Toggle) == "function" then
        pcall(function() v19:Toggle() end)
        return
    end

    local v74 = (gethui and gethui()) or game:GetService("CoreGui") or LocalPlayer:FindFirstChildOfClass("PlayerGui")
    if v74 then
        for _, v77 in ipairs(v74:GetChildren()) do
            if v77:IsA("ScreenGui") and (v77.Name:find("WindUI") or v77.Name == "ABAQuestFarm") then
                v77.Enabled = not v77.Enabled
            end
        end
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    local function v81(p20)
        return p20 and input.KeyCode ~= Enum.KeyCode.Unknown and p20 == input.KeyCode.Name
    end

    if v81(t1.TOGGLE_ENGINE_KEY) then
        t1.ENGINE_ENABLED = not t1.ENGINE_ENABLED
        u9 = t1.ENGINE_ENABLED
        saveConfig()

        if v30 and type(v30.Set) == "function" then
            v30:Set(t1.ENGINE_ENABLED)
            return
        end

        if u9 then
            print("🟢 Farming Engine STARTED via hotkey [" .. tostring(t1.TOGGLE_ENGINE_KEY) .. "]")
            task.spawn(v12)
            return
        end

        print("🔴 Farming Engine STOPPED via hotkey [" .. tostring(t1.TOGGLE_ENGINE_KEY) .. "]")
        runId += 1
        notify("Engine stopped.")
        return
    end

    if v81(t1.TOGGLE_UI_KEY) then
        v42()
        print("👁 UI Visibility toggled via hotkey [" .. tostring(t1.TOGGLE_UI_KEY) .. "]")
    end
end)

if t1.AUTO_HIDE_UI then
    task.spawn(function()
        task.wait(0.6)
        v42()
        print("👁 UI automatically hidden on launch.")
    end)
end

notify("Script loaded. R: Engine | L: UI")

if t1.ENGINE_ENABLED then
    u9 = true
    task.spawn(v12)
end
