-- Load Orion Library with error handling
local function loadOrionLibrary()
    local success, response = pcall(function()
        return game.HttpGet('https://raw.githubusercontent.com/shlexware/Orion/main/source')
    end)

    if success and response then
        return loadstring(response)()
    else
        error("Failed to load Orion Library: " .. tostring(response))
    end
end

local OrionLib = loadOrionLibrary()
local Window = OrionLib:MakeWindow({Name = "Lite Hub Muscle Legends", HidePremium = false, SaveConfig = true, IntroEnabled = false})

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

local punchLoop
local equipLoop
local animationSpeedLoop
local killLoop
local whitelistedPlayers = {}
local whitelistDropdown
local selectPlayerDropdown
local selectedPlayer = nil  -- Store the selected player for killing

-- Function to update whitelist dropdown options
local function updateWhitelistDropdown()
    local options = {}
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(options, player.Name)
        end
    end
    whitelistDropdown:Refresh(options, true)
end

-- Function to update player dropdown options for auto kill
local function updateSelectPlayerDropdown()
    local options = {}
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            table.insert(options, player.Name)
        end
    end
    selectPlayerDropdown:Refresh(options, true)
end

-- Add Punching Section
Tab:AddSection({
    Name = "Punching"
})

-- Add Auto Punch Toggle to use the existing Punch tool infinitely
Tab:AddToggle({
    Name = "Auto Punch",
    Default = false,
    Callback = function(Value)
        if Value then
            -- Equip tool if not already equipped
            local punchTool = Backpack:FindFirstChild("Punch") or Character:FindFirstChild("Punch")
            if punchTool and punchTool.Parent ~= Character then
                LocalPlayer.Character.Humanoid:EquipTool(punchTool)
            end
            
            -- Activate Punch infinitely every 0.1 seconds
            punchLoop = RunService.Heartbeat:Connect(function()
                local punchTool = Character:FindFirstChild("Punch")
                if punchTool and punchTool.Parent == Character then
                    punchTool:Activate() -- Activate the Punch tool
                end
                wait(0.1) -- Wait for 0.1 seconds between each punch
            end)
        else
            -- Stop the punch loop when the toggle is off
            if punchLoop then
                punchLoop:Disconnect()
                punchLoop = nil -- Clear reference
            end
            
            -- Deactivate the punch tool if it exists
            local punchTool = Character:FindFirstChild("Punch")
            if punchTool then
                punchTool:Deactivate()
            end
        end
    end
})

-- Add Fast Auto Punch Toggle to speed up the punch animation
Tab:AddToggle({
    Name = "Fast Auto Punch",
    Default = false,
    Callback = function(Value)
        if Value then
            -- Speed up the punch animation
            animationSpeedLoop = RunService.Heartbeat:Connect(function()
                local punchTool = Character:FindFirstChild("Punch")
                if punchTool then
                    for _, animTrack in pairs(LocalPlayer.Character.Humanoid:GetPlayingAnimationTracks()) do
                        if animTrack.Name == "PunchAnimation" then  -- Replace with the correct animation name
                            animTrack:AdjustSpeed(2)  -- Increase the speed to 2x (or adjust as needed)
                        end
                    end
                end
            end)
        else
            -- Reset the animation speed when the toggle is off
            if animationSpeedLoop then
                animationSpeedLoop:Disconnect()
                animationSpeedLoop = nil -- Clear reference
            end
            for _, animTrack in pairs(LocalPlayer.Character.Humanoid:GetPlayingAnimationTracks()) do
                if animTrack.Name == "PunchAnimation" then  -- Replace with the correct animation name
                    animTrack:AdjustSpeed(1)  -- Reset speed to normal
                end
            end
        end
    end
})

-- Add Killing Section
Tab:AddSection({
    Name = "Killing"
})

-- Add Whitelist Dropdown
whitelistDropdown = Tab:AddDropdown({
    Name = "Whitelist",
    Default = {},
    Multi = true,  -- Allow multiple selections
    Options = {},
    Callback = function(selected)
        whitelistedPlayers = selected  -- Update the list of whitelisted players
    end
})

-- Initialize the whitelist dropdown with current players
updateWhitelistDropdown()

-- Update dropdown when players join or leave
Players.PlayerAdded:Connect(function()
    updateWhitelistDropdown()
    updateSelectPlayerDropdown()
end)
Players.PlayerRemoving:Connect(function()
    updateWhitelistDropdown()
    updateSelectPlayerDropdown()
end)

-- Add Auto Kill Toggle with Whitelist Check
Tab:AddToggle({
    Name = "Auto Kill",
    Default = false,
    Callback = function(Value)
        if Value then
            -- Start Auto Kill loop
            killLoop = RunService.Heartbeat:Connect(function()
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and not table.find(whitelistedPlayers, player.Name) then
                        if player.Character and player.Character:FindFirstChild("Head") then
                            local head = player.Character.Head
                            local rightArm = LocalPlayer.Character:FindFirstChild("Right Arm") or LocalPlayer.Character:FindFirstChild("RightHand")

                            -- Check if rightArm exists
                            if rightArm and head then
                                -- Teleport the head to the right arm
                                head.CFrame = rightArm.CFrame * CFrame.new(1.5, 0, 0)  -- Adjust position as needed
                                head.Size = Vector3.new(0.5, 0.5, 0.5)  -- Make head smaller
                            end
                        end
                    end
                end
            end)
        else
            -- Stop the Auto Kill loop
            if killLoop then
                killLoop:Disconnect()
                killLoop = nil -- Clear reference
            end
            
            -- Reset other players' head size
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                    player.Character.Head.Size = Vector3.new(1, 1, 1) -- Reset head size to normal
                end
            end
        end
    end
})

-- Add Target Section
Tab:AddSection({
    Name = "Target"
})

-- Add Select Player Dropdown
selectPlayerDropdown = Tab:AddDropdown({
    Name = "Select Player",
    Default = {},
    Multi = false,  -- Single selection
    Options = {},
    Callback = function(selected)
        selectedPlayer = selected[1]  -- Update the selected player (single selection)
    end
})

-- Initialize the select player dropdown with current players
updateSelectPlayerDropdown()

-- Update the select player dropdown whenever players join or leave
Players.PlayerAdded:Connect(function()
    updateSelectPlayerDropdown()
end)
Players.PlayerRemoving:Connect(function()
    updateSelectPlayerDropdown()
end)

-- Add "Kill Target" Toggle
Tab:AddToggle({
    Name = "Kill Target",
    Default = false,
    Callback = function(Value)
        if Value then
            -- Check if a player is selected
            if selectedPlayer then
                local playerToKill = Players:FindFirstChild(selectedPlayer)
                if playerToKill and playerToKill.Character and playerToKill.Character:FindFirstChild("Head") then
                    local head = playerToKill.Character.Head
                    local rightHand = LocalPlayer.Character:FindFirstChild("Right Arm") or LocalPlayer.Character:FindFirstChild("RightHand")

                    -- Check if rightHand exists
                    if rightHand and head then
                        -- Teleport the head to the right hand
                        head.CFrame = rightHand.CFrame * CFrame.new(0, 0, 0)  -- Adjust position as needed
                    end
                end
            end
        end
    end
})


