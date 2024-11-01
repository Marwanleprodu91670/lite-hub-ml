-- Load Orion Library with error handling
local function loadOrionLibrary()
    local success, response = pcall(function()
        return game:HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source')
    end)

    if success and response then
        return loadstring(response)()
    else
        error("Failed to load Orion Library: " .. tostring(response))
    end
end

local OrionLib = loadOrionLibrary()
local Window = OrionLib:MakeWindow({
    Name = "Lite Hub Muscle Legends",
    HidePremium = false,
    SaveConfig = true,
    IntroEnabled = false
})

-- Create the Player Tab
local Tab = Window:MakeTab({
    Name = "Player",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

-- Define required variables
local LocalPlayer = game.Players.LocalPlayer
local Backpack = LocalPlayer:WaitForChild("Backpack")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local punchLoop, animationSpeedLoop, killLoop
local whitelistedPlayers = {}
local whitelistDropdown, selectPlayerDropdown
local selectedPlayer = nil -- Store the selected player for killing

-- Function to refresh dropdown options
local function updateDropdown(dropdown, excludeLocalPlayer)
    local options = {}
    for _, player in pairs(Players:GetPlayers()) do
        if not (excludeLocalPlayer and player == LocalPlayer) then
            table.insert(options, player.Name)
        end
    end
    dropdown:Refresh(options, true)
end

-- Initialize the whitelist dropdown
whitelistDropdown = Tab:AddDropdown({
    Name = "Whitelist",
    Default = {},
    Multi = true,
    Options = {},
    Callback = function(selected)
        whitelistedPlayers = selected
    end
})

-- Add Punching Section
Tab:AddSection({Name = "Punching"})

-- Add Auto Punch Toggle
Tab:AddToggle({
    Name = "Auto Punch",
    Default = false,
    Callback = function(Value)
        if Value then
            local punchTool = Backpack:FindFirstChild("Punch") or Character:FindFirstChild("Punch")
            if punchTool and punchTool.Parent ~= Character then
                LocalPlayer.Character.Humanoid:EquipTool(punchTool)
            end
            
            punchLoop = RunService.Heartbeat:Connect(function()
                local punchTool = Character:FindFirstChild("Punch")
                if punchTool and punchTool.Parent == Character then
                    punchTool:Activate()
                end
                wait(0.1)
            end)
        else
            if punchLoop then
                punchLoop:Disconnect()
                punchLoop = nil
            end
            local punchTool = Character:FindFirstChild("Punch")
            if punchTool then
                punchTool:Deactivate()
            end
        end
    end
})

-- Add Fast Auto Punch Toggle
Tab:AddToggle({
    Name = "Fast Auto Punch",
    Default = false,
    Callback = function(Value)
        if Value then
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
                animationSpeedLoop = nil
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

-- Add Auto Kill Toggle with Whitelist Check
Tab:AddToggle({
    Name = "Auto Kill",
    Default = false,
    Callback = function(Value)
        if Value then
            killLoop = RunService.Heartbeat:Connect(function()
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and not table.find(whitelistedPlayers, player.Name) then
                        if player.Character and player.Character:FindFirstChild("Head") then
                            local head = player.Character.Head
                            local rightArm = LocalPlayer.Character:FindFirstChild("Right Arm") or LocalPlayer.Character:FindFirstChild("RightHand")

                            if rightArm and head then
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
                killLoop = nil
            end
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
selectPlayerDropdown = Tab:AddDropdown({
    Name = "Select Player",
    Default = {},
    Multi = false,
    Options = {},
    Callback = function(selected)
        selectedPlayer = selected[1]
    end
})

-- Add Kill Target Toggle
Tab:AddToggle({
    Name = "Kill Target",
    Default = false,
    Callback = function(Value)
        if Value and selectedPlayer then
            local playerToKill = Players:FindFirstChild(selectedPlayer)
            if playerToKill and playerToKill.Character and playerToKill.Character:FindFirstChild("Head") then
                local head = playerToKill.Character.Head
                local rightHand = LocalPlayer.Character:FindFirstChild("Right Arm") or LocalPlayer.Character:FindFirstChild("RightHand")

                if rightHand and head then
                    head.CFrame = rightHand.CFrame
                end
            end
        end
    end
})

-- Player update events
Players.PlayerAdded:Connect(function()
    updateDropdown(whitelistDropdown, true)
    updateDropdown(selectPlayerDropdown, true)
end)
Players.PlayerRemoving:Connect(function()
    updateDropdown(whitelistDropdown, true)
    updateDropdown(selectPlayerDropdown, true)
end)

-- Initialize dropdowns with players excluding LocalPlayer
updateDropdown(whitelistDropdown, true)
updateDropdown(selectPlayerDropdown, true)
