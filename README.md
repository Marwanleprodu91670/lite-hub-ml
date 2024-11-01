-- Load the Orion Library
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

local Window = OrionLib:MakeWindow({Name = "Lite Hub Muscle Legends", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})



-- Variables
local autoPushupsEnabled = false
local autoSitupsEnabled = false
local autoWeightEnabled = false
local autoHandstandsEnabled = false
local autoPunchEnabled = false
local autoKillEnabled = false
local whitelistEnabled = false
local killTargetEnabled = false
local spyPlayerEnabled = false
local selectedWhitelistPlayer = nil
local selectedTargetPlayer = nil
local selectedSpyPlayer = nil
local lockPositionEnabled = false
local selectedPosition = CFrame.new(0, 0, 0)

--another variable to make script still work after player die
local function onCharacterAdded(character)
    character:WaitForChild("Humanoid") -- Wait for the Humanoid to load

    -- Reinitialize any required toggles or tools here
    if autoPushupsEnabled then AutoPushups() end
    if autoSitupsEnabled then AutoSitups() end
    if autoWeightEnabled then AutoWeight() end
    if autoHandstandsEnabled then AutoHandstands() end
    if autoPunchEnabled then AutoPunch() end
    if autoKillEnabled then AutoKill() end
    if killTargetEnabled then KillTarget() end
end

local player = game.Players.LocalPlayer

-- Initial character setup
if player.Character then
    onCharacterAdded(player.Character)
end

-- Connect the character added event
player.CharacterAdded:Connect(onCharacterAdded)


