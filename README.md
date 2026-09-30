-- ==========================================
-- 📌 Version: V.8.7.2 - ABA Quest Farm (Auto Character Switch Fix & Current Character Noti)
-- ==========================================

-- ==========================================
-- 🔒 User Whitelist Check (เช็คชื่อก่อนรัน)
-- ==========================================
local ALLOWED_USERS = {
    "Bunowaiau359", -- ไอดีหลัก[cite: 2]
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
	MAIN_USERNAME = "Bunowaiau359",
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
-- 🛠️ ระบบตรวจสอบตัวละครปัจจุบัน & สลับตัวละครแบบ Spam Remote
-- ==========================================
local function getInput()
    local bp = LocalPlayer:FindFirstChild("Backpack")
    local c = LocalPlayer.Character
    return (bp and bp:FindFirstChild("Input")) or (c and c:FindFirstChild("Input")) or LocalPlayer:FindFirstChild("Input")
end

local function getChooseRemote()
    local bp = LocalPlayer:FindFirstChild("Backpack")
    local c = LocalPlayer.Character
    local st = (bp and bp:FindFirstChild("ServerTraits")) or (c and c:FindFirstChild("ServerTraits")) or LocalPlayer:FindFirstChild("ServerTraits")
    return st and st:FindFirstChild("Choose")
end

-- ตรวจสอบหาชื่อตัวละครที่กำลังใช้งานอยู่จริง ๆ
local function getCurrentCharacter()
    local char = LocalPlayer.Character
    if char then
        for _, name in ipairs({"Character", "CharacterName", "Char", "CurrentCharacter"}) do
            local v = char:FindFirstChild(name)
            if v and v:IsA("StringValue") and v.Value ~= "" then return v.Value end
            local attr = char:GetAttribute(name)
            if attr and tostring(attr) ~= "" then return tostring(attr) end
        end
    end

    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        for _, name in ipairs({"Character", "CharacterName", "Char", "CurrentCharacter"}) do
            local v = bp:FindFirstChild(name)
            if v and v:IsA("StringValue") and v.Value ~= "" then return v.Value end
        end
    end

    local rs = LocalPlayer:FindFirstChild("ReplicatedStats")
    if rs then
        for _, name in ipairs({"Character", "CharacterName", "Char", "SelectedCharacter"}) do
            local v = rs:FindFirstChild(name)
            if v and v:IsA("StringValue") and v.Value ~= "" then return v.Value end
        end
    end

    -- ตรวจสอบจากชื่อ Tools สกิลในตัว
    if bp then
        local tools = {}
        for _, t in ipairs(bp:GetChildren()) do
            if t:IsA("Tool") and t.Name ~= "Input" then
                table.insert(tools, t.Name)
            end
        end
        if char then
            for _, t in ipairs(char:GetChildren()) do
                if t:IsA("Tool") and t.Name ~= "Input" then
                    table.insert(tools, t.Name)
                end
            end
        end
        if #tools > 0 then
            return table.concat(tools, ", ")
        end
    end

    return "Unknown"
end

-- เช็คว่าตัวละครปัจจุบันตรงกับเป้าหมายเควสต์หรือไม่
local function isCharacterMatching(targetChar)
    if not targetChar or targetChar == "" then return true end
    local current = getCurrentCharacter()
    if current == "Unknown" then return false end

    local cClean = string.lower(string.gsub(current, "%s+", ""))
    local tClean = string.lower(string.gsub(targetChar, "%s+", ""))
    return string.find(cClean, tClean, 1, true) ~= nil or string.find(tClean, cClean, 1, true) ~= nil
end

-- ฟังก์ชันส่งคำสั่งเลือกตัวละครผ่าน Remote ทั้งหมด
local function fireSelectRemote(charName)
    if not charName or charName == "" then return end

    -- 1. ServerTraits.Choose[cite: 2]
    pcall(function()
        local choose = getChooseRemote()
        if choose then
            choose:FireServer(charName)
            task.wait(0.04)
            choose:FireServer("PLAY")[cite: 2]
        end
    end)

    -- 2. Input Remote[cite: 1]
    pcall(function()
        local inp = getInput()
        if inp then
            inp:FireServer("CharacterButton", charName)[cite: 1]
            task.wait(0.04)
            inp:FireServer("ClickPlay")[cite: 1]
        end
    end)
end

-- ฟังก์ชันจำลองการกดปุ่มบนหน้าต่าง UI เลือกตัวละคร
local function tryClickCharacterGui(charName)
    local pg = LocalPlayer:FindFirstChild("PlayerGui")
    if not pg then return end

    pcall(function()
        for _, gui in ipairs(pg:GetChildren()) do
            if gui:IsA("ScreenGui") then
                for _, btn in ipairs(gui:GetDescendants()) do
                    if btn:IsA("TextButton") or btn:IsA("ImageButton") then
                        local bName = string.lower(btn.Name)
                        local bText = btn:IsA("TextButton") and string.lower(btn.Text) or ""
                        local target = string.lower(charName)

                        if bName == target or bText == target or string.find(bName, target, 1, true) or string.find(bText, target, 1, true) then
                            if getconnections then
                                for _, conn in pairs(getconnections(btn.MouseButton1Click)) do conn:Fire() end
                                for _, conn in pairs(getconnections(btn.MouseButton1Down)) do conn:Fire() end
                            end
                        end

                        if bName == "play" or bText == "play" or bName == "clickplay" then
                            if getconnections then
                                for _, conn in pairs(getconnections(btn.MouseButton1Click)) do conn:Fire() end
                            end
                        end
                    end
                end
            end
        end
    end)
end

-- ฟังก์ชันสแปม Remote เปลี่ยนตัว + รีเซ็ต วนลูปจนกว่าตัวละครจะตรงเป้าหมาย
local function forceSwitchCharacterWithSpam(targetChar, runningCheck)
    if not targetChar or targetChar == "" then return end

    local current = getCurrentCharacter()
    notify("Current: [" .. current .. "] ➔ Target: [" .. targetChar .. "]")
    print(string.format("🔍 [ABA Check] Current: '%s' | Quest Target: '%s'", current, targetChar))

    if isCharacterMatching(targetChar) then
        print("✅ ตัวละครตรงกับเควสต์อยู่แล้ว ไม่จำเป็นต้องเปลี่ยน")
        return
    end

    local maxAttempts = 8
    local attempt = 0

    while (runningCheck and runningCheck() or true) and attempt < maxAttempts do
        attempt += 1
        current = getCurrentCharacter()
        notify(string.format("Switching (%d/%d): [%s] ➔ [%s]", attempt, maxAttempts, current, targetChar))
        print(string.format("🔄 [Attempt %d/%d] Current: '%s' -> Switching to '%s'", attempt, maxAttempts, current, targetChar))

        -- 1. กดปุ่มเปิดหน้าเลือกตัวละคร (M Key / Menu Remote)
        pcall(function()
            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.M, false, game)
            task.wait(0.04)
            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.M, false, game)

            local inp = getInput()
            if inp then
                inp:FireServer("Menu")
                task.wait(0.04)
                inp:FireServer("CharacterSelect")
            end
        end)

        tryClickCharacterGui(targetChar)
        fireSelectRemote(targetChar)
        task.wait(0.1)

        -- 2. สั่ง Reset ตัวละคร (ฆ่าตัวตาย + ส่ง Loaded Remote)[cite: 2]
        pcall(function()
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                hum.Health = 0[cite: 2]
            end
            local loaded = ReplicatedStorage:FindFirstChild("Loaded")
            if loaded then
                loaded:FireServer()[cite: 2]
            end
        end)

        -- 3. ลูปสแปมส่ง Remote ระหว่างเกิดใหม่ มี Delay 0.25 วินาที กันบัคและแลค
        local cycleDeadline = os.clock() + 6
        while (runningCheck and runningCheck() or true) and os.clock() < cycleDeadline do
            fireSelectRemote(targetChar)
            tryClickCharacterGui(targetChar)

            -- กดยืนยัน Respawn Done ถ้ามี GUI[cite: 2]
            pcall(function()
                local pg = LocalPlayer:FindFirstChild("PlayerGui")
                local r = pg and pg:FindFirstChild("Respawning")[cite: 2]
                if r and r:FindFirstChild("Done") then
                    r.Done:FireServer()[cite: 2]
                end
            end)

            task.wait(0.25) -- หน่วงเวลา 0.25 วิ กันเซิร์ฟเวอร์ดีเลย์และป้องกันการแลค

            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if char and hum and hum.Health > 0 and (os.clock() - (cycleDeadline - 6) > 2) then
                if isCharacterMatching(targetChar) then
                    break
                end
            end
        end

        -- ยืนยันปิด ForceField[cite: 2]
        pcall(function()
            local inp = getInput()
            if inp then inp:FireServer("ForceFieldOff") end[cite: 2]
        end)

        -- ตรวจสอบว่าเปลี่ยนสำเร็จหรือยัง
        task.wait(0.5)
        current = getCurrentCharacter()
        if isCharacterMatching(targetChar) then
            print("🎉 [Success] Switched successfully to: " .. targetChar)
            notify("✅ Switched! Now playing as: " .. targetChar)
            return
        end
    end

    local finalCurrent = getCurrentCharacter()
    notify("Finished switch check. Current: [" .. finalCurrent .. "]")
    print("⚠️ Switch loop ended. Current detected: " .. finalCurrent)
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
        v51 = t1.MAIN_USERNAME == "EnterMainUsernameHere"[cite: 1]
    end

    if v51 then
        warn("⚠️ Auto Farm aborted: Please set your MAIN_USERNAME in the UI!")[cite: 1]
        notify("Set Main Username before starting.")[cite: 1]
        return
    end

    if string.lower(LocalPlayer.Name) == string.lower(t1.MAIN_USERNAME) then
        print("🟢 MAIN account detected and active: " .. LocalPlayer.Name)[cite: 1]
        notify("Main detected. Loading quest data...")[cite: 1]
        task.spawn(function()
            if not game:IsLoaded() then
                game.Loaded:Wait()[cite: 1]
            end

            if not running() then
                return
            end

            local QuestStuff = ReplicatedStorage:WaitForChild("QuestStuff", 999)[cite: 1]
            local ReplicatedStats = LocalPlayer:WaitForChild("ReplicatedStats", 10)[cite: 1]
            local v85 = ReplicatedStats and ReplicatedStats:WaitForChild("DailyQuest", 10)[cite: 1]

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
                    v95 = running() and n3 < 5[cite: 1]
                end
                if not v95 then
                    break
                end

                n3 += 1

                if t1.AUTO_REFRESH_QUESTS and n3 == 1 then
                    notify("Refreshing quest slots...")[cite: 1]
                    for _ = 1, 4 do
                        if not running() then
                            return
                        end
                        QuestStuff:FireServer("Take", v92)[cite: 1]
                        task.wait(0.05)[cite: 1]
                        QuestStuff:FireServer("Take", v93)[cite: 1]
                        task.wait(0.05)[cite: 1]
                    end
                    task.wait(0.1)[cite: 1]
                end

                if not running() then
                    return
                end

                notify("Taking quest " .. v91 .. " (attempt " .. n3 .. ")")[cite: 1]
                QuestStuff:FireServer("Take", v91)[cite: 1]
                task.wait(0.4)[cite: 1]
                QuestStuff:FireServer("Close")[cite: 1]

                local s3 = ""
                for _ = 1, 25 do
                    if not running() then
                        return
                    end
                    local Value = v85.Value[cite: 1]
                    if Value then
                        Value = v85.Value ~= ""[cite: 1]
                    end
                    if Value then
                        s3 = tostring(v85.Value)[cite: 1]
                        break
                    end
                    task.wait(0.1)[cite: 1]
                end

                if s3 ~= "" then
                    local ok, result = pcall(function()
                        return HttpService:JSONDecode(s3)[cite: 1]
                    end)

                    if ok and type(result) == "table" then
                        local v102 = result[v91][cite: 1]
                        if v102 and v102.Requirements then
                            s2 = v102.Requirements.Character or ""[cite: 1]
                            if v102.Requirements.Combo then
                                s1 = "Combo"[cite: 1]
                                n2 = tonumber(v102.Requirements.Combo) or 0[cite: 1]
                            elseif v102.Requirements.Damage then
                                s1 = "Damage"[cite: 1]
                                n2 = tonumber(v102.Requirements.Damage) or 0[cite: 1]
                            elseif v102.Requirements.Points then
                                s1 = "Points"[cite: 1]
                                n2 = tonumber(v102.Requirements.Points) or 0[cite: 1]
                            elseif v102.Requirements.Kills then
                                s1 = "Kills"[cite: 1]
                                n2 = tonumber(v102.Requirements.Kills) or 0[cite: 1]
                            end

                            local Description = v102.Description[cite: 1]
                            if Description then
                                Description = v102.Description:lower():find("mode") or v102.Description:lower():find("awaken")[cite: 1]
                            end
                            if Description then
                                s1 = "ModeKills"[cite: 1]
                            end

                            v86 = true[cite: 1]
                        end
                    end
                end

                if not v86 then
                    print("⚠️ Quest data sync delayed, retrying... (Attempt " .. n3 .. "/5)")[cite: 1]
                    task.wait(0.8)[cite: 1]
                end
            end

            if not v86 or not running() then
                warn("❌ Failed to load quest data after multiple attempts.")[cite: 1]
                if running() then notify("Quest data unavailable. Stop and retry.") end[cite: 1]
                return
            end

            print(string.format("🎯 Target Loaded | Mode: %s | Target: %d | Character: '%s'", s1, n2, s2))[cite: 1]
            notify("Quest loaded: " .. s1 .. " | Target: " .. n2)[cite: 1]

            -- ตรวจสอบจำนวนผู้เล่น ถ้าไม่ถึง 2 คน ให้รอหรือวาปเข้า VIP[cite: 1]
            if #Players:GetPlayers() < t1.REQUIRED_PLAYERS and running() then
                notify(string.format("Waiting for %d players... (%ds before VIP)", t1.REQUIRED_PLAYERS, t1.PRIVATE_SERVER_DELAY))[cite: 1]
                local waitElapsed = 0
                while running() and #Players:GetPlayers() < t1.REQUIRED_PLAYERS do
                    task.wait(0.5)[cite: 1]
                    waitElapsed += 0.5[cite: 1]
                    if waitElapsed >= t1.PRIVATE_SERVER_DELAY and not isJoiningVIP then
                        joinPrivateServer()[cite: 1]
                        break
                    end
                end
            end

            -- เมื่อมีผู้เล่นครบ >= 2 คนขึ้นไป
            if v86 and #Players:GetPlayers() >= t1.REQUIRED_PLAYERS and running() then
                
                -- ตรวจสอบและสลับตัวละครให้ตรงกับเควสต์
                if s2 and s2 ~= "" and running() then
                    local currentNow = getCurrentCharacter()
                    notify("Current: [" .. currentNow .. "] | Quest: [" .. s2 .. "]")
                    print(string.format("🔎 Pre-combat check: Current='%s' | Target='%s'", currentNow, s2))

                    if not isCharacterMatching(s2) then
                        forceSwitchCharacterWithSpam(s2, running)
                    else
                        print("✅ Current character already matches: " .. s2)
                        notify("Character matches quest: " .. s2)
                    end
                end

                if not running() then return end

                task.spawn(function()
                    local leaderstats = LocalPlayer:WaitForChild("leaderstats", 10)[cite: 1]
                    local u117 = (leaderstats and leaderstats:FindFirstChild("Kills") and tonumber(leaderstats.Kills.Value)) or 0[cite: 1]
                    local v120 = (leaderstats and leaderstats:FindFirstChild("Damage") and tonumber(leaderstats.Damage.Value)) or 0[cite: 1]
                    local v121 = (leaderstats and leaderstats:FindFirstChild("Points") and tonumber(leaderstats.Points.Value)) or 0[cite: 1]
                    local v123 = (s1 == "Kills" or s1 == "ModeKills")[cite: 1]
                    local u124 = u117 + (v123 and n2 or 0)[cite: 1]
                    local v125 = v120 + (s1 == "Damage" and n2 or 0)[cite: 1]
                    local v128 = v121 + ((s1 == "Points" and n2) or 0)[cite: 1]

                    local respawnConnection = LocalPlayer.CharacterAdded:Connect(function()
                        if not u87 and leaderstats and running() then
                            task.wait(0.5)[cite: 1]
                            local Kills = leaderstats:FindFirstChild("Kills")[cite: 1]
                            u117 = (Kills and tonumber(Kills.Value)) or 0[cite: 1]
                            u124 = u117 + ((s1 == "Kills" or s1 == "ModeKills") and n2 or 0)[cite: 1]
                        end
                    end)

                    while running() and not u87 do
                        local v129 = false[cite: 1]
                        local leaderstats2 = LocalPlayer:FindFirstChild("leaderstats")[cite: 1]

                        if leaderstats2 and n2 > 0 then
                            if s1 == "Kills" or s1 == "ModeKills" then
                                local Kills = leaderstats2:FindFirstChild("Kills")[cite: 1]
                                local v134 = (Kills and tonumber(Kills.Value)) or 0[cite: 1]
                                v129 = (u124 <= v134) and (v134 > u117)[cite: 1]
                            elseif s1 == "Damage" then
                                local Damage = leaderstats2:FindFirstChild("Damage")[cite: 1]
                                local v136 = (Damage and tonumber(Damage.Value)) or 0[cite: 1]
                                v129 = (v125 <= v136) and (v120 < v136)[cite: 1]
                            elseif s1 == "Points" then
                                local Points = leaderstats2:FindFirstChild("Points")[cite: 1]
                                local v138 = (Points and tonumber(Points.Value)) or 0[cite: 1]
                                v129 = (v128 <= v138) and (v121 < v138)[cite: 1]
                            end
                        end

                        if v129 and not u87 then
                            u87 = true[cite: 1]
                            print("✅ Quest goal achieved via Leaderstats!")[cite: 1]
                            notify("Target reached. Combat stopped; waiting to return...")[cite: 1]
                            v10((math.max(0, 1 + t1.TP_DELAY)))[cite: 1]

                            if running() then
                                local Train = ReplicatedStorage:WaitForChild("Train", 10)[cite: 1]
                                if Train then
                                    notify("Returning to Training...")[cite: 1]
                                    Train:FireServer()[cite: 1]
                                end
                            end
                            break
                        end
                        task.wait(0.2)[cite: 1]
                    end
                    respawnConnection:Disconnect()[cite: 1]
                end)

                task.spawn(function()
                    print("⚔️ Combat Loop ACTIVE for mode: " .. s1)[cite: 1]

                    local t2 = {
						Enum.KeyCode.One,
						Enum.KeyCode.Two,
						Enum.KeyCode.Three,
						Enum.KeyCode.Four
					}[cite: 1]
                    notify("Farming started. M1: " .. tostring(t1.AUTO_M1) .. " | Skills: " .. tostring(t1.AUTO_SKILLS))[cite: 1]
                    local targetPlayer[cite: 1]
                    local lastTeleport = 0[cite: 1]
                    local waitingForTarget = false[cite: 1]

                    local function aliveRoot(player)
                        local character = player and player.Character[cite: 1]
                        local humanoid = character and character:FindFirstChildOfClass("Humanoid")[cite: 1]
                        local root = character and character:FindFirstChild("HumanoidRootPart")[cite: 1]
                        if humanoid and humanoid.Health > 0 and root then return root end[cite: 1]
                    end

                    local n4 = 1[cite: 1]
                    local v142 = n4[cite: 1]

                    while running() and not u87 do
                        if #Players:GetPlayers() < t1.REQUIRED_PLAYERS then
                            if not waitingForTarget then
                                waitingForTarget = true[cite: 1]
                                notify("Combat paused: < 2 players. Rejoining VIP in " .. t1.PRIVATE_SERVER_DELAY .. "s...")[cite: 1]
                                task.spawn(function()
                                    task.wait(t1.PRIVATE_SERVER_DELAY)[cite: 1]
                                    if running() and #Players:GetPlayers() < t1.REQUIRED_PLAYERS then
                                        joinPrivateServer()[cite: 1]
                                    end
                                end)
                            end
                            task.wait(0.5)[cite: 1]
                            continue
                        end

                        if t1.AUTO_TP_PLAYERS then
                            local ownRoot = aliveRoot(LocalPlayer)[cite: 1]
                            local targetRoot = targetPlayer and targetPlayer.Parent == Players and aliveRoot(targetPlayer)[cite: 1]
                            if not targetRoot and ownRoot then
                                local closest = math.huge[cite: 1]
                                targetPlayer = nil[cite: 1]
                                for _, player in ipairs(Players:GetPlayers()) do
                                    if player ~= LocalPlayer then
                                        local root = aliveRoot(player)[cite: 1]
                                        if root then
                                            local distance = (root.Position - ownRoot.Position).Magnitude[cite: 1]
                                            if distance < closest then
                                                closest = distance[cite: 1]
                                                targetPlayer = player[cite: 1]
                                                targetRoot = root[cite: 1]
                                            end
                                        end
                                    end
                                end
                                if targetPlayer then notify("Teleporting to: " .. targetPlayer.Name) end[cite: 1]
                            end

                            if not ownRoot or not targetRoot then
                                if not waitingForTarget then
                                    waitingForTarget = true[cite: 1]
                                    notify("Waiting for a living character or target...")[cite: 1]
                                end
                                task.wait(0.2)[cite: 1]
                                continue
                            end

                            if os.clock() - lastTeleport >= t1.MAIN_TP_INTERVAL then
                                local position = (targetRoot.CFrame * CFrame.new(0, 0, t1.MAIN_TP_DISTANCE)).Position[cite: 1]
                                ownRoot.CFrame = CFrame.lookAt(position, targetRoot.Position)[cite: 1]
                                ownRoot.AssemblyLinearVelocity = Vector3.zero[cite: 1]
                                lastTeleport = os.clock()[cite: 1]
                            end
                        end

                        if waitingForTarget then
                            waitingForTarget = false[cite: 1]
                            notify("Target ready. Resuming combat...")[cite: 1]
                        end

                        if s1 == "ModeKills" then
                            VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.G, false, game)[cite: 1]
                            task.wait(0.02)[cite: 1]
                            VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.G, false, game)[cite: 1]
                        end

                        if s1 == "Combo" then
                            local Stats = LocalPlayer:FindFirstChild("Stats")[cite: 1]
                            local v144 = Stats and Stats:FindFirstChild("Combo")[cite: 1]
                            local v145 = (v144 and tonumber(v144.Value)) or 0[cite: 1]

                            if v145 >= n2 then
                                print("🛑 Combo target reached (" .. v145 .. "). Holding for drop...")[cite: 1]
                                notify("Combo target reached. Holding for 6 seconds...")[cite: 1]
                                v10(6)[cite: 1]

                                if not u87 and running() then
                                    u87 = true[cite: 1]
                                    v10((math.max(0, 1 + t1.TP_DELAY)))[cite: 1]
                                    if not running() then return end
                                    local Train = ReplicatedStorage:WaitForChild("Train", 10)[cite: 1]
                                    if Train then
                                        notify("Returning to Training...")[cite: 1]
                                        Train:FireServer()[cite: 1]
                                    end
                                    return
                                end
                            end
                        end

                        if not u87 and running() then
                            if t1.AUTO_SKILLS then
                                local v147 = false[cite: 1]
                                for i = 1, 4 do
                                    if not running() or u87 then break end[cite: 1]
                                    local v149 = (v142 + i - 2) % 4 + 1[cite: 1]
                                    if ({
										function() return t1.SKILL_1 end,
										function() return t1.SKILL_2 end,
										function() return t1.SKILL_3 end,
										function() return t1.SKILL_4 end
									})[v149]() then
                                        VirtualInputManager:SendKeyEvent(true, t2[v149], false, game)[cite: 1]
                                        task.wait(0.05)[cite: 1]
                                        VirtualInputManager:SendKeyEvent(false, t2[v149], false, game)[cite: 1]
                                        v142 = v149 % 4 + 1[cite: 1]
                                        v147 = true[cite: 1]
                                        task.wait(t1.SKILL_DELAY)[cite: 1]
                                        break
                                    end
                                end
                                if not v147 then task.wait(0.2) end[cite: 1]
                            end

                            if not u87 and running() and t1.AUTO_M1 then
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)[cite: 1]
                                task.wait(t1.M1_HOLD_DURATION)[cite: 1]
                                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)[cite: 1]
                            end

                            task.wait(t1.M1_CLICK_DELAY)[cite: 1]
                        end
                    end
                end)
            end
        end)
        return
    end

    print("🔵 ALT detected and active: " .. LocalPlayer.Name)[cite: 1]
    notify("Alt detected. Following Main: " .. t1.MAIN_USERNAME)[cite: 1]
    task.spawn(function()
        pcall(function()
            if getconnections then
                for _, v in pairs(getconnections(LocalPlayer.Idled)) do
                    pcall(function() v:Disable() end)[cite: 1]
                    pcall(function() v:Disconnect() end)[cite: 1]
                end
            end

            local v157 = getgenv and getgenv() or _G[cite: 1]
            if v157._AIO_AntiAfkConnection then
                pcall(function() v157._AIO_AntiAfkConnection:Disconnect() end)[cite: 1]
            end

            v157._AIO_AntiAfkConnection = LocalPlayer.Idled:Connect(function()
                local VirtualInputManager2 = Instance.new("VirtualInputManager")[cite: 1]
                VirtualInputManager2:SendMouseButtonEvent(0, 0, 0, true, game, 0)[cite: 1]
                VirtualInputManager2:SendMouseButtonEvent(0, 0, 0, false, game, 0)[cite: 1]
                VirtualInputManager2:Destroy()[cite: 1]
            end)
        end)
    end)

    local function v52(p3)
        if p3 and p3.Character then
            return p3.Character:FindFirstChild("HumanoidRootPart")[cite: 1]
        end
        return nil
    end

    task.spawn(function()
        while running() do
            local t1MAIN_USERNAME = Players:FindFirstChild(t1.MAIN_USERNAME)[cite: 1]
            if t1MAIN_USERNAME and t1MAIN_USERNAME ~= LocalPlayer then
                local v108 = v52(LocalPlayer)[cite: 1]
                local v109 = v52(t1MAIN_USERNAME)[cite: 1]
                if v108 and v109 then
                    v108.CFrame = v109.CFrame * CFrame.new(t1.ALT_TP_OFFSET)[cite: 1]
                    v108.AssemblyLinearVelocity = Vector3.zero[cite: 1]
                end
            end
            v10(t1.ALT_TP_INTERVAL)[cite: 1]
        end
    end)
