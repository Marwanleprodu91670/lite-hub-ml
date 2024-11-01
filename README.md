-- Load the Orion Library
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()
local Window = OrionLib:MakeWindow({Name = "Game Script", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})

-- Variables
local autoPunchEnabled = false
local autoKillEnabled = false
local whitelistEnabled = false
local killTargetEnabled = false
local selectedWhitelistPlayer = nil
local selectedTargetPlayer = nil

-- Function to toggle Auto Punch
function AutoPunch()
    spawn(function()
        while autoPunchEnabled do
            local player = game.Players.LocalPlayer
            local punchTool = player.Backpack:FindFirstChild("Punch") or player.Character:FindFirstChild("Punch")
            
            if punchTool then
                -- Equip and activate the Punch tool
                player.Character.Humanoid:EquipTool(punchTool)
                punchTool:Activate()
            end
            wait(0.1) -- Adjust delay as needed
        end
    end)
end

-- Function to toggle Auto Kill
function AutoKill()
    spawn(function()
        while autoKillEnabled do
            local player = game.Players.LocalPlayer
            local character = player.Character or player.CharacterAdded:Wait()
            local rightHand = character:FindFirstChild("RightHand") -- or use a specific part if needed

            if rightHand then
                for _, target in pairs(game.Players:GetPlayers()) do
                    if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
                        -- Skip teleporting if whitelist is enabled and the player is selected
                        if not (whitelistEnabled and target.Name == selectedWhitelistPlayer) then
                            target.Character.Head.CFrame = rightHand.CFrame
                        end
                    end
                end
            end
            wait(0.1) -- Adjust delay as needed
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
            wait(0.1) -- Adjust delay as needed
        end
    end)
end

-- Function to refresh player list for dropdown
local function RefreshPlayerList()
    local players = {}
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Name ~= game.Players.LocalPlayer.Name then -- Exclude local player
            table.insert(players, player.Name)
        end
    end
    return players
end

-- Add "Player" Tab
local PlayerTab = Window:MakeTab({
    Name = "Player",
    Icon = "rbxassetid://4483345998", -- Example icon
    PremiumOnly = false
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

-- Empty Section
PlayerTab:AddSection({
    Name = " "
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

-- Update dropdown options whenever players change
game.Players.PlayerAdded:Connect(function()
    PlayerTab:UpdateDropdown("Select Player to Whitelist", RefreshPlayerList())
    PlayerTab:UpdateDropdown("Select Target Player", RefreshPlayerList())
end)

game.Players.PlayerRemoving:Connect(function()
    PlayerTab:UpdateDropdown("Select Player to Whitelist", RefreshPlayerList())
    PlayerTab:UpdateDropdown("Select Target Player", RefreshPlayerList())
end)

-- Initialize Orion Library
OrionLib:Init()