-- Function to toggle Auto Pushups
function AutoPushups()
    spawn(function()
        while autoPushupsEnabled do
            local player = game.Players.LocalPlayer
            local pushupsTool = player.Backpack:FindFirstChild("Pushups") or player.Character:FindFirstChild("Pushups")
            
            if pushupsTool then
                player.Character.Humanoid:EquipTool(pushupsTool)
                pushupsTool:Activate()
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Auto Situps
function AutoSitups()
    spawn(function()
        while autoSitupsEnabled do
            local player = game.Players.LocalPlayer
            local situpsTool = player.Backpack:FindFirstChild("Situps") or player.Character:FindFirstChild("Situps")
            
            if situpsTool then
                player.Character.Humanoid:EquipTool(situpsTool)
                situpsTool:Activate()
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Auto Weight
function AutoWeight()
    spawn(function()
        while autoWeightEnabled do
            local player = game.Players.LocalPlayer
            local weightTool = player.Backpack:FindFirstChild("Weight") or player.Character:FindFirstChild("Weight")
            
            if weightTool then
                player.Character.Humanoid:EquipTool(weightTool)
                weightTool:Activate()
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Auto Handstands
function AutoHandstands()
    spawn(function()
        while autoHandstandsEnabled do
            local player = game.Players.LocalPlayer
            local handstandsTool = player.Backpack:FindFirstChild("Handstands") or player.Character:FindFirstChild("Handstands")
            
            if handstandsTool then
                player.Character.Humanoid:EquipTool(handstandsTool)
                handstandsTool:Activate()
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Auto Punch
function AutoPunch()
    spawn(function()
        while autoPunchEnabled do
            local player = game.Players.LocalPlayer
            local punchTool = player.Backpack:FindFirstChild("Punch") or player.Character:FindFirstChild("Punch")
            
            if punchTool then
                player.Character.Humanoid:EquipTool(punchTool)
                punchTool:Activate()
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Auto Kill
function AutoKill()
    spawn(function()
        while autoKillEnabled do
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local rightHand = character:FindFirstChild("RightHand")

            if rightHand then
                for _, target in pairs(game.Players:GetPlayers()) do
                    if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
                        if not (whitelistEnabled and target.Name == selectedWhitelistPlayer) then
                            target.Character.Head.CFrame = rightHand.CFrame
                        end
                    end
                end
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Kill Target
function KillTarget()
    spawn(function()
        while killTargetEnabled do
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local rightHand = character:FindFirstChild("RightHand")

            if rightHand and selectedTargetPlayer then
                local targetPlayer = game.Players:FindFirstChild(selectedTargetPlayer)
                if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
                    targetPlayer.Character.Head.CFrame = rightHand.CFrame
                end
            end
            wait(0.1)
        end
    end)
end

-- Function to toggle Spy Player
function SpyPlayer()
    spawn(function()
        while spyPlayerEnabled do
            if selectedSpyPlayer then
                local targetPlayer = game.Players:FindFirstChild(selectedSpyPlayer)
                if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
                    workspace.CurrentCamera.CameraSubject = targetPlayer.Character.Humanoid
                end
            end
            wait(0.1)
        end
    end)
end

-- Function to reset camera to local player
function ResetCamera()
    workspace.CurrentCamera.CameraSubject = game.Players.LocalPlayer.Character.Humanoid
end

-- Function to lock or unlock player position
function LockPosition()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    
    if lockPositionEnabled then
        humanoid.PlatformStand = true -- Freeze the player
    else
        humanoid.PlatformStand = false -- Unfreeze the player
    end
end

-- Function to refresh player list for dropdown
local function RefreshPlayerList()
    local players = {}
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Name ~= game.Players.LocalPlayer.Name then
            table.insert(players, player.Name)
        end
    end
    return players
end

-- Function for Anti Ping
function AntiPing()
    local decalsyeeted = true -- Leaving this on makes games look shitty but the fps goes up by at least 20. 
    local g = game
    local w = g.Workspace
    local l = g.Lighting
    local t = w.Terrain
    t.WaterWaveSize = 0
    t.WaterWaveSpeed = 0
    t.WaterReflectance = 0
    t.WaterTransparency = 0
    l.GlobalShadows = false
    l.FogEnd = 9e9
    l.Brightness = 0
    settings().Rendering.QualityLevel = "Level01"
    for i, v in pairs(g:GetDescendants()) do
        if v:IsA("Part") or v:IsA("Union") or v:IsA("CornerWedgePart") or v:IsA("TrussPart") then
            v.Material = "Plastic"
            v.Reflectance = 0
        elseif v:IsA("Decal") or v:IsA("Texture") and decalsyeeted then
            v.Transparency = 1
        elseif v:IsA("ParticleEmitter") or v:IsA("Trail") then
            v.Lifetime = NumberRange.new(0)
        elseif v:IsA("Explosion") then
            v.BlastPressure = 1
            v.BlastRadius = 1
        elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") then
            v.Enabled = false
        elseif v:IsA("MeshPart") then
            v.Material = "Plastic"
            v.Reflectance = 0
            v.TextureID = 10385902758728957
        end
    end
    for i, e in pairs(l:GetChildren()) do
        if e:IsA("BlurEffect") or e:IsA("SunRaysEffect") or e:IsA("ColorCorrectionEffect") or e:IsA("BloomEffect") or e:IsA("DepthOfFieldEffect") then
            e.Enabled = false
        end
    end
end

-- Add "Main" Tab
local MainTab = Window:MakeTab({
    Name = "Main",
    Icon = "rbxassetid://128585740744252",
    PremiumOnly = false
})

-- General Farm Section
MainTab:AddSection({
    Name = "General Farm"
})

-- Auto Pushups Toggle
MainTab:AddToggle({
    Name = "Auto Pushups",
    Default = false,
    Callback = function(value)
        autoPushupsEnabled = value
        if value then
            AutoPushups()
        end
    end
})

-- Auto Situps Toggle
MainTab:AddToggle({
    Name = "Auto Situps",
    Default = false,
    Callback = function(value)
        autoSitupsEnabled = value
        if value then
            AutoSitups()
        end
    end
})

-- Auto Weight Toggle
MainTab:AddToggle({
    Name = "Auto Weight",
    Default = false,
    Callback = function(value)
        autoWeightEnabled = value
        if value then
            AutoWeight()
        end
    end
})

-- Auto Handstands Toggle
MainTab:AddToggle({
    Name = "Auto Handstands",
    Default = false,
    Callback = function(value)
        autoHandstandsEnabled = value
        if value then
            AutoHandstands()
        end
    end
})

-- Misc Section
MainTab:AddSection({
    Name = "Misc"
})

-- Lock Position Toggle
MainTab:AddToggle({
    Name = "Lock Position",
    Default = false,
    Callback = function(value)
        lockPositionEnabled = value
        LockPosition()
    end
})

-- Anti Ping Button (now moved after Lock Position)
MainTab:AddButton({
    Name = "Anti Ping",
    Callback = function()
        AntiPing()
    end
})

-- Add "Player" Tab
local PlayerTab = Window:MakeTab({
    Name = "Player",
    Icon = "rbxassetid://93620904978325",
    PremiumOnly = false
})

-- Kill Farming Section
PlayerTab:AddSection({
    Name = "Kill Farming"
})

-- Auto Punch Toggle
PlayerTab:AddToggle({
    Name = "Auto Punch",
    Default = false,
    Callback = function(value)
        autoPunchEnabled = value
        if value then
            AutoPunch()
        end
    end
})

-- Auto Kill Toggle
PlayerTab:AddToggle({
    Name = "Auto Kill",
    Default = false,
    Callback = function(value)
        autoKillEnabled = value
        if value then
            AutoKill()
        end
    end
})

-- Whitelist Toggle
PlayerTab:AddToggle({
    Name = "Whitelist",
    Default = false,
    Callback = function(value)
        whitelistEnabled = value
    end
})

-- Dropdown for Player Whitelist
PlayerTab:AddDropdown({
    Name = "Select Player to Whitelist",
    Default = "",
    Options = RefreshPlayerList(),
    Callback = function(value)
        selectedWhitelistPlayer = value
    end
})

-- Target Section
PlayerTab:AddSection({
    Name = "Target"
})

-- Kill Target Toggle
PlayerTab:AddToggle({
    Name = "Kill Target",
    Default = false,
    Callback = function(value)
        killTargetEnabled = value
        if value then
            KillTarget()
        end
    end
})

-- Dropdown for Target Player Selection
PlayerTab:AddDropdown({
    Name = "Select Target Player",
    Default = "",
    Options = RefreshPlayerList(),
    Callback = function(value)
        selectedTargetPlayer = value
    end
})

-- Add "Spy" Tab
local SpyTab = Window:MakeTab({
    Name = "Spy",
    Icon = "rbxassetid://104759358264096",
    PremiumOnly = false
})

-- Spy Player Section
SpyTab:AddSection({
    Name = "Spy Player"
})

-- Spy Player Toggle
SpyTab:AddToggle({
    Name = "Spy Player",
    Default = false,
    Callback = function(value)
        spyPlayerEnabled = value
        if value then
            SpyPlayer()
        else
            ResetCamera()
        end
    end
})

-- Dropdown for Spy Player Selection
SpyTab:AddDropdown({
    Name = "Select Spy Player",
    Default = "",
    Options = RefreshPlayerList(),
    Callback = function(value)
        selectedSpyPlayer = value
    end
})

-- Player Added/Removed Events
game.Players.PlayerAdded:Connect(function()
    PlayerTab:UpdateDropdown("Select Player to Whitelist", RefreshPlayerList())
    PlayerTab:UpdateDropdown("Select Target Player", RefreshPlayerList())
    SpyTab:UpdateDropdown("Select Spy Player", RefreshPlayerList())
end)

game.Players.PlayerRemoving:Connect(function()
    PlayerTab:UpdateDropdown("Select Player to Whitelist", RefreshPlayerList())
    PlayerTab:UpdateDropdown("Select Target Player", RefreshPlayerList())
    SpyTab:UpdateDropdown("Select Spy Player", RefreshPlayerList())
end)

MainTab:AddButton({
    Name = "Destroy Ad",
    Callback = function()
        -- Find and destroy the part named "RobloxForwardPortals"
        local part = workspace:FindFirstChild("RobloxForwardPortals")
        if part then
            part:Destroy()
            print("Part 'RobloxForwardPortals' has been destroyed.")
        else
            print("Part 'RobloxForwardPortals' not found.")
        end
    end
})

local TeleportTab = Window:MakeTab({
    Name = "Teleport",
    Icon = "rbxassetid://137532037898337",
    PremiumOnly = false
})
 
TeleportTab:AddSection({
    Name = "Teleports"
})
 
-- Dropdown for selecting island
TeleportTab:AddDropdown({
    Name = "Select Island",
    Options = {
        "Tiny Island",
        "Frost Island",
        "Mythical Island",
        "Inferno Island",
        "Legend Island",
        "Muscle King"
    },
    Callback = function(option)
        local positions = {
            ["Tiny Island"] = CFrame.new(-38.4037132, 9.66723633, 1832.29114),
            ["Frost Island"] = CFrame.new(-2533.64258, 13.7738762, -408.134796),
            ["Mythical Island"] = CFrame.new(2170.5437, 13.8738737, 1073.75525),
            ["Inferno Island"] = CFrame.new(-6678.75635, 13.8738737, -1285.4198),
            ["Legend Island"] = CFrame.new(4685.5625, 997.608765, -3910.30908),
            ["Muscle King"] = CFrame.new(-8546.25879, 23.045435, -5636.78418)
        }
        selectedPosition = positions[option]
    end
})
 
-- Toggle to enable teleport
TeleportTab:AddToggle({
    Name = "Teleport To Island",
    Default = false,
    Callback = function(state)
        local player = game.Players.LocalPlayer
        if state and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = selectedPosition
            print("Teleported to:", selectedPosition.Position)
        end
    end
})
 
-- Dropdown for selecting rock
TeleportTab:AddDropdown({
    Name = "Select Rock",
    Options = {
        "Tiny Rock",
        "Frozen Rock",
        "Mystic Rock",
        "Inferno Rock",
        "Rock Of Legends",
        "Muscle King Rock"
    },
    Callback = function(option)
        local rockPositions = {
            ["Tiny Rock"] = CFrame.new(17.6410236, -1.30998898, 2106.48926, -1, 0, 0, 0, 1, 0, 0, 0, -1),
            ["Frozen Rock"] = CFrame.new(-2551.75854, -0.359962642, -243.308777, -1, 0, 0, 0, 1, 0, 0, 0, -1),
            ["Mystic Rock"] = CFrame.new(2186.14111, -0.359961987, 1250.59802, 0.996191859, -0, -0.0871884301, 0, 1, -0, 0.0871884301, 0, 0.996191859),
            ["Inferno Rock"] = CFrame.new(-7262.18701, -0.359961987, -1259.24426, 1, 0, 0, 0, 1, 0, 0, 0, 1),
            ["Rock Of Legends"] = CFrame.new(4140.41797, 987.453186, -4089.34937, -1, 0, 0, 0, 1, 0, 0, 0, -1),
            ["Muscle King Rock"] = CFrame.new(-8971.56641, 27.4031715, -6061.27734, -1, 0, 0, 0, 1, 0, 0, 0, -1)
        }
        selectedPosition = rockPositions[option]
    end
})
 
-- Toggle to enable teleport to rock
TeleportTab:AddToggle({
    Name = "Teleport To Rock",
    Default = false,
    Callback = function(state)
        toggleOn = state
        if toggleOn then
            local player = game.Players.LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = selectedPosition
                print("Teleported to:", selectedPosition.Position)
            end
        end
    end
})

local Tab = Window:MakeTab({
	Name = "Credit",
	Icon = "rbxassetid://120110582639983",
	PremiumOnly = false
})

local Section = Tab:AddSection({
	Name = "Script Made By Adopt"
})

local Section = Tab:AddSection({
	Name = "Credit to Ras for making showcase video"
})

local Section = Tab:AddSection({
	Name = "Credit to Ztx for Helping me make and fix auto kill"
})

Tab:AddSection({
    Name = "Discord Server:"
})

-- Button to copy Discord link
Tab:AddButton({
    Name = "Copy Discord Link",
    Callback = function()
        setclipboard("https://discord.gg/mpwFtrAA")
        
        -- Create a notification after copying the link
        OrionLib:MakeNotification({
            Name = "Link Copied!",
            Content = "Discord link copied to clipboard!",
            Image = "rbxassetid://4483345998",
            Time = 5
        })
        
        print("Discord link copied to clipboard!")
    end
})

-- Initialize Orion Library