end

local ok, result = pcall(function()
    return loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()[cite: 1]
end)

if not ok or not result then
    warn("⚠ Failed to load WindUI! Running script without GUI interface.")[cite: 1]
    if t1.ENGINE_ENABLED then
        u9 = true[cite: 1]
        task.spawn(v12)[cite: 1]
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
})[cite: 1]

local v20 = v19:Tab({ Title = "Main Setup", Icon = "home" })[cite: 1]
local v21 = v19:Tab({ Title = "Combat Settings", Icon = "sword" })[cite: 1]

local function saveConfig()
    pcall(function()
        if not isfolder or not writefile then return end[cite: 1]
        if not isfolder("abaquestfarm") then makefolder("abaquestfarm") end[cite: 1]
        writefile("abaquestfarm/config.json", HttpService:JSONEncode(t1))[cite: 1]
    end)
end

v20:Input({
	Title = "Main Username",
	Desc = "Enter the exact username of your Main account",
	Value = t1.MAIN_USERNAME,
	Placeholder = "Username...",
	Callback = function(p4)
        t1.MAIN_USERNAME = p4[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v20:Input({
	Title = "Quest Slot Number",
	Desc = "Target quest slot number (1 to 6)",
	Value = tostring(t1.SELECTED_QUEST_SLOT),
	Placeholder = "4",
	Callback = function(p5)
        local num = tonumber(p5)[cite: 1]
        if num then
            t1.SELECTED_QUEST_SLOT = num[cite: 1]
            saveConfig()[cite: 1]
        end
    end
})[cite: 1]

v20:Toggle({
	Title = "Auto Refresh Quests",
	Desc = "Enable instant quest refresh glitch",
	Value = t1.AUTO_REFRESH_QUESTS,
	Callback = function(p6)
        t1.AUTO_REFRESH_QUESTS = p6[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v20:Input({
	Title = "Glitch Quest Slot 1 Number",
	Desc = "First quest slot number for glitching (e.g. 1)",
	Value = tostring(t1.GLITCH_QUEST_1),
	Placeholder = "1",
	Callback = function(p7)
        local num = tonumber(p7)[cite: 1]
        if num then
            t1.GLITCH_QUEST_1 = num[cite: 1]
            saveConfig()[cite: 1]
        end
    end
})[cite: 1]

v20:Input({
	Title = "Glitch Quest Slot 2 Number",
	Desc = "Second quest slot number for glitching (e.g. 2)",
	Value = tostring(t1.GLITCH_QUEST_2),
	Placeholder = "2",
	Callback = function(p8)
        local num = tonumber(p8)[cite: 1]
        if num then
            t1.GLITCH_QUEST_2 = num[cite: 1]
            saveConfig()[cite: 1]
        end
    end
})[cite: 1]

v20:Input({
	Title = "Private Server Code",
	Desc = "Optional private server code to join automatically",
	Value = t1.PRIVATE_SERVER,
	Placeholder = "Code here...",
	Callback = function(p9)
        t1.PRIVATE_SERVER = (p9 and p9 ~= "") and p9 or "JblH87"[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
    Title = "Auto TP to Players",
    Desc = "Follow a living target during farming",
    Value = t1.AUTO_TP_PLAYERS,
    Callback = function(value)
        t1.AUTO_TP_PLAYERS = value[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

local v30 = v20:Toggle({
	Title = "Start / Stop Engine",
	Desc = "Unified toggle to start and stop the farming engine",
	Value = t1.ENGINE_ENABLED,
	Callback = function(p10)
        t1.ENGINE_ENABLED = p10[cite: 1]
        u9 = p10[cite: 1]
        saveConfig()[cite: 1]

        if p10 then
            print("🚀 Farming Engine STARTED via UI toggle.")[cite: 1]
            notify("Engine started.")[cite: 1]
            task.spawn(v12)[cite: 1]
            return
        end

        print("🛑 Farming Engine STOPPED via UI toggle.")[cite: 1]
        runId += 1[cite: 1]
        notify("Engine stopped.")[cite: 1]
    end
})[cite: 1]

v20:Toggle({
	Title = "Auto Hide UI on Start",
	Desc = "Hides UI automatically when the script executes",
	Value = t1.AUTO_HIDE_UI,
	Callback = function(p11)
        t1.AUTO_HIDE_UI = p11[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v20:Keybind({
	Title = "Menu Toggle Key",
	Desc = "Key to open or close the UI menu",
	Value = t1.TOGGLE_UI_KEY,
	Callback = function(p12)
        t1.TOGGLE_UI_KEY = (typeof(p12) == "EnumItem") and p12.Name or tostring(p12)[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v20:Keybind({
	Title = "Engine Start/Stop Key",
	Desc = "Key to start or stop the farming engine",
	Value = t1.TOGGLE_ENGINE_KEY,
	Callback = function(p13)
        t1.TOGGLE_ENGINE_KEY = (typeof(p13) == "EnumItem") and p13.Name or tostring(p13)[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Auto M1 Attacks",
	Desc = "Enable auto attacking",
	Value = t1.AUTO_M1,
	Callback = function(p14)
        t1.AUTO_M1 = p14[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Auto Skills",
	Desc = "Enable automatic skill spamming",
	Value = t1.AUTO_SKILLS,
	Callback = function(p15)
        t1.AUTO_SKILLS = p15[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Use Skill 1",
	Value = t1.SKILL_1,
	Callback = function(p16)
        t1.SKILL_1 = p16[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Use Skill 2",
	Value = t1.SKILL_2,
	Callback = function(p17)
        t1.SKILL_2 = p17[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Use Skill 3",
	Value = t1.SKILL_3,
	Callback = function(p18)
        t1.SKILL_3 = p18[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

v21:Toggle({
	Title = "Use Skill 4",
	Value = t1.SKILL_4,
	Callback = function(p19)
        t1.SKILL_4 = p19[cite: 1]
        saveConfig()[cite: 1]
    end
})[cite: 1]

local function v42()
    if not v19 then return end[cite: 1]
    if type(v19.Toggle) == "function" then
        pcall(function() v19:Toggle() end)[cite: 1]
        return
    end

    local v74 = (gethui and gethui()) or game:GetService("CoreGui") or LocalPlayer:FindFirstChildOfClass("PlayerGui")[cite: 1]
    if v74 then
        for _, v77 in ipairs(v74:GetChildren()) do
            if v77:IsA("ScreenGui") and (v77.Name:find("WindUI") or v77.Name == "ABAQuestFarm") then
                v77.Enabled = not v77.Enabled[cite: 1]
            end
        end
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end[cite: 1]

    local function v81(p20)
        return p20 and input.KeyCode ~= Enum.KeyCode.Unknown and p20 == input.KeyCode.Name[cite: 1]
    end

    if v81(t1.TOGGLE_ENGINE_KEY) then
        t1.ENGINE_ENABLED = not t1.ENGINE_ENABLED[cite: 1]
        u9 = t1.ENGINE_ENABLED[cite: 1]
        saveConfig()[cite: 1]

        if v30 and type(v30.Set) == "function" then
            v30:Set(t1.ENGINE_ENABLED)[cite: 1]
            return
        end

        if u9 then
            print("🟢 Farming Engine STARTED via hotkey [" .. tostring(t1.TOGGLE_ENGINE_KEY) .. "]")[cite: 1]
            task.spawn(v12)[cite: 1]
            return
        end

        print("🔴 Farming Engine STOPPED via hotkey [" .. tostring(t1.TOGGLE_ENGINE_KEY) .. "]")[cite: 1]
        runId += 1[cite: 1]
        notify("Engine stopped.")[cite: 1]
        return
    end

    if v81(t1.TOGGLE_UI_KEY) then
        v42()[cite: 1]
        print("👁 UI Visibility toggled via hotkey [" .. tostring(t1.TOGGLE_UI_KEY) .. "]")[cite: 1]
    end
end)

if t1.AUTO_HIDE_UI then
    task.spawn(function()
        task.wait(0.6)[cite: 1]
        v42()[cite: 1]
        print("👁 UI automatically hidden on launch.")[cite: 1]
    end)
end

notify("Script loaded. R: Engine | L: UI")[cite: 1]

if t1.ENGINE_ENABLED then
    u9 = true[cite: 1]
    task.spawn(v12)[cite: 1]
end
