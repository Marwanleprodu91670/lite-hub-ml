-- Load Orion Library
local OrionLib = loadstring(game.HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source'))()
local Window = OrionLib:MakeWindow({Name = "Lite Hub Muscle Legends", HidePremium = false, SaveConfig = true, IntroEnabled = false})

-- Create the Player Tab
local Tab = Window:MakeTab({
    Name = "Player",
    Icon = rbxassetid4483345998,
    PremiumOnly = false
})

-- Define required variables
local LocalPlayer = game.Players.LocalPlayer
local Backpack = LocalPlayer:WaitForChild("Backpack")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local punchLoop, equipLoop, animationSpeedLoop, killLoop
local whitelistedPlayers = {}
local selectedPlayer = nil

-- Function to update dropdown options
local function updateDropdown(dropdown, options)
    dropdown:Refresh(options, true)
end

-- Function to get all player names except LocalPlayer
local function getPlayerNames()
    local options = {}
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(options, player.Name)
        end
    end
    return options
end

-- Add Punching Section
Tab:AddSection({Name = "Punching"})

-- Add Auto Punch Toggle
Tab:AddToggle({
    Name = "Auto Punch",
    Default = false,
    Callback = function(isEnabled)
        if isEnabled then
            local punchTool = Backpack:FindFirstChild("Punch") or Character:FindFirstChild("Punch")
            if punchTool and punchTool.Parent ~= Character then
                LocalPlayer.Character.Humanoid:EquipTool(punchTool)
            end

            -- Activate Punch infinitely
            punchLoop = RunService.Heartbeat:Connect(function()
                if Character:FindFirstChild("Punch") then
                    Character.Punch:Activate()
                end
                wait(0.1)
            end)
        else
            if punchLoop then
                punchLoop:Disconnect()
            end
            if Character:FindFirstChild("Punch") then
                Character.Punch:Deactivate()
            end
        end
    end
})

-- Add Fast Auto Punch Toggle
Tab:AddToggle({
    Name = "Fast Auto Punch",
    Default = false,
    Callback = function(isEnabled)
        if isEnabled then
            animationSpeedLoop = RunService.Heartbeat:Connect(function()
                for _, animTrack in pairs(LocalPlayer.Character.Humanoid:GetPlayingAnimationTracks()) do
                    if animTrack.Name == "PunchAnimation" then
                        animTrack:AdjustSpeed(2)
                    end
                end
            end)
        else
            if animationSpeedLoop then
                animationSpeedLoop:Disconnect()
            end
            for _, animTrack in pairs(LocalPlayer.Character.Humanoid:GetPlayingAnimationTracks()) do
                if animTrack.Name == "PunchAnimation" then
                    animTrack:AdjustSpeed(1)
                end
            end
        end
    end
})

-- Add Killing Section
Tab:AddSection({Name = "Killing"})

-- Add Whitelist Dropdown
local whitelistDropdown = Tab:AddDropdown({
    Name = "Whitelist",
    Default = {},
    Multi = true,
    Options = getPlayerNames(),
    Callback = function(selected)
        whitelistedPlayers = selected
    end
})

-- Update dropdown when players join or leave
Players.PlayerAdded:Connect(function() updateDropdown(whitelistDropdown, getPlayerNames()) end)
Players.PlayerRemoving:Connect(function() updateDropdown(whitelistDropdown, getPlayerNames()) end)

-- Add Auto Kill Toggle
Tab:AddToggle({
    Name = "Auto Kill",
    Default = false,
    Callback = function(isEnabled)
        if isEnabled then
            killLoop = RunService.Heartbeat:Connect(function()
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and not table.find(whitelistedPlayers, player.Name) then
                        if player.Character and player.Character:FindFirstChild("Head") then
                            local head = player.Character.Head
                            local rightArm = Character:FindFirstChild("Right Arm") or Character:FindFirstChild("RightHand")
                            if rightArm then
                                head.CFrame = rightArm.CFrame * CFrame.new(1.5, 0, 0)
                                head.Size = Vector3.new(0.5, 0.5, 0.5)
                            end
                        end
                    end
                end
            end)
        else
            if killLoop then
                killLoop:Disconnect()
            end
            -- Reset other players' head size
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                    player.Character.Head.Size = Vector3.new(1, 1, 1)
                end
            end
        end
    end
})

-- Add Target Section
Tab:AddSection({Name = "Target"})

-- Add Select Player Dropdown
local selectPlayerDropdown = Tab:AddDropdown({
    Name = "Select Player",
    Default = {},
    Multi = false,
    Options = getPlayerNames(),
    Callback = function(selected)
        selectedPlayer = selected[1]
    end
})

-- Update the select player dropdown whenever players join or leave
Players.PlayerAdded:Connect(function() updateDropdown(selectPlayerDropdown, getPlayerNames()) end)
Players.PlayerRemoving:Connect(function() updateDropdown(selectPlayerDropdown, getPlayerNames()) end)

-- Add "Kill Target" Toggle
Tab:AddToggle({
    Name = "Kill Target",
    Default = false,
    Callback = function(isEnabled)
        if isEnabled and selectedPlayer then
            local playerToKill = Players:FindFirstChild(selectedPlayer)
            if playerToKill and playerToKill.Character and playerToKill.Character:FindFirstChild("Head") then
                local head = playerToKill.Character.Head
                local rightHand = Character:FindFirstChild("Right Arm") or Character:FindFirstChild("RightHand")
                if rightHand then
                    head.CFrame = rightHand.CFrame
                end
            end
        end
    end
})

-- Add an Empty Section
Tab:AddSection({Name = "Empty Section"})

-- Create the Spy Tab
local SpyTab = Window:MakeTab({
    Name = "Spy",
    Icon = rbxassetid4483345998,
    PremiumOnly = false
})

-- Add Select Player Stats Dropdown
local selectPlayerStatsDropdown = SpyTab:AddDropdown({
    Name = "Select Player Stats",
    Default = {},
    Multi = false,
    Options = getPlayerNames(),
    Callback = function(selected)
        selectedPlayerForStats = selected[1]
    end
})

local selectedPlayerForStats = nil

-- Add Durability Section
local durabilitySection = SpyTab:AddSection({Name = "Durability:"})
local durabilityLabel = durabilitySection:AddLabel("Durability: --")

-- Function to update durability of the selected player
local function updateDurability()
    while true do
        wait(1)
        if selectedPlayerForStats then
            local playerToTrack = Players:FindFirstChild(selectedPlayerForStats)
            if playerToTrack and playerToTrack:FindFirstChild("Durability") then
                local durabilityValue = playerToTrack.Durability.Value
                durabilityLabel:SetText("Durability: " .. durabilityValue)
            else
                durabilityLabel:SetText("Durability: --")
            end
        else
            durabilityLabel:SetText("Durability: --")
        end
    end
end

-- Start the durability update loop
spawn(updateDurability)

-- Initialize the select player stats dropdown with current players
updateDropdown(selectPlayerStatsDropdown, getPlayerNames())

-- Update the select player stats dropdown whenever players join or leave
Players.PlayerAdded:Connect(function() updateDropdown(selectPlayerStatsDropdown, getPlayerNames()) end)
Players.PlayerRemoving:Connect(function() updateDropdown(selectPlayerStatsDropdown, getPlayerNames()) end)

-- Add an Empty Section
SpyTab:AddSection({Name = "Another Empty Section"})

-- You can add more features and functionalities as needed

-- End of Script
